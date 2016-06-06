---
title: es6系列教程三
date: 2016-06-02 20:45:03
tags:
  - es6
  - javascript
categories: 原创
---
> Experience is the mother.

# Destructuring
解构赋值(destructuring assignment)语法是一个Javascript表达式，它使得从数组或者对象中提取数据赋值给不同的变量成为可能。

## 语法
```javascript
  [a,b] = [1,2];
  [a,b,...rest] = [1,2,3,4,5];
  {a,b} = {a:1,b:2}
  {a,b,...rest} = {a:1,b:2,c:3,d:4} //ES7
```

> {a,b} = {a:1,b:2}作为独立语法是非法的，左侧的{a,b}会被当成块结构而不是一个对象。
然而({a,b} = {a:1,b:2})的形式是允许的，其等价于var {a,b} = {a:1,b:2}.  

## 简述
对象字面量和数组字面量提供了一种简单的定义一个特定的数据组的方法。一旦你创建了这类数据组，你可以用任意的方法使用这些数据组，甚至从函数中返回它们。

解构赋值的一个特别有用的功能是：你可以用一个表达式读取整个结构。你可以从后面的例子中了解到很多它的其他的有趣应用。

解构赋值的作用类似于Perl和Python语言中的相似特性。

## 解构数组
### 简单例子
```javascript
  var foo = ["one","two","three"];

  //没有解构赋值
  var one = foo[0];
  var two = foo[1];
  var three = foo[2];

  //解构赋值
  var [one,two,three] = foo;
```

### 变换变量
```javascript
  var a = 1;
  var b = 2;
  [a,b] = [b,a];
```
执行这段代码后，b是1、a是3.没有解构赋值的情况下，交换两个变量需要一个临时变量.

### 返回多值
感谢解构赋值，函数现在可以返回多个值。尽管函数一直都可以返回一个数组，但现在这样做有更多的灵活性。
```javascript
  function f(){
    return [1,2];
  }
```

如你所见，两个返回值被括号括起来，用类似数组表示法的方式被返回。你可用这种方法返回任意量的变量。这个方法中，f()返回[1,2].
```javascript
  var a,b;
  [a,b] = f();
  console.log("A is " + a + " B is " + b);
```

[a,b] = f()把函数的返回值按顺序赋值给括号中的变量：a赋值为1、b赋值为2.

你也可以把返回值当作数组对待：
```javascript
  var a = f();
  console.log("A is " + a);
```
这样a就是一个包含1和2两个值的数组。

### 忽略某些返回值
你也可以忽略你不感兴趣的返回值：
```javascript
  function f(){
    return [1,2,3];
  }

  var [a,b] = f();
  console.log("A is " + a + " B is " + b);
```

运行这段代码后，a是1、b是3。2这个值被忽略，你可以忽略任意(或全部)返回值。例如：
```javascript
  [,,] = f();
```

### 用正则表达式匹配提取值
用正则表达式方法exec()匹配字符串返回一个数组，该数组第一个值是完全匹配正则表达式的字符串，然后的值是匹配正则表达式括号内内容部分。解构赋值允许你轻易地提取出需要的部分，忽略完全匹配的字符串--如果不需要的话。
```javascript
  var url = "https://developer.mozilla.org/en-US/Web/JavaScript";

  var parsedURL = /^(\w+)\:\/\/([]^\/]+)\/(.*)$/.exec(url);
  var [,protocol,fullhost,fullpath] = parseURL;

  console.log(protocol); //输出"https:"
```

## 解构对象
### 简单示例
```javascript
  var o = {p:42,q:true};
  var {p,q} = o;

  console.log(p);
  console.log(q);

  //用新变量名赋值
  var {p:foo,q:bar} = o;

  console.log(foo); //42
  console.log(bar); //true
```

### 函数参数默认值
ES5版本
```javascript
  function drawES5Chart(options) {
    options = options === undefined ? {} : options;
    var size = options.size === undefined ? 'big' : options.size;
    var cords = options.cords === undefined ? { x: 0, y: 0 } : options.cords;
    var radius = options.radius === undefined ? 25 : options.radius;
    console.log(size, cords, radius);
    // now finally do some chart drawing
  }

  drawES5Chart({
    cords: { x: 18, y: 30 },
    radius: 30
  });
```

ES6版本
```javascript
  function drawES6Chart({size = 'big',cords = {x:0,y:0},radius = 25} = {}){
    console.log(size,cords,radius);
  }

  drawES6Chart({
    cords: {x:18,y:30},
    radius: 30
  })
```

### 解构嵌套对象和数组
```javascript
  var metadata = {
    title: "Scratchpad",
    translations: [
      {
        locale: "de",
        localization_tags: [ ],
        last_edit: "2014-04-14T08:43:37",
        url: "/de/docs/Tools/Scratchpad",
        title: "JavaScript-Umgebung"
      }
    ],
    url: "/en-US/docs/Tools/Scratchpad"
  };

  var { title: englishTitle, translations: [{ title: localeTitle }] } = metadata;

  console.log(englishTitle); // "Scratchpad"
  console.log(localeTitle);  // "JavaScript-Umgebung"
```

### For of迭代和解构
```javascript
  var people = [
    {
      name: "Mike Smith",
      family: {
        mother: "Jane Smith",
        father: "Harry Smith",
        sister: "Samantha Smith"
      },
      age: 35
    },
    {
      name: "Tom Jones",
      family: {
        mother: "Norah Jones",
        father: "Richard Jones",
        brother: "Howard Jones"
      },
      age: 25
    }
  ];

  for(var {name:n,family:{ father:f }} of people){
    console.log("Name: " + n + ",Father: " + f);
  }

  // "Name: Mike Smith, Father: Harry Smith"
  // "Name: Tom Jones, Father: Richard Jones"
```

### 从作为函数实参的对象中提取数据
```javascript
  function userId({id}) {
    return id;
  }

  function whois({displayName: displayName, fullName: {firstName: name}}){
    console.log(displayName + " is " + name);
  }

  var user = {
    id: 42,
    displayName: "jdoe",
    fullName: {
        firstName: "John",
        lastName: "Doe"
    }
  };

  console.log("userId: " + userId(user)); // "userId: 42"
  whois(user); // "jdoe is John"
```
这段代码从user对象中提取并输出id、displayName和firstName。

### 对象属性计算名和解构
计算属性名，如object literals，可以被解构。
```javascript
  let key = "z";
  let { [key]:foo } = { z:"bar" };

  console.log(foo); //"bar"
```

# Default + Rest + Spread
## 默认参数
如果一个参数没有被传入对应的实参或者传入了undefined，则该形参会被赋一个默认值。现在我们可以在定义函数的时候指定参数的默认值，而不用像以前那样通过逻辑或操作符来达到目的了。
```javascript
  function sayHello(name){
    //传统的指定默认参数的方式
    var name = name || 'coffee';
    console.log('Hello ' + name);
  }

  //运用ES6的默认参数
  function sayHello2(name = 'coffee'){
    console.log(`Hello ${name}`);
  }

  sayHello(); //输出：Hello coffee
  sayHello('gaoquan'); //输出：Hello gaoquan
  sayHello2(); //输出：Hello coffee
  sayHello2('gaoquan'); //输出：Hello gaoquan
```

## Rest剩余参数
在函数被调用时，剩余参数表示为一个数组名，该数组包含了那些没有对应形参的，长度不确定的剩余实参。

### 语法
```javascript
  function (a,b, ...theArgs){
    //...
  }
```

### 简述
如果一个函数的最后一个形参是以`...`为前缀的，则在函数被调用时，该形参会成为一个数组，数组中的元素都是传递给该函数的多出来的实参的值。

在上例中，theArgs会包含传递给函数的从第三个实参开始到最后所有的实参(第一个实参映射到a，第二个实参映射到b).

#### 剩余参数和arguments对象之间的区别
剩余参数和arguments对象之间的区别主要有三个：
 - 剩余参数只包含那些没有对应形参的实参，而arguments对象包含了传给函数的所有实参。
 - arguments对象不是一个真实的数组，而剩余参数是真实的Array实例，也就是说你能够在它上面直接使用所有的数组方法，比如sort、forEach、pop
 - arguments对象还有一些附加的属性(比如callee属性)

#### arguments对象转换为剩余参数
使用剩余参数可以避免将arguments转为数组的麻烦.
```javascript
  // 下面的代码模拟了剩余数组
  function f(a,b){
    var args = Array.prototype.slice.call(arguments,f.length);
    // ...
  }

  //现在代码可以简化为这样了
  function(a,b,...args){
    // ...
  }
```

### 例子
因为theArgs是个数组，所以你可以使用length属性得到剩余参数的个数：
```javascript
  function fun1(...theArgs){
    alert(theArgs.length);
  }

  fun1(); //弹出"0"，因为theArgs没有元素
  fun1(5); //弹出"1"，因为theArgs只有一个元素
  fun1(5,6,7); //弹出"3"，因为theArgs有三个元素
```

下例中，剩余参数包含了从第二个到最后的所有实参，然后用第一个实参依次乘以它们：
```javascript
  function multiply(multiplier,...theArgs){
    return theArgs.map(function(element){
      return multiplier * element;
    });
  }

  var arr = multiply(2,1,2,3);
  console.log(arr); //弹出“2,4,6”
```

下例演示了你可以在剩余参数上使用任意的数组方法，而arguments对象不可以：
```javascript
  function sortRestArgs(...theArgs){
    var sortedArgs = theArgs.sort();
    return sortedArgs;
  }

  alert(sortRestArgs(5,3,7,1)); //弹出 1,3,5,7

  function sortArguments() {
    var sortedArgs = arguments.sort();
    return sortedArgs; // 不会执行到这里
  }

  alert(sortArguments(5,3,7,1)); // 抛出TypeError异常:arguments.sort is not a function
```
如果想在arguments对象上使用数组方法，你首先得将它转换为真实的数组，比如使用 `[].slice.call(arguments)`

## 展开运算符
### 概述
展开运算符允许一个表达式在某处展开，在多个参数(用于函数调用)或者多个元素(用于数组字面量)或者多个变量(用于解构赋值)的地方就会这样。

### 语法
用于函数调用：
```javascript
  myFunction(...iterableObj);
```

用于数组字面量：
```javascript
  [...iterableObj,4,5,6]
```

用于解构赋值：
```javascript
  [a,b,...iterableObj] = [1,2,3,4,5];
```

### 例子
#### 更好的apply方法
**例子：** 目前为止，我们都是使用`Function.prototype.apply`方法来将一个数组展开成多个参数：
```javascript
  function myFunction(x,y,z){ }
  var args = [0,1,2];
  myFunction.apply(null,args);
```

如果使用了ES6的展开运算符，你可以这么写：
```javascript
  function myFunction(x,y,z){ }
  var args = [0,1,2];
  myFunction(...args);
```

还可以使用多个数组：
```javascript
  function myFunction(v,w,x,y,z){ }
  var args = [0,1];
  myFunction(-1,...args,2,...[3]);
```

#### 更强大的数组字面量
**例子：** 目前为止，如果你想创建一个包含某些已有数组里的元素的新数组，通常会用到push、splice、concat等数组方法。有了新的展开运算符，可以这样写：
```javascript
  var parts = ['shoulder','knees'];
  var lyrics = ['head', ...parts, 'and', 'toes']; // ["head", "shoulders", "knees", "and", "toes"]
```
和函数调用一样，数组字面量中也可以使用...多次。

#### 配合new运算符
**例子：** 在ES5中，我们无法同时使用new运算符合apply方法(apply方法调用[[Call]]而不是[[Construct]])。在ES6中，我们可以使用展开运算符，和普通的函数调用一样。
```javascript
  var dataFields = readDateFields(database);
  var d = new Date(...dataFields);
```

#### 更好的push方法
**例子：** 在ES5中，我们可以使用push方法将一个数组添加到另一个数组的末尾：
```javascript
  var arr1 = [0,1,2];
  var arr2 = [3,4,5];
  //将arr2中的所有元素添加到arr1中
  Array.prototype.push.apply(arr1,arr2);
```

在ES6中，可以这么写：
```javascript
  var arr1 = [0, 1, 2];
  var arr2 = [3, 4, 5];
  arr1.push(...arr2);
```

#### 将类数组对象转换成数组
展开操作可以将一个类数组对象中索引范围在[0,length)之间的所有属性的值添加到一个数组中，这样就可以得到一个真正的数组：
```javascript
  var nodeList = document.querySelectorAll('div');
  var array = [...nodeList];
```
