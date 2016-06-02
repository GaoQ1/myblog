---
title: es6系列教程二
date: 2016-06-02 10:06:25
tags:
  - es6
  - javascript
categories: 原创
---
> It's an imperfect world, but it's the only one we've got.

# Class
从ECMAScript 6开始，JavaScript有了类(class)这个概念。但是需要注意的是，这并不是说：JavaScript从此变得像其他基于面向对象语言一样，有了一种全新的继承模式。JavaScript中的类只是javascript现有的、基于原型的继承模型的一种语法包装(语法糖)，它能让我们用更简洁明了的语法实现继承。

## 定义类
ES6中的类实际上就是个函数，而且正如函数的定义方式有函数声明和函数表达式两种一样，类的定义方式也有两种，分别是：
 - 类声明
 - 类表达式

### 类声明
类声明是定义类的一种方式，就像下面这样，使用class关键字后跟一个类名（这里是Polygon），就可以定义一个类。
```javascript
  class Polygon{
    constructor(height,width){
      this.height = height;
      this.width = width;
    }
  }
```

#### 变量提升
类声明和函数声明不同的一点是，函数声明存在变量提升现象，而类声明不会。也就是说，你必须先声明类，然后才能使用它，否则代码会抛出ReferenceError异常，像下面这样：

```javascript
  var p = new Polygon(); //ReferenceError

  class Polygon{}
```

### 类表达式
类表达式是定义类的另一种方式，就像函数表达式一样，在类表达式中，类名是可有可无的。如果定义了类名，则类名只有在类体内部才能访问。

```javascript
  //匿名类表达式
  var Polygon = class {
    constructor(height,width){
      this.height = height;
      this.width = width;
    }
  };

  //命名类表达式
  var Polygon = class Polygon{
    constructor(height,width){
      this.height = height;
      this.width = width;
    }
  }
```

## 类体和方法定义
类的成员需要定义在一对花括号{}里，花括号里的代码和花括号本身组成了类体。类成员包括类构造器和类方法(包括静态方法和实例方法)。

### 严格模式
类体中的代码都强制在严格模式中执行，即便你没有写"use strict"。

### 构造器(constructor方法)
constructor方法是一个特殊的类方法，它既不是静态方法也不是实例方法，它仅在实例化一个类的时候被调用。一个类只能拥有一个名为constructor的方法，否则会抛出SyntaxError异常。

在子类的构造器中可以使用super关键字调用父类的构造器。

### 原型方法
```javascript
  class Polygon {
    constructor(height,width){
      this.height = height;
      this.width = width;
    }

    get area(){
      return this.calcArea()
    }

    calcArea(){
      return this.height * this.width;
    }
  }
```

### 静态方法
static关键字用来定义类的静态方法。静态方法是指那些不需要对类进行实例化，使用类名就可以直接访问的方法。静态方法经常用来作为工具函数。
```javascript
  class Point{
    constructor(x,y){
      this.x = x;
      this.y = y;
    }

    static distance(a,b){
      const dx = a*x - b*y;
      const dy = a*y - b*y;

      return Math.sqrt(dx*dx + dy*dy);
    }
  }

  const p1 = new Point(5,5);
  const p2 = new Point(10,10);

  console.log(Point.distance(p1,p2));
```

## 使用extends关键字创建子类
extends 关键字可以用来创建继承于某个类的子类。
```javascript
  class Animal {
    constructor(name){
      this.name = name;
    }

    speak(){
      console.log(this.name + ' make a noise.');
    }
  }

  class Dog extends Animal {
    speak(){
      console.log(this.name + ' barks.');
    }
  }
```

## 为内置对象类型(Array,RegExp等)创建子类
### 使用super关键字引用父类
super这个关键字。有两种用法，含义不同。
 1. 作为函数调用时(即super(...args)),super代表父类的构造函数。
 2. 作为对象调用时(即super.prop或super.method()),super代表父类。注意，此时super即可以引用父类实例的属性和方法，也可以引用父类的静态方法。

```javascript
  class B extends A {
    get m(){
      return this._p * super._p;
    }
    set m(){
      throw new Error('该属性只读');
    }
  }
```
上面代码中，子类通过super关键字，调用父类实例的_p属性。

由于，对象总是继承其他对象的，所以可以在任意一个对象中，使用super关键字。

```javascript
  var obj = {
    toString() {
      return "MyObject: " + super.toString();
    }
  }

  obj.toString(); //MyObject: [Object Object]
```

## Class的取值函数(getter)和存值函数(setter)
与ES5一样，在class内部可以使用get和set关键字，对某个属性设置存值函数和取值函数，拦截该属性的存取行为。

```javascript
  class MyClass {
    constructor(){...}

    get prop(){
      return 'getter';
    }

    set prop(value){
      console.log('setter: ' + value);
    }
  }

  let inst = new MyClass();

  inst.prop = 123;
  //setter: 123

  inst.prop
  //getter
```
上面的代码中，prop属性有对应的存值函数和取值函数，因此赋值和读取行为都被自定义了。

存值函数和取值函数是设置在属性的descriptor对象上的。
```javascript
  class CustomHTMLElement{
    constructor(element){
      this.element = element;
    }

    get html(){
      return this.element.innerHTML;
    }

    set html(value){
      this.element.innerHTML = value;
    }
  }

  var descriptor = Object.getOwnPropertyDescriptor(
    CustomHTMLElement.ptoperty,"html"
  );

  "get" in descriptor //ture
  "set" in descriptor //true
```
上面代码中，存值函数和取值函数是定义在html属性的描述对象上面，这与ES5完全一致。

# Enhanced Object Literals 增强的对象字面量
对象字面量被增强了，写法更加简洁与灵活，同时在定义对象的时候能够做的事情更多了。具体表现在：
 - 可以在对象字面量里面定义原型
 - 定义方法可以不用function关键字
 - 直接调用父类方法

这样一来，对象字面量与前面提到的类概念更加吻合，在编写面向对象的javascript时更加轻松方便。
```javascript
  //通过对象字面量创建对象
  var human = {
    breathe(){
      console.log('breathing...');
    }
  };

  var worker = {
    __proto__: human, //设置此对象的原型为human，相当于继承human
    company: 'freelancer',
    work(){
      console.log('working...');
    }
  };

  human.breathe(); //输出‘breathing...’
  //调用继承来的breathe方法
  worker.breathe(); //输出‘breathing...’
```

# Template Strings 字符串模板
模板字符串允许嵌入表达式，并且支持多行字符串和字符串插补特性。
## 语法
```javascript
  `string text`

  `string text line 1
   string text line 2`

   `string text ${expression} string text`

   tag `string text ${expression} string text`
```

## 描述
模板字符串使用反括号(` `)来代替普通字符串中的用双引号和单引号。模板字符串可以包含特性语法(${expression})的占位符。占位符中的表达式和周围的文本会在一起传递给一个默认函数，该函数负责将所有的部分连接起来，如果一个模板字符串由表达式开头，则该字符串被称为带标签的模板字符串，该表达式通常是一个函数，它会在模板字符串处理后被调用，在输出最终结果前，你都可以通过该函数对模板字符串来进行操作处理。

### 多行字符串
在新行中插入的任何字符都是模板字符串中的一部分，使用普通字符串，你可以通过以下的方式获得多行字符串：
```javascript
  console.log("string text line 1 \n\
  string text line 2");
  //"string text line 1
  // string text line 2"
```

要获得同样效果的多行字符串，只需要使用代码：
```javascript
  console.log(`string text line 1
  string text line 2`);
  // "string text line 1
  // string text line 2"
```

### 表达式插补
在普通字符串中嵌入表达式，必须使用如下语法：
```javascript
  var a = 5;
  var b = 10;
  console.log("Fifteen is " + (a+b) + " and\nnot " + (2*a+b) + ".");
  // "Fifteen is 15 and
  // not 20."
```

现在通过模板字符串，我们可以使用一种更优雅的方式来表示：
```javascript
  var a = 5;
  var b = 10;
  console.log(`Fifteen is &{a+b} and\nnot ${2*a + b}.`);
  // "Fifteen is 15 and
  // not 20."
```

### 带标签的模板字符串
模板字符串的一种更高级的形式称为带标签的模板字符串。它允许你通过标签函数修改模板字符串的输出。标签函数的第一个参数是一个包含了字符串字面值的数组；第二个参数，在第一个参数后的每一个参数，都是已经被处理好的替换表达式。最后，标签函数返回处理好的字符串。在后面的示例中，标签函数的名称可以为任意的合法标志符。
```javascript
  var a = 5;
  var b = 10;

  function tag(string,...values){
    console.log(strings[0]); // "Hello "
    console.log(strings[1]); // " world "
    console.log(values[0]);  // 15
    console.log(values[1]);  // 50

    return "Bazinga!";
  }

  tag`Hello ${ a + b } world ${ a * b}`;
  // "Bazinga!"
```

### 原始字符串
在标签函数的第一个参数中，存在一个特殊的属性raw ，我们可以通过它来访问模板字符串的原始字符串。
```javascript
  function tag(strings, ...values) {
  console.log(strings.raw[0]);
  // "string text line 1 \\n string text line 2"
  }

  tag`string text line 1 \n string text line 2`;
```

另外，使用String.raw() 方法创建原始字符串和使用默认模板函数和字符串连接创建是一样的。
```javascript
  String.raw`Hi\n${2+3}!`;
  // "Hi\\n5!"
```
