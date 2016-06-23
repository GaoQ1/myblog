---
title: JSON.stringify--JSON.parse
date: 2016-06-23 15:54:07
tags:
- javascript
- JSON
categories: 笔记
---
> When things go wrong, don't go with them.

## JSON.stringify()概述
JSON.stringify()方法可以将任意的javascript值序列化成JSON字符串。
`JSON.stringify(value[, replacer [, space]])`

### 参数
value -- 将要序列化成JSON字符串的值。
replacer -- (可选)如果该参数是一个函数，则在序列化过程中，被序列化的值的每个属性都会经过该函数的转换和处理；如果该参数是一个数组，则只有包含在这个数组中的属性名才会被序列化到最终的JSON字符串中。
space -- (可选)指定缩进用的空白字符，用于美化输出(pretty-print)。

### 描述
关于序列化，有下面五点注意事项：
 - 非数组对象的属性不能保证以特定的顺序出现序列化后的字符串中。
 - 布尔值、数字、字符串的包装对象在序列化过程中会自动转换成对应的原始值。
 - undefined、任意的函数以及symbol值，在序列化过程中会被忽略(出现在非数组对象的属性值中时)或者被转换成null(出现在数组中时)。
 - 所有以symbol为属性键的属性都会被完全忽略掉，即便replacer参数中强制指定包含了它们。
 - 不可枚举的属性会被忽略。

```javascript
  JSON.stringify({});   //'{}'
  JSON.stringify(true);   //'true'
  JSON.stringify("foo");   //'"foo"'
  JSON.stringify([1, "false", false]);   //'[1, "false", false]'
  JSON.stringify({x:5});    //'{"x":5}'

  JSON.stringify({x: 5, y: 6});              
  // '{"x":5,"y":6}' 或者 '{"y":6,"x":5}' 都可能
  JSON.stringify([new Number(1), new String("false"), new Boolean(false)]);
  // '[1,"false",false]'
  JSON.stringify({x: undefined, y: Object, z: Symbol("")});
  // '{}'
  JSON.stringify([undefined, Object, Symbol("")]);          
  // '[null,null,null]'
  JSON.stringify({[Symbol("foo")]: "foo"});                 
  // '{}'
  JSON.stringify({[Symbol.for("foo")]: "foo"}, [Symbol.for("foo")]);
  // '{}'
  JSON.stringify({[Symbol.for("foo")]: "foo"}, function (k, v) {
    if (typeof k === "symbol"){
      return "a symbol";
    }
  });

  // '{}'  

  // 不可枚举的属性默认会被忽略：
  JSON.stringify( Object.create(null, { x: { value: 'x', enumerable: false }, y: { value: 'y', enumerable: true } }) );
  // '{"y":"y"}'
```

### space参数
space参数用来控制结果字符串里面的间距。如果是一个数字，则在字符串化时每一级别会比上一级别缩进多这个数字值的空格(最多10个空格);如果是一个字符串，则每一级别会比上一级别多缩进用该字符串(或该字符串的前十个字符)。
```javascript
  JSON.stringify({a: 2}, null, ' ');    //'{\n"a": 2\n}'
```

使用制表符(\t)来缩进
```javascript
  JSON.stringify({uno: 1, doc: 2}, null, '\t');
  // '{            \
  //     "uno": 1, \
  //     "dos": 2  \
  // }'
```

### toJSON方法
如果一个被序列化的对象拥有toJSON方法，那么该toJSON方法就会覆盖该对象默认的序列化行为：不是按个对象被序列化，而是调用toJSON方法后的返回值会被序列化，例如：
```javascript
  var obj = {
    foo: 'foo',
    toJSON: function(){
      return 'bar';
    }
  };

  JSON.stringify(obj);  //'"bar"'
  JSON.stringify({x: obj});  //'{"x": "bar"}'
```

### 使用JSON.stringify结合localStorage的例子
一些时候，你想存储用户创建的一个对象，并且即使在浏览器被关闭后仍能恢复该对象。下面的例子是JSON.stringify适用于这种情形的一个样板：
```javascript
  //创建一个示例数据
  var session = {
    'screens': [],
    'state': true
  };
  session.screens.push({"name":"screenA", "width":450, "height":250});
  session.screens.push({"name":"screenB", "width":650, "height":350});
  session.screens.push({"name":"screenC", "width":750, "height":120});
  session.screens.push({"name":"screenD", "width":250, "height":60});
  session.screens.push({"name":"screenE", "width":390, "height":120});
  session.screens.push({"name":"screenF", "width":1240, "height":650});

  //使用JSON.stringify转换为JSON字符串
  //然后使用localStorage保存在session名称里
  localStorage.setItem('session', JSON.stringify(session));

  //然后是如何转换通过JSON.stringify生成的字符串，该字符串以JSON格式保存在localStorage里
  var restoreSession = JSON.parse(localStorage.getItem('session'));

  //现在restoreSession 包含了保存在localStorage里的对象
  console.log(restoreSession);
```

## JSON.parse()概述
JSON.parse()方法可以将一个JSON字符串解析成为一个javascript值。在解析过程中，还可以选择性的修改某些属性的原始解析值。
`JSON.parse(text[, reviver])`

### 参数
text -- 要解析的JSON字符串
reviver -- 一个函数，用来转换解析出的属性值

### 返回值
从text字符串解析出的一个Object

### 异常
如果解析的JSON字符串包含语法错误，则会抛出SyntaxError异常。

## 示例
### 使用JSON.parse()
```javascript
  JSON.parse('{}');              // {}
  JSON.parse('true');            // true
  JSON.parse('"foo"');           // "foo"
  JSON.parse('[1, 5, "false"]'); // [1, 5, "false"]
  JSON.parse('null');            // null
```

### 使用reviver函数
如果指定了reviver函数，则解析出的javascript值(解析值)会经过一次转换后才将被最终返回(返回值)。更具体点就是：解析值本身以及它所包含的所有属性，会按照一定的顺序(从最最里层的属性开始，一级级往外，最终到达顶层，也就是解析值本身)分别的去调用reviver函数，在调用过程中，当前属性所属的对象回作为this值，当前属性名和属性值会分别作为第一个和第二个参数传入reviver中。如果reviver返回undefined，则当前属性会从所属对象中删除，如果返回了其他值，则返回的值会成为当前属性新的属性值。

当遍历到最顶层的值(解析值)时，传入reviver函数的参数会是空字符串""(因为此时已经没有真正的属性)和当前的解析值(有可能已经被修改过)，当前的this值会是{"":修改过得解析值}，在编写reviver函数时，要注意到这个特例。

```javascript
  JSON.parse('{"p":5}', function(k,v){
    if(k === '') return v;  // 如果到了最顶层，则直接返回属性值，
    return v*2;             // 否则将属性值变为原来的 2 倍。
  })                        // { p: 10 }

  JSON.parse('{"1": 1, "2": 2,"3": {"4": 4, "5": {"6": 6}}}', function (k, v) {
    console.log(k); // 输出当前的属性名，从而得知遍历顺序是从内向外的，
                    // 最后一个属性名会是个空字符串。
    return v;       // 返回原始属性值，相当于没有传递 reviver 参数。
  });

  // 1
  // 2
  // 4
  // 6
  // 5
  // 3
  // ""
```

### JSON.parse()不允许用逗号作为结尾
```javascript
  //both will throw a SyntaxError
  JSON.parse("[1,2,3,4, ]");
  JSON.parse('{"foo":1, }');
```
