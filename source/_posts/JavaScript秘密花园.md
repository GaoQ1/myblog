---
title: JavaScript秘密花园
date: 2016-08-05 16:22:23
tags:
- javascript
categories: 笔记
---
> Laughing is the most touching mask.

## 对象

### 对象使用和属性
JavaScript中所有变量都可以当作对象使用，除了两个例外`null`和`undefined`。
```javascript
  false.toString(); //'false'
  [1,2,3].toString(); //'1,2,3'

  function Foo(){}
  Foo.bar = 1;
  Foo.bar; //1
```
一个常见的误解是数字的字面值(literal)不能当作对象使用。这是因为JavaScript解析器的一个错误，它试图将点操作符解析为浮点数字面值的一部分。
```javascript
  2.toString(); //SyntaxError
```
有很多变通方法可以让数字的字面值看起来像对象。
```javascript
  2..toString(); //第二个点号可以正常解析
  2 .toString(); //注意点号前面的空格
  (2).toString(); //2先被计算
```

### 对象作为数据类型
JavaScript的对象可以作为哈希值使用，主要用来保存命名的键与值的对应关系。
使用对象的字面语法- {} -可以创建一个简单对象。这个新创建的对象从`Object.prototye`继承下来，没有任何自定义属性。
```javascript
  var foo = {}; //一个空对象
  //一个新对象，拥有一个值为12的自定义属性'test'
  var bar = {test: 12};
```

### 访问属性
有两种方式来访问对象的属性，点操作符或者中括号操作符。
```javascript
  var foo = {name: 'kitten'};
  foo.name; // kitten
  foo['name']; // kitten

  var get = 'name';
  foo[get]; // kitten

  foo.1234; // SyntaxError
  foo['1234']; // works
```
两种语法是等价的，但是中括号操作符在下面两种情况下依然有效
 - 动态设置属性
 - 属性名不是一个有效的变量名(比如属性名中包含空格，或者属性名是JS的关键词)

### 删除属性
删除属性的唯一方法是使用`delete`操作符;设置属性为`undefined`或者`null`并不能真正的删除属性，而仅仅是移除了属性和值的关联。
```javascript
  var obj = {bar: 1, foo:2, baz:3};
  obj.bar = undefined;
  obj.foo = null;
  delete obj.baz;

  for(var i in obj){
    if(obj.hasOwnProperty(i)){
      console.log(i, ''+obj[i]);
    }
  }
```
上面的输出的结果有`bar undefined`和`foo null`只有`baz`被真正的删除了，所以从输出结果中消失。

### 属性名的语法
```javascript
  var test = {
    'case': 'I am a keyword so I must be notated as a string',
    delete: 'I am a keyword too so me' // 出错：SyntaxError
  }
```
对象的属性名可以使用字符串或者普通字符声明。但是由于javascript解析器的另一个错误设计，上面的第二种声明方式在ECMAScript5之前会抛出`SyntaxError`的错误。
这个错误的原因 是delete是javascript语言的一个关键词；因此为了在更低版本的javascript引擎下也能运行，必须使用字符串字面值声明方式。

### 原型
JavaScript不包含传统的类继承模型，而是使用prototype原型模型。
虽然这经常被当作是JavaScript的缺点被提及，其实基于原型的继承模型比传统类继承还要强大。实现传统的类继承模型是很简单，但是实现JavaScript中的原型继承则要困难的多。
由于JavaScript是唯一一个被广泛使用的基于原型继承的语言，所以理解两种继承模式的差异是需要一定时间的。

第一个不同之处在于JavaScript使用原型链的继承方式。
```javascript
  function Foo(){
    this.value = 42;
  }
  Foo.prototype = {
    method: function(){}
  };

  function Bar(){}

  //设置Bar的prototype属性为Foo的实例对象
  Bar.prototype = new Foo();
  Bar.prototype.foo = 'Hello World';

  //修正Bar.prototype.constructor为Bar本身
  Bar.prototype.constructor = Bar;

  var test = new Bar() // 创建Bar的一个新实例

  //原型链
  test [Bar的实例]
    Bar.prototype [Foo的实例]
      {foo: 'Hello World'}
      Foo.prototype
        {method: ...};
        Object.prototype
          {toString: ...}
```
**注意：** 简单的使用`Bar.prototype = Foo.prototype`将会导致两个对象共享相同的原型。因此，改变任意一个对象的原型都会影响到另一个对象的原型，在大多数情况下这不是希望的结果。不要使用`Bar.prototype = Foo`，因为这不会执行`Foo`的原型，而是指向函数`Foo`。因此原型链将会回溯到`Function.prototype`而不是`Foo.prototype`，因此`method`将不会在`Bar`的原型链上。

上面的例子中，`test`对象从`Bar.prototype`和`Foo.prototype`继承下来;因此，它能访问`Foo`的原型方法`method`。同时，它也能够访问那个定义在原型上的`Foo`实例属性`value`。需要注意的是`new Bar()`不会创造出一个新的`Foo`实例，而是重复使用它原型上的那个实例；因此，所有的`Bar`实例都会共享相同的`value`属性。

### 属性查找
当查找一个对象的属性时，JavaScript会向上遍历原型链，直到找到给定名称的属性为止。
到查找到达原型链的顶部-也就是`Object.prototype`-但是仍然没有找到指定的属性，就会返回`undefined`。

### 原型属性
当原型属性用来创建原型链时，可以把任何类型的值赋给它(prototype)。然而将原子类型赋给prototype的操作将会被忽略。
```javascript
  function Foo(){}
  Foo.prototype = 1; //无效
```
而将对象赋值给prototype,正如上面的例子所示，将会动态的创建原型链。

### 性能
如果一个属性在原型链的上端，则对于查找时间将带来不利影响。特别的，试图获取一个不存在的属性将会遍历整个原型链。
并且，当使用`for in`循环遍历对象属性时，原型链上的所有的属性都将被访问。

### 扩展内置类型的原型
一个错误特性被经常使用，那就是拓展`Object.prototype`或者其他内置类型的原型对象。
这种技术被称之为`mobkey patching`并且会破坏封装。虽然它被广泛的应用到一些JavaScript类库中，比如`Prototype`，但是为内置类型添加一些非标准的函数仍然不是一个好主意。
扩展内置类型的唯一理由是为了和新的JavaScript保持一致，比如`Array.forEach`.

### 总结
在写复杂的JavaScript应用之前，充分理解原型链继承的工作方式是每个JavaScript程序员必修的功课。要提防原型链过长带来的性能问题，并指导如何通过缩短原型链来提高性能。更进一步，绝对不要扩展内置类型的原型，除非是为了和新的JavaScript引擎兼容。

### hasOwnProperty函数
为了判断一个对象是否包含自定义属性而不是原型链上的属性，我们需要使用继承自`Object.prototype`的`hasOwnProperty`方法。
`hasOwnProperty`是JavaScript中唯一一个处理属性但是不查找原型链的函数。
**注意：** 通过判断一个属性是否undefined是不够的。因为一个属性可能确实存在，只不过它的值被设置为undefined。
```javascript
  //修改Object.prototype
  Object.prototype.bar = 1;
  var foo = {goo: undefined};

  foo.bar; //1
  'bar' in foo; //true

  foo.hasOwnProperty('bar'); //false
  foo.hasOwnProperty('goo'); //true
```
只有`hasOwnProperty`可以给出正确和期望的结果，这在遍历对象的属性时会很有用。没有其他方法可以用来排除原型链上的属性，而不是定义在对象自身上的属性。

**hasOwnProperty** 作为属性
`Javascript`不会保护`hasOwnProperty`被非法占用，因此如果一个对象碰巧存在这个属性，就需要使用外部的`hasOwnProperty`函数来获取正确的结果。
```javascript
  var foo = {
    hasOwnProperty: function(){
      return false;
    },
    bar: 'Here be dragons'
  };

  foo.hasOwnProperty('bar') // 总是返回false

  //使用其他对象的hasOwnProperty，并将其上下文设置为foo
  ({}).hasOwnProperty.call(foo,'bar'); //true
```

当检查对象上某个属性是否存在时，`hasOwnProperty`是唯一可用的方法。同时在使用`for in`loop遍历对象时，推荐总是使用`hasOwnProperty`方法，这将会避免原型对象扩展带来的干扰。

### for in循环
和`in`操作符一样，`for in`循环同样在查找对象属性时遍历原型链上的所有的属性。
**注意：** `for in`循环不会遍历那些`enumerable`设置为`false`的属性；比如数组的`length`属性。
```javascript
  //修改Object.prototype
  Obkect.prototype.bar = 1;

  var foo = {moo: 2};
  for(var i in foo){
    console.log(i); //输出两个属性： bar和moo
  }
```
由于不可能改变`for in`自身的行为，因此有必要过滤那些不希望出现在循环体中的属性，这可以通过`Object.prototype`原型上的额`hasOwnProperty`函数来完成。

### 使用hasOwnProperty过滤
**注意：** 由于`for in`总是要遍历整个原型链，因此如果一个对象的继承层次太深的话会影响性能。
```javascript
  //foo 变量是上例中的
  for(var i in foo){
    if(foo.hasOwnProperty(i)){
      console.log(i);
    }
  }
```
这个版本的代码是唯一正确的写法。由于我们使用`hasOwnProperty`，所以这次只输出`moo`。如果不使用`hasOwnProperty`，则这段代码在原生对象原型(比如`Object.prototype`)被扩展时可能会出错。
推荐总是使用`hasOwnProperty`。不要对代码运行的环境做任何假设，不要假设原生对象是否已经被扩展了。

## 函数
### 函数声明与表达式
函数是javascript中的一等对象，这意味着可以把函数像其他值一样传递。一个常见的用法是把匿名函数作为回调函数传递到异步函数中。

**函数声明**
```javascript
  function foo(){}
```
上面的方法会被在执行前被解析(hoisted)，因此它存在于当前上下文的任意一个地方，即使在函数定义体的上面被调用也是对的。
```javascript
  foo(); //正常运行，因为foo在代码前已经被创建
  function foo(){}
```

**函数赋值表达式**
```javascript
  var foo = function(){}
```
这个例子把一个匿名函数赋值给变量`foo`。
```javascript
  foo; //'undefined'
  foo(); //Error: TypeError
  var foo = function(){}
```
由于`var`定义了一个声明语句，对变量`foo`的解析是在代码运行之前，因此`foo`变量在代码运行时已经被定义过了。
但是由于赋值语句只在运行时执行，因此在相应代码执行之前，`foo`的值缺省为`undefined`。

**命名函数的赋值表达式**
另一个特殊的情况是将命名函数赋值给一个变量
```javascript
  var foo = function bar(){
    bar(); //正常运行
  }
  bar(); //Error: ReferenceError
```
bar函数声明外是不可见的，这是因为我们已经把函数值赋值给了foo；然而在bar内部依然可见。这是由于JavaScript的命名处理所致，函数名在函数内总是可见的。
**注意：** 在IE8及IE8以下版本浏览器bar在外部也是可见的，是因为浏览器对命名函数赋值表达式进行了错误的解析，解析成两个函数`foo`和`bar`

### this的工作原理
JavaScript有一套完全不同于其他语言的对`this`的处理机制。在五种不同的情况下，this指向的各不相同。

**全局范围内**
> this  

当在全局范围内使用this，它将会指向全局对象。

**函数调用**
> foo();

这里的this也会指向全局对象。

**方法调用**
> test.foo();

这个例子中，this指向test对象。*ES5注意：* 在严格模式下，不存在全局变量，这种情况下this将会是undefined。

**调用构造函数**
> new foo();

如果函数倾向于和`new`关键词一块使用，则我们称这个函数是 *构造函数* 。在函数内部，`this`指向新创建的对象。

**显示的设置this**
> function foo(a,b,c){}
  var bar = {};
  foo.apply(bar,[1,2,3]); //数组将会被扩展，如下所示
  foo.call(bar,1,2,3); //传递到foo的参数是：a = 1,b = 2,c = 3

当使用`Function.prototype`上的call或者apply方法时，函数内的this将会被显示设置为函数调用的第一个参数。
因此函数调用的规则在上例中已经不使用了，在foo函数内this被设置成了bar。

**常见误解**
尽管大部分的情况都说的过去，不过第二个规则被认为是JavaScript语言另一个错误设计的地方，因为它从来就没有实际的用途。
```javaScript
  Foo.method = function(){
    function test(){
      // this将会被设置为全局对象(浏览器环境中也就是window对象)
    }
    test();
  }
```
**注意：** 在对象的字面声明语法中，this不能用来指向对象本身。因此`var obj = {me: this}`中的me不会指向obj，因为this只可能出现在上述的五种情况中。在这个例子中，如果是浏览器中运行，obj.me等于window对象。

一个常见的误解是`test`中的`this`将会指向`Foo`对象，实际上不是这样子的。
为了在`test`中获取对`Foo`对象的引用，我们需要在`method`函数内部创建一个局部变量指向`Foo`对象。
```javaScript
  Foo.method = function(){
    var that = this;
    function test(){
      //使用that来指向Foo对象
    }
    test();
  }
```
that只是我们随意起的名字，不过这个名字被广泛的用来指向外部的this对象。在闭包一节，我们可以看到that可以作为参数传递。

### 方法的赋值表达式
另一个看起来奇怪的地方是函数别名，也就是将一个方法赋值给一个变量。
```javaScript
  var test = someObject.methodTest;
  test();
```
上例中，test就像一个普通的函数被调用；因此，函数内的this将不再被指向到`someObject`对象。
虽然this的晚绑定特性似乎并不友好，但这确实是基于原型继承赖以生存的土壤。
```javaScript
  function Foo(){}
  Foo.prototype.method = function(){}

  function Bar(){}
  Bar.prototype = Foo.prototype;

  new Bar().method();
```
让method被调用时，this将会指向Bar的实例对象。

### 闭包和引用
闭包是JavaScript一个非常重要的特性，这意味着当前作用域总是能够访问外部作用域中的变量。因为 **函数** 是JavaScript中唯一拥有自身作用域的结构，因此闭包的创建依赖于函数。
**模拟私有变量**
```javaScript
  function Counter(start){
    var count = start;
    return {
      increment: function(){
        count++;
      },
      get: function(){
        return count;
      }
    }
  }

  var foo = Counter(4);
  foo.increment();
  foo.get(); //5
```
这里,`COunter`函数返回两个闭包，函数`increment`和函数`get`。这两个函数都维持着对外部作用域`Counter`的引用，因此总可以访问此作用域内定义的变量`count`。

**为什么不可以在外部访问私有变量**
因为JavaScript中不可以对作用域进行引用或赋值，因此没有办法在外部访问count变量。唯一的途径就是通过那两个闭包。
```javaScript
  var foo = new Counter(4);
  foo.hack = function(){
    count = 1337;
  }
```
上面的代码不会改变定义在Counter作用域中的count变量的值，因为foo.hack没有定义在那个作用域内。它将会创建或者覆盖全局变量count。

**循环中的闭包**
一个常见的错误出现在循环中使用闭包，假设我们需要在每次循环中调用循环序号
```javaScript
  for(var i=0;i<1-;i++){
    setTimeout(function(){
      console.log(i)
    },1000)
  }
```
上面的代码不会输出数字0到9，而是会输出数字10十次。
当console.log被调用的时候，匿名函数保持对外部变量i的引用，此时for循环已经结束，i的值被修改成了10.
为了得到想要的结果，需要在每次循环中创建变量i的拷贝。

**避免引用错误**
为了正确的获得循环序号，最好使用 *匿名包装器* (就是我们通常说的自执行匿名函数)
```javaScript
  for(var i=0;i<10;i++){
    (function(e){
      setTimeout(function(){
        console.log(e)
      },1000)
    })(i)
  }
```
外部匿名函数会立即执行，并把i作为它的参数，此时函数内e变量就拥有了i的一个拷贝。
当传递给 setTimeout 的匿名函数执行时，它就拥有了对 e 的引用，而这个值是不会被循环改变的。

有另一个方法完成同样的工作，那就是从匿名包装器中返回一个函数。这和上面的代码效果一样。
```javaScript
  for(var i=0;i<10;i++){
    setTimeout((function(e){
      return function(){
        console.log(e)
      }
    })(i),1000)
  }
```

### arguments对象
Javascript中每个函数内都能访问一个特别变量`arguments`。这个变量维持着所有传递到这个函数中的参数列表。
arguments变量不是一个数组(Array)。尽管在语法上它有数组相关的属性length，但它不从Array.prototype继承，实际上它是一个对象(Object)。
**注意：** 由于`arguments`已经被定义为函数内的一个变量。因此通过`var`关键字定义`arguments`或者将`arguments`声明为一个形式参数，都将导致原生的`arguments`不会被创建。

因此，无法对`arguments`变量使用标准的数组方法，比如push、pop或者slice。虽然使用for循环遍历也是可以的，但是为了更好的使用数组方法，最好把它转化为一个真正的数组。

**转化为数组**
下面的代码将会创建一个新的数组，包含所有arguments对象中的元素。
```javaScript
  Array.prototype.slice.call(arguments);
```
这个转化比较慢，在性能不好的代码中不推荐这种做法。

**传递参数**
下面是将参数从一个函数传递到另一个函数的推荐做法。
```javaScript
  function foo(){
    bar.apply(null, arguments);
  }
  function bar(a,b,c){
    //...
  }
```

另一个技巧是同时使用call和apply，创建一个快速的解绑定包装器。
```javaScript
  function Foo(){}

  Foo.prototype.method = function(a,b,c){
    console.log(this,a,b,c);
  }
  //创建一个解绑定的'method'
  //输入参数为:this, arg1, arg2, ...argN
  Foo.method = function(){
    //结果： Foo.prototype.method.call(this,arg1,arg2...argN)
    Function.call.apply(Foo.prototype.method,arguments);
  }
```
上面的Foo.method函数和下面代码的效果是一样的：
```javaScript
  Foo.method = function(){
    var args = Array.prototype.slice.call(arguments);
    Foo.prototype.method.apply(args[0],args.slice(1));
  }
```

**自动更新**
`arguments`对象为其内部属性以及函数形式参数创建getter和setter方法。
因此，改变形参的值会影响到`arguments`对象的值，反之亦然。
```javaScript
  function foo(a,b,c){
    arguments[0] = 2;
    a; //2

    b = 4;
    arguments[1]; //4

    var d = c;
    d = 9;
    c; //3
  }
  foo(1,2,3)；
```

**性能真相**
不管它是否有被使用，arguments对象总会被创建，除了两个特殊情况 - 作为局部变量声明和作为形式参数。
arguments的getters和setters方法总会被创建；因此使用arguments对性能不会有什么影响。除非是需要对`arguments`对象的属性进行多次访问。

在严格模式下，arguments的描述有助于我们的理解，请看下面的代码：
**注意：** ES5提示，这些getters和setters在严格模式下不会被创建。
```javaScript
  //阐述在ES5的严格模式下`arguments`的特性
  function f(a){
    'use strict'
    a= 42;
    return [a, arguments[0]];
  }
  var pair = f(17);
  console.assert(pair[0] === 42);
  console.assert(pair[1] === 17);
```
然而，的确有一种情况会显著的影响现在JavaScript引擎的性能。这就是使用arguments.callee。
```javaScript
  function foo(){
    arguments.callee; //do something with this function object
    arguments.callee.caller; //and the calling function object
  }

  function bigLoop(){
    for(var i=0;i<100000;i++){
      foo();
    }
  }
```
上面代码中，foo不再是一个单纯的内联函数inlining(指的是解析器可以做内联处理)，因为它需要知道它自己和它的调用者。这不仅抵消了内联函数带来的性能提升，而且破坏了封装，因此现在函数可能要依赖于特定的上下文。

因此强烈建议不要使用`arguments.callee`和它的属性。

### 构造函数
JavaScript中的构造函数和其它语言中的构造函数是不同的。通过new关键字方式调用的函数都被认为是构造函数。

在构造函数内部 - 也就是被调用的函数内 - this指向新创建的对象Object。这个新创建的对象的prototye被指向到构造函数的prototye。

如果被调用的函数没有显式的return表达式，则隐式的会返回this对象 - 也就是新创建的对象。
```javaScript
  function Foo(){
    this.bar = 1;
  }

  Foo.prototype.test = function(){
    console.log(this.bla);
  }

  var test = new Foo();
```
上面代码把Foo作为构造函数调用，并设置新创建对象的prototype为Foo.prototype.

显式的return表达式将会影响返回结果，但仅限于返回的是一个对象。
```javaScript
  function Bar(){
    return 2;
  }
  new Bar(); //返回新创建的对象

  function Test(){
    this.value = 2;

    return {
      foo: 1
    };
  }
  new Test(); //返回的对象
```
new Bar()返回的是新创建的对象，而不是数字的字面值2.因此`new Bar().constructor === Bar`，但是如果返回的是数字对象，结果就不同了，如下所示
```javaScript
  function Bar(){
    return new Number(2)
  }
  new Bar().constructor === Number
```
这里得到的`new Test()`是函数返回的对象，而不是通过new关键字新创建的对象，因此：
```javaScript
  (new Test()).value === undefined
  (new Test()).foo === 1
```
如果`new`被遗漏了，则函数不会返回新创建的对象。
```javaScript
  function Foo(){
    this.bla = 1; //获取设置全局参数
  }
  Foo(); //undefined
```
虽然上例在有些情况下也能正常运行，但是由于JavaScript中this的工作原理，这里的this指向全局对象。

http://bonsaiden.github.io/JavaScript-Garden/zh/#function.constructors
