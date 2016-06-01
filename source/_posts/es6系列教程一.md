---
title: es6系列教程一
date: 2016-06-01T17:03:45.000Z
tags:
  - es6
  - javascript
categories: 原创
---

> Our lives stretched out ahead of us, like a perpetual sunrise.

# let
开门见山我们先来尝尝一个例子：

```javascript
  var list = document.getElementById("list");

  for (var i = 1; i <= 5; i++) {
    var item = document.createElement("li");
    item.appendChild(document.createTextNode("Item " + i));

    var j = i; //注意这里使用的是var

    item.onclick = function (ev) {
       console.log("Item " + j + " is clicked.");
     };
    list.appendChild(item);
  }
```
上面这段代码的意图是创建5个li，并且点击它们的时候能够打印出当前的序号。但是由于使用了var定义j变量，所以我们发现打印出来的结果都是`Item 5 is clicked.`,这是因为这里var定义的j是函数级变量，5个内部函数都指向了同一个j，而j最后一次赋值是5.而我们流弊的前辈们变相的引入了块级作用域，并轻松解决了上面的问题，具体如下：

```javascript
var list = document.getElementById("list");

for (var i = 1; i <= 5; i++) {
  var item = document.createElement("li");
  item.appendChild(document.createTextNode("Item " + i));

  (function(i){
    var j = i;
    item.onclick = function (ev) {
       console.log("Item " + j + " is clicked.");
     };
  })(i)

  list.appendChild(item);
}
```
这样的话，问题就迎刃而解。因为每循环一次都会产生新的块级作用域(也就是形如`(function(){})()`这样的写法，没进入一次就生成新的跨级作用域)，在新的块级作用域中5个内部函数指向了不同的j。

现在，该我们的主角出场了。为了实现块级作用域，es6引入了let声明变量的方法，这样的话上面的例子，可以直接简化成：

```javascript
var list = document.getElementById("list");

for (var i = 1; i <= 5; i++) {
  var item = document.createElement("li");
  item.appendChild(document.createTextNode("Item " + i));

  let j = i; //这里直接使用let

  item.onclick = function (ev) {
     console.log("Item " + j + " is clicked.");
   };
  list.appendChild(item);
}
```

由此可以看出let与var的区别，let允许把变量限制在块级域中，而var申明的变量要么是全局的，要么是函数级的，而无法是块级的，这是其中之一的区别。

## 作用域规则
下面我们看看let和var在作用域上规则上的区别。用let定义的变量的作用域是定义在它们的块内，以及包含在这个块中的子块；而var定义的变量的作用域是定义在它们的函数内：

```javascript
  function varTest(){
    var x = 31;
    if(true){
      var x = 71; //相同的变量
      console.log(x); //71
    }
    console.log(x); //71    
  }

  function letTest(){
    let x = 31;
    if(true){
      let x = 71; //不同的变量
      console.log(x); //71
    }
    console.log(x); //31
  }
```

在程序或函数的顶层，let的表现就像var一样：

```javascript
  var x = 'global';
  let y = 'global';
  console.log(this.x);
  console.log(this.y);  
```
上面这段代码的运行后会显示两次"global"。

## let的暂存死区与错误
在同一个函数或同一个作用域中用let重复定义一个变量将引起TypeError

```javascript
  if(x){
    let foo;
    let foo; //TypeError
  }
```

在es6中，let将会提升这个变量到语句块的顶部。然而，在这个语句块中，在变量声明之前引用这个变量会导致一个ReferenceError的结果，因为let变量在“暂存死区”（从块的开始到声明这段）。

```javascript
  function do_something(){
    console.log(foo); //ReferenceError
    let foo = 2;
  }
```

在switch申明中你可能会遇到这样的错误，因为一个switch只有一个作用快。

```javascript
  switch(x){
    case 0:
      let foo;
      break;
    case 1:
      let foo; //TypeError for redeclaration
      break;
  }
```

## 循环定义中的let作用域
循环体中是可以引用在for申明时用let定义的变量，尽管let不是出现在大括号之间。

```javascript
  var i = 0;
  for(let i=i; i< 10; i++){
    console.log(i);
  }
```

```javascript
  注：以上let申明的i将会变成undefined; chrome版本50.0.2661.102(64-bit); 推荐一下写法：
  var i = 0;
  for(let l=i;l<10;l++){
    console.log(l);
  }
```

域作用规则
`for (let expr1;expr2;expr3) statement`
在这个例子中，expr2,expr3和statement都是包含在一个隐含域块中，其中也包含了expr1

## 再看些例子
### let 对比 var
let的作用域是块，var的作用域是函数

```javascript
  var a = 5;
  var b = 10;

  if(a===5){
    let a = 4; //The scope is inside the if-block
    var b = 1; //The scope is inside the function

    console.log(a); //4
    console.log(b); //1
  }

  console.log(a); //5
  console.log(b); //1
```

### let在循环中
可以用let来代替var，在for定义块中使用块级变量

```javascript
  for(let i=0;i<10;i++){
    console.log(i); // 0,1,2,3,4,5,6,7,8,9
  }
  console.log(i); //i is not defined
```

#const
const声明创建一个只读的常量。这不意味着常量指向的值不可变，而是变量标识符的只能赋值一次。这个声明创建了一个常量，可以在全局作用域或者函数内声明常量，常量需要被初始化。也就是说，在定义常量的同时必须初始化(这是有意义的，鉴于变量的值在初始化后就不能改变)。

常量拥有块作用域，和使用let 定义的变量十分相似。常量的值不能通过再赋值改变，也不能再次声明。

一个常量不能和它所在作用域内的其他变量或函数拥有相同的名称。

## 例子
下面的例子演示了常量的特征。
```javascript
  // 注意: 常量在声明的时候可以使用大小写，但通常情况下会使用全部大写英文。

  // 定义常量MY_FAV并赋值7
  const MY_FAV = 7;

  // 在 Firefox 和 Chrome 这会失败但不会报错(在 Safari这个赋值会成功)
  MY_FAV = 20;

  // 输出 7
  console.log("my favorite number is: " + MY_FAV);

  // 尝试重新声明会报错
  const MY_FAV = 20;

  //  MY_FAV 保留给上面的常量，这个操作会失败
  var MY_FAV = 20;

  // MY_FAV 依旧为7
  console.log("my favorite number is " + MY_FAV);

  // 下面是一个语法错误
  const A = 1; A = 2;

  // 常量要求一个初始值
  const FOO; // SyntaxError: missing = in const declaration

  // 常量可以定义成对象
  const MY_OBJECT = {"key": "value"};

  // 重写对象和上面一样会失败
  MY_OBJECT = {"OTHER_KEY": "value"};

  // 对象属性并不在保护的范围内，下面这个声明会成功执行
  MY_OBJECT.key = "otherValue";
```

# 箭头函数
箭头函数就是个简写形式的函数表达式，并且它拥有词法作用域的this值（即不会新产生自己作用域下的this,arguments,super和new.target等对象）。此外，箭头函数时匿名的。
## 基本语法
```javascript
  (param1,param2,...,paramN) => { statements }
  (param1,param2,...,paramN) => expression
          // equivalent to => { return expression; }

  //如果只有一个参数，圆括号是可选的
  (singleParam) => { statements }
  singleParam => { statements }

  //无参数的函数需要使用圆括号
  () => { statements }
```

## 高级语法
```javascript
  //返回对象字面量时应该用圆括号将其包起来：
  params => {foo:bar}

  //支持Rest parameters 和 default parameters:
  (param1,param2,...rest) => { statements }
  (param1 = defaultValue1,param2,...,paramN = defaultValueN) => { statements }

  //Destructring within the parameter list is also supported
  var f = ([a,b] = [1,2],{x:c} = {x:a+b}) => a + b +c;
  f(); //6
```

## 描述
箭头函数的引用有两个方面的影响：一是更简短的函数书写，二是对this的词法解析。

### 更短的函数
在一些函数式编程模式里，更短的函数书写方式很受欢迎。试比较：
```javascript
  var a = [
    "Hydrogen",
    "Helium",
    "Lithium",
    "Beryl­lium"
  ];
  var a2 = a.map(function(s){return s.length});

  var a3 = a.map(s => s.length);
```

### this的词法
在箭头函数出现之前，每个新定义的函数都有其自己的this值（例如，构造函数的this指向了一个新的对象；严格模式下的函数的this值为undefined；如果函数是作为对象的方法被调用的，则其this指向了那个调用它的对象）。在面向对象风格的编程中，这被证明是非常恼人的事情。

```javascript
  function Person(){
    //构造函数Person()定义的`this`就是新实例对象自己
    this.age = 0;
    setInterval(function growUp(){
      //在非严格模式下，growUp()函数定义了其内部的`this`
      //在全局对象，不同于构造函数Person()的定义的`this`
      this.age ++;
    },1000)
  }

  var p = new Person();
```

在ECMAScript3/5中，这个问题可以通过新增一个变量来指向期望的this对象，然后将该变量放到闭包中来解决。

```javascript
  function Person(){
    var self = this;
    self.age = 0;

    setInterval(function growUp(){
      //回调里面的`self`变量就指向了期望的那个对象了
      self.age ++;
    },1000)
  }
```
除此之外，还可以使用bound function,把期望的this值传递给growUp()函数。

箭头函数则会捕获其所在上下文的this值，作为自己的this值，因此下面的代码将如运行。

```javascript
  function Person(){
    this.age = 0;

    setInterval(() => {
      this.age ++; //this正确地指向了person对象
    },1000)
  }
```

### 与strict mode的关系
考虑到this是词法层面上的，严格模式中与this相关的规则都将被忽略。

```javascript
  var f = () => {
    `use strict`;
    return this;
  }
  f() === window //或全局对象
```

### 使用call或apply调用
由于this已经在词法层面完成了bound，通过call()或apply()方法调用一个函数时，只是传入了参数而已，对this并没有影响。

```javascript
  var adder = {
    base: 1,

    add: function(a){
      var f = v => v + this.base;
      return f(a);
    },

    addThruCall: function(a){
      var f = v => v + this.base;
      var b = {
        base : 2
      };

      return f.call(b,a)
    }
  };

  console.log(adder.add(1)); //输出2
  console.log(adder.addThruCall(1)); //仍然输出2
```

## arguments的词法
箭头函数不会在其内部暴露arguments对象：arguments.length,arguments[0],arguments[1]等等，
都不会指向箭头函数arguments，而是指向了箭头函数所在作用域的一个名为arguments的值（如果真的有的话，否则，就是undefined）

```javascript
  var arguments = 42;
  var arr = () => arguments;

  arr(); //42

  function foo(){
    var f = () => arguments[0];
    return f(2);
  }

  foo(1); //1
```
箭头函数没有自己的arguments对象，不过大多数情况下，rest参数可以给出一个解决方案：

```javascript
  function foo(){
    var f = (...args) => args[0];
    return f(2);
  }

  foo(1); //2
```

### 使用yield关键字
yield关键字通常不在箭头函数中使用。因此，箭头函数不能用作Generator函数

### 返回对象字面量
请牢记，用params => {object:literal}这种简单的语法返回一个对象字面量是行不通的：

```javascript
  var func = () => { foo: 1 };
  //calling func() returns undefined!

  var func = () => { foo: function(){} };
  //SyntaxError: function statement required a name
```

这是因为花括号(即{})里面的代码被解析为声明序列了(例如，foo被认为是一个label，而非对象字面量里的键)。

所以，记得用圆括号把对象字面量包起来：
`var func = () => ({foo:1})`

## 示例
```javascript
  //一个空箭头函数，返回undefined
  let empty = () => {};

  (() => "foobar")() //返回"foobar"

  var simple = a => a > 15 ? 15 : a;
  simple(16); //15
  simple(10); //10

  let max = (a,b) => a > b ? a : b;

  //Easy array filtering, mapping, ...
  var arr = [5,6,13,0,1,18,23];
  var sum = arr.reduce((a,b) => a + b); //66
  var even = arr.filter(v => v % 2 == 0); //[6,0,18]
  var double = arr.map(v => v * 2); //[10,12,26,0,2,36,46]
```
