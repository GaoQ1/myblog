---
title: JavaScript设计模式
date: 2016-08-17 10:13:37
tags:
- javascript
- 设计模式
categories: 笔记
---
> A particular fine spring came around.

## 构造器模式
在面向对象编程中，构造器是一个当新建对象的内存被分配后，用来初始化该对象的一个特殊函数。在JavaScript中几乎所有的东西都是对象，我们经常会对对象的构造器十分感兴趣。
对象构造器是被用来创建特殊类型的对象的，首先它要准备使用的对象，其次在对象初次被创建时，通过接收参数，构造器要用来对成员的属性和方法进行赋值。

### 对象创建
下面是我们创建对象的三种基本方式：
下面的每一种都会创建一个新的对象：
```javascript
  var newObject = {};

  //or
  var newObject = Object.create(null);

  //or
  var newObject = new Object();
```

最后一个例子中"Object"构造器创建了一个针对特殊值的对象包装，只不过这里没有传值给它，所以它将会返回一个空对象。
有四种方式可以将一个键值对赋给一个对象：
```javascript
  // ECMAScript 3 兼容形式
  //1.“点号”法
  newObject.someKey = "Hello World";

  //获取属性
  var key = newObject.someKey;

  // 2. “方括号”法

  // 设置属性
  newObject["someKey"] = "Hello World";

  // 获取属性
  var key = newObject["someKey"];

  //ECMAScript 5 仅兼容性形式
  //3.Object.defineProperty方式

  //设置属性
  Object.defineProperty(newObject, "someKey", {
    value: "hello world",
    writable: true,
    enumerable: true,
    configurable: true
  });

  //如果上面的方式你感到难以阅读，可以简短的写成下面这样：

  var defineProp = function(obj, key, value){
    config.value = value;
    Object.defineProperty(obj, key, config);
  }

  //为了使用它，我么需要创建一个"person"对象
  var person = Object.create(null);

  //用属性构造对象
  defineProp(person, "car", "Delorean");
  defineProp(person, "dateOfBirth", "1981");
  defineProp(person, "hasBeard", "false");

  //4.Object.defineProperties方式
  Object.defineProperties(newObject, {
    "someKey": {
       value: "Hello World",
       writable: true
     },

     "anotherKey": {
       value: "Foo bar",
       writable: false
     }
  })
  // 3和4中的读取属行可用1和2中的任意一种
```

在这本书的后面，这些方法会被用于继承，如下：
```javascript
  //创建一个继承与person的赛车司机
  var driver = Object.create(person);

  //设置司机的属性
  defineProp(driver, "topSpeed", "100mph");

  //获取继承的属性
  console.log(driver.dateOfBirth);

  //获取我们设置的属性
  console.log(driver.topSpeed);
```

### 基础构造器
正如我们先前所看到的，javascript不支持类的概念，但它有一种与对象一起工作的构造器函数。使用new关键字来调用该函数，我们可以告诉JavaScript把这个函数当做一个构造器来用，它可以用自己所定义的成员来初始化一个对象。

在这个构造器内部，关键字this引用到刚被创建的对象。回到对象创建，一个基本的构造函数看起来像这样：
```javascript
  function Car(model, year, miles){
    this.model = model;
    this.year = year;
    this.miles = miles;

    this.toString = function(){
      return this.model + this.miles;
    };
  }

  //使用，我们可以实例化一个Car
  var civic = new Car("Honda Civic", 2009, 20000);
  var mondeo = new Car("Ford Mondeo", 2010, 5000);

  //output of the toString() method being called on these objects
  console.log(civic.toString());
  console.log(mondeo.toString());
```
上面这是个简单版本的构造器模式，但它还是有些问题。一个是难以继承，另一个是每个Car构造函数创建的对象中，toString()之类的函数都被重新定义。这不是非常好，理想的情况是所有Car类型的对象都应该应用同一个函数。ECMAScript3和ECMAScript5-兼容版，对于构造对象他们提供了另外一些选择，解决限制小菜一碟。

### 使用“原型”的构造器
在JavaScript中函数有一个prototype的属性。当我们调用JavaScript的构造器创建一个对象时，构造函数prototype上的属性对于所创建的对象来说都可见。照这样，就可以创建多个访问相同prototype的Car对象了。下面，我们来扩展一下原来的例子：
```javascript
  function Car(model, year, miles){
    this.model = model;
    this.year = year;
    this.miles = miles;
  }

  Car.prototype.toString = function () {
    return this.model + " has done " + this.miles + " miles";
  };

  var civic = new Car( "Honda Civic", 2009, 20000 );
  var mondeo = new Car( "Ford Mondeo", 2010, 5000 );

  console.log( civic.toString() );
  console.log( mondeo.toString() );
```
通过上面的代码，单个toString()实例被所有的Car对象所共享。

## 模块化模式
**模块**
模块是任何健壮的应用程序体系结构不可或缺的一部分，特点是有助于保持应用项目的代码单元技能清晰地分离又有组织。
在JavaScript中，实现模块有几个选项，他们包括：
 - 模块化模式
 - 对象表示法
 - AMD模块
 - CommonJS模块
 - ECMAScript Harmony模块
模块化模式是基于对象的文字部分。

### 对象字面量
在对象字面值的标记里，一个对象被描述为一组以逗号分隔的名称/值对括在大括号({})的集合。对象内部的名称可以是字符串或是标记符后跟着一个冒号":"。在对象里最后一个名称/值对不应该以","为结束符，因为这样会导致错误。
```javascript
  var myObjectLiteral = {
    variableKey: variableValue,
    functionKey: function(){
      //...
    }
  }
```
对象字面量不要求使用新的操作实例，但是不能够在结构体开始使用，因为打开"{"可能被解释为一个块的开始。在对象外新的成员会被加载，使用分配如下：smyModule.prototype = "someValue";下面我们可以看到一个更完整的使用对象字面值定义一个模块的例子：
```javascript
  var myModule = {
    myProperty: "someValue",
    myConfig: {
      useCaching: true,
      language: "en"
    },
    myMethod: function () {
      console.log( "Where in the world is Paul Irish today?" );
    },
    myMethod2: function () {
      console.log( "Caching is:" + ( this.myConfig.useCaching ) ? "enabled" : "disabled" );
    },
    myMethod3: function( newConfig ) {
      if ( typeof newConfig === "object" ) {
        this.myConfig = newConfig;
        console.log( this.myConfig.language );
      }
    }
  };

  // 输出: Where in the world is Paul Irish today?
  myModule.myMethod();

  // 输出: enabled
  myModule.myMethod2();

  // 输出: fr
  myModule.myMethod3({
    language: "fr",
    useCaching: false
  });
```
使用对象字面值可以协助封装和组织你的代码。也就是说，如果我们选择了这种技术，我们可能对模块模式有同样的兴趣。即使使用对象字面值，但也只有一个函数的返回值。

### 模块化模式
模块化模式最初被定义为一种对传统软件工程的类提供私有和公共封装的方法。
在JavaScript中，模块化模式用来进一步模拟类的概念，通过这样一种方式：我们可以在一个单一的对象中包含公共/私有的方法和变量，从而从全局范围中屏蔽特定的部分。这个结果是可以减少我们的函数名称与在页面中其他脚本区域定义的函数名称冲突的可能性。

### 私有信息
模块模式使用闭包的方式来将“私有信息”，状态和组织结构封装起来。提供了一种将公有和私有方法，变量封装混合在一起的方式，这种方式防止内部信息泄露到全局中，从而避免了和其他开发者发生冲突的可能性。在这种模式下只有公有的API会返回，其它将全部保留在闭包的私有空间中。

这种方法提供了一个比较清晰的解决方案，在只暴露一个接口供其它部分使用的情况下，将执行繁重任务的逻辑保护起来。这个模式非常类似于立即调用函数表达式，但是这种模式返回的是对象，而立即调用函数表达式返回的是一个函数。

需要注意的是，在javaScript事实上没有一个显式的真正意义上的"私有化"概念，因为与传统语言不同，javaScript没有访问修饰符。从技术上将，变量不能被声明为公有的或私有的，因此我们使用函数域的方式去模拟这个概念。在模块模式中，因为闭包的缘故，声明的变量或者方法只在模块内部有效。在返回对象中定义的变量或者方法可以供任何人使用。

**例子**
下面这个例子通过创建一个自包含的模块实现了模块模式。
```javascript
  var testModule = (function(){
    var counter = 0;
    return {
      incrementCounter: function(){
        return counter++;
      },

      resetCounter: function(){
        console.log("counter: ",counter);
        counter = 0;
      }
    };
  })();

  //Increment our counter
  testModule.incrementCounter();

  //Check the counter value and reset
  //Outputs: 1
  testModule.resetCounter();
```
在这里我们看到，其它部分的代码不能访问我们的incrementCounter()或者resetCounter()的值。counter变量被完全从全局域中隔离起来了，因此其表现的就像一个私有变量一样，它的存在只局限于模块的闭包内部，因此只有两个函数可以访问counter。我们的方法是有名字空间限制的，因此在我们代码的测试部分，我们需要给所有函数调用前面加上模块的名字(例如:"testModule")。

当使用模块模式时，我们会发现通过使用简单的模板，对于开始使用模式非常有用。下面是一个模板包好了命名空间，公共变量和私有变量。
```javascript
  var myNamespace = (function(){
    var myPrivateVar, myPrivateMethod;

    //A private counter variable
    myPrivateVar = 0;

    //A private function which logs any arguments
    myPrivateMethod = function(foo){
      console.log(foo);
    };

    return {
      //A public valiable
      myPublicVar: "foo",

      //A public function utilizing privates
      myPublicFunction: function(bar){
        // Increment our private counter
        myPrivateVar++;

        // Call our private method using bar
        myPrivateMethod( bar );
      }
    };
  })();
```

看一下另外一个例子，下面我们看到一个使用这种模式实现的购物车。这个模块完全自包含在一个叫做basketModule全局变量中。模块中的购物车数组是私有的，应用的其他部分不能直接读取。只存在与模块的闭包中，因此只有可以访问其域的方法可以访问这个变量。
```javascript
  var basketModule = (function(){
    // privates
    var basket = [];

    function doSomethingPrivate() {
      //...
    }

    function doSomethingElsePrivate() {
      //...
    }

    // Return an object exposed to the public
    return {
      // Add items to our basket
      addItem: function( values ) {
        basket.push(values);
      },

      // Get the count of items in the basket
      getItemCount: function () {
        return basket.length;
      },

      // Public alias to a  private function
      doSomething: doSomethingPrivate,

      // Get the total value of items in the basket
      getTotal: function () {
        var q = this.getItemCount(),
            p = 0;

        while (q--) {
          p += basket[q].price;
        }

        return p;
      }
    };
  })();
```

在模块内部，你可能注意到我们返回了另外一个对象。这个自动赋值给了basketModule因此我们可以这样和这个对象交互。
```javascript
  // basketModule returns an object with a public API we can use

  basketModule.addItem({
    item: "bread",
    price: 0.5
    });

  basketModule.addItem({
    item: "butter",
    price: 0.3
    });

  // Outputs: 2
  console.log( basketModule.getItemCount() );

  // Outputs: 0.8
  console.log( basketModule.getTotal() );

  // However, the following will not work:

  // Outputs: undefined
  // This is because the basket itself is not exposed as a part of our
  // the public API
  console.log( basketModule.basket );

  // This also won't work as it only exists within the scope of our
  // basketModule closure, but not the returned public object
  console.log( basket );
```

上面的方法都处于basketModule的名字空间中。
请注意在上面的basket模块中，域函数是如何在我们所有的函数中被封装起来的，以及我们如何立即调用这个域函数，并且将返回值保存下来。这种方式有以下的优势：
 - 可以创建只能被我们模块访问的私有函数。这些函数没有暴露出来(只有一些API是暴露出来的)，它们被认为是完全私有的。
 - 当我们在一个调试器中，需要发现哪个函数抛出异常的时候，可以很容易的看到调用栈，因为这些函数是正常声明的并且是命名的函数。
 - 这种模式同样可以让我们在不同的情况下返回不同的函数。我见过有开发者使用这种技巧用于执行UA测试，目的是为了在它们的模块里面针对IE专门提供一条代码路径，但是现在我们也可以简单的使用特征检测达到相同的目的。

### Import mixins(导入混合)
这个变体展示了如何将全局(例如jQuery, Underscore)作为一个参数传入模块的匿名函数。这种方式允许我们导入全局，并且按照我们的想法在本地为这些全局起一个别名。
```javascript
  //Global module
  var myModule = (function(jQ, _ ){
    function privateMethod1(){
      jQ(".container").html("test");
    }

    function privateMethod2(){
      console.log(_.min([10,5,100,2,1000]));
    }

    return {
      publicMethod: function(){
        privateMethod1();
      }
    }
  })(jQuery, _);

  myModule.publicMethod();
```

### Exports(导出)
这个变体允许我们声明全局对象而不是使用它们，同样也支持在下一个例子中我们将会看到的全局导入的概念。
```javascript
  //Global module
  var myModule = (function(){
    //Module object
    var module = {}, privateVariable = "Hello World";

    function privateMethod(){
      //...
    }

    module.publicProperty = "Foobar";
    module.publicMethod = function(){
      console.log(privateVariable);
    };

    return module;
  })()
```
工具箱和框架特定的模块模式实现。

**jQuery**
因为jQuery编码规范没有规定插件如何实现模块模式，因此有很多方式可以实现模块模式。在下面的例子中，定义了一个library函数，这个函数声明了一个新的库，并且在新的库(例如模块)创建的时候，自动将初始化函数绑定到document的ready上。
```javascript
  function library(module){
    $(function(){
      if(module.init){
        module.init();
      }
    });
    return module;
  }

  var myLibrary = library(function(){
    return {
      init: function(){
        //module implementation
      }
    }
  })();
```
**优势**
既然我们已经看到单例模式很有用，为什么还是使用模块模式呢？首先，对于有面向对象背景的开发者来讲，至少从javascript语言上来讲，模块模式相对于真正的封装概念更清晰。

其次，模块模式支持私有数据，因此在模块模式中，公共部分代码可以访问私有化数据，但是在模块外部，不能访问类的私有部分。

**缺点**
模块模式的缺点是因为我们采用不同的方式访问公有和私有成员，因此当我们想要改变这些成员的可见性的时候，我们不得不在所有使用这些成员的地方修改代码。

我们也不能在对象之后添加的方法里面访问这些私有变量。也就是说，很多情况下，模块模式很有用，并且当使用正确的时候，潜在地可以改善我们代码的结构。

其它缺点包括不能为私有成员创建自动化的单元测试，以及在紧急修复bug时所带来的额外的复杂性。根本没有可能可以对私有成员打补丁。相反地，我们必须覆盖所有的使用存在bug私有成员的公共方法。开发者不能简单的扩展私有成员，因此我们需要记得，私有成员并非它们表面上看上去那么具有扩展性。

## 暴露模块模式
既然我们对模块模式已经有一些了解，让我们看一下改进版本 - Christian Heilmann的启发模块模式。启发模块模式来自于，当Heilmann对这样一个现状的不满，即当我们想要在一个公有方法中调用链另一个公有方法，或者访问公有变量的时候，我们不得不重复主对象的名称。他也不喜欢模块模式中，当想要将某个成员变成公共成员时，修改文字标记的做法。

因此他工作的结果就是一个更新的模式，在这个模式中，我们可以简单地在私有域中定义我们所有的函数和变量，并且返回一个匿名对象，这个对象包含有一些指针，这些指针指向我们想要暴露出来的私有成员，使这些私有成员公有化。

下面给出了一个如何使用暴露式模块模式的例子：
```javascript
  var myRevealingModule = function(){
    var privateVar = "Ben Cherry",
        publicVar = "Hey there!";

    function privateFunction(){
      console.log("Name: " + privateVar);
    }

    function publicSetName( strName ) {
        privateVar = strName;
    }

    function publicGetName() {
        privateFunction();
    }

    // Reveal public pointers to
    // private functions and properties

    return {
        setName: publicSetName,
        greeting: publicVar,
        getName: publicGetName
    };
  }();

  myRevealingModule.setName("Paul Kinlan");
```

这个模式可以用于将私有函数和属性以更加规范的命名方式展现出来。
```javascript
var myRevealingModule = function () {

    var privateCounter = 0;

    function privateFunction() {
        privateCounter++;
    }

    function publicFunction() {
        publicIncrement();
    }

    function publicIncrement() {
        privateFunction();
    }

    function publicGetCount(){
      return privateCounter;
    }

    // Reveal public pointers to
    // private functions and properties       

   return {
        start: publicFunction,
        increment: publicIncrement,
        count: publicGetCount
    };

}();

myRevealingModule.start();
```
 **优势**
这个模式是我们脚本的语法更加一致。同样在模块的最后关于那些函数和变量可以被公共访问也变得更加清晰，增强了可读性。

 **缺点**
这个模式的一个缺点是如果私有函数需要使用公有函数，那么这个公有函数在需要打补丁的时候就不能被重载。因为私有函数仍然使用的是私有的实现，并且这个迷失不能用于公有成员，只用于函数。
公有成员使用私有成员也遵循上面不能打补丁的规则。
因为上面的原因，使用暴露式模块模式创建的模块相对于原始的模块模式更容易出问题，因此在使用的时候需要小心。

## 单例模块
单例模式之所以这么叫，是因为它限制一个类只能有一个实例化对象。经典的实现方式是，创建一个类，这个类包含一个方法，这个方法在没有对象存在的情况下，将会创建一个新的实例对象。如果对象存在，这个方法只是返回这个对象的引用。

单例和静态类不同，因为我们可以退出单例的初始化时间。通常这样做是因为，在初始化的时候需要一些额外的信息，而这些信息在声明的时候无法得知。对于并不知晓对单例模式引用的代码来讲，单例模式没有为它们提供一种方式可以简单的获取单例模式。这是因为，单例模式既不返回对象也不返回类，它只返回一种结构。可以类比闭包中的变量不是闭包，提供闭包的函数域是闭包。

在JavaScript语言中，单例服务作为一个从全局空间的代码实现中隔离出来共享的资源空间是为了提供一个单独的函数访问指针。

我们能像这样实现一个单例：
```javascript
  var mySingleton = (function(){
    //Instance stores a reference to the Singleton
    var instance;

    function init(){
      //单例
      //私有方法和变量
      function privateMethod(){
        console.log("I am private");
      }

      var privateVariable = "I am also private";

      var privateRandomNumber = Math.random();

      return {
        // 共有方法和变量
        publicMethod: function () {
          console.log( "The public can see me!" );
        },

        publicProperty: "I am also public",

        getRandomNumber: function() {
          return privateRandomNumber;
        }
      };
    };

    return {
      //如果存在获取此单例实例，如果不存在创建一个单例实例
      getInstance: function(){
        if(!instance){
          instance = init();
        }
        return instance;
      }
    };
  })();

  var myBadSingleton = (function(){
    //存储单例实例的引用
    var instance;

    function init(){
      //单例
      var privateRandomNumber = Math.random();

      return {
        getRandomNumber: function(){
          return privateRandomNumber;
        }
      };
    };

    return {
      //总是创建一个新的实例
      getInstance: function(){
        instance = init();
        return instance;
      }
    }
  })();

  //使用
  var singleA = mySingleton.getInstance();
  var singleB = mySingleton.getInstance();
  console.log( singleA.getRandomNumber() === singleB.getRandomNumber() ); // true

  var badSingleA = myBadSingleton.getInstance();
  var badSingleB = myBadSingleton.getInstance();
  console.log( badSingleA.getRandomNumber() !== badSingleB.getRandomNumber() ); // true
```

创建一个全局访问的单例实例(通常通过MySingleton.getInstance())因为我们不能(至少在静态语言中)直接调用new MySingleton()创建实例。这在javascript语言中是不可能的。
单例模式的描述如下：
 - 每个类只有一个实例，这个实例必须通过一个广为人知的接口，来被客户访问。
 - 子类如果要扩展这个唯一的实例，客户可以不用修改代码就能使用这个扩展后的实例。
关于第二点，可以参考如下的实例，我们需要这样编码：
```javascript
  mySingleton.getInstance = function(){
    if(this._instance == null){
      this._instance = new FooSingleton();
    }else{
      this._instance = new BasicSingleton();
    }
    return this._instance;
  }
```
在这里，getInstance 有点类似于工厂方法，我们不需要去更新每个访问单例的代码。FooSingleton可以是BasicSinglton的子类，并且实现了相同的接口。
为什么对于单例模式来讲，延迟执行这么重要？

**在C++代码中，单例模式将不可预知的动态初始化顺序问题隔离掉，将控制权返回给程序员。**
区分类的静态实例和单例模式很重要：尽管单例模式可以被实现成一个静态实例，但是单例可以懒构造，在真正用到之前，单例模式不需要分配资源或者内存。
如果我们有个静态对象可以被直接初始化，我们需要保证代码总是以同样的顺序执行当你有很多源文件的时候，这种方式没有可扩展性。
单例模式和静态对象都很有用，但是不能滥用-同样的我们也不能滥用其他模式。
在实践中，当一个对象需要和另外的对象进行跨系统协作的时候，单例模式很有用。下面是一个单例模式在这种情况下使用的例子：
```javascript
  var SingletonTester = (function(){
    // options: an object containing configuration options for the singleton
    function Singleton(options){
      options = options || {};
      this.name = "SingletonTesterer";
      this.pointX = options.pointX || 6;
      this.pointY = options.pointX || 10;
    };

    var instance;
    var _static = {
      name: "SingletonTesterer",
      getInstance: function(options){
        if(instance === undefined){
          instance = new Singleton(options);
        }
        return instance;
      }
    };

    return _static;
  })();

  var singletonTest = SingletonTesterer.getInstance({
    pointX: 5
  });

  console.log(singletonTest.pointX);
```

尽管单例模式有着合理的使用需求，但是通常当我们发现自己需要在javascript使用它的时候，这是一种信号，表明我们可能需要去重新评估自己的设计。
这通常表明系统中的模块要么紧耦合要么逻辑过于分散在代码库的多个部分。单例模式更难测试，因为可能有多种多样的问题出现，例如隐藏的依赖关系，很难去创建多个实例，很难理清依赖关系，等等。


## 观察者模式
观察者模式是这样一种设计模式。一个被称作被观察者的对象，维护一组被称为观察者的对象，这些对象依赖于被观察者，被观察者自动将自身的状态的任何变化通知给它们。

当一个被观察者需要将一些变化通知给观察者的时候，它将采用广播的方式，这条广播可能包含特定于这条通知的一些数据。

当特定的观察者不再需要接受来自于它所注册的被观察者通知的时候，被观察者可以将其从所维护的组中删除。在这里提及一下设计模式现有的定义很有必要。这个定义是与所使用的语言无关的。通过这个定义，最终我们可以更深层次地了解设计模式如何使用以及其优势。在四人帮的《设计模式：可重用的面向对象软件的元素》这本书中，是这样定义观察者模式的：

一个或者更多的观察者对一个被观察者的状态感兴趣，将自身的这种感兴趣通过附着自身的方式注册在被观察者身上。防备观察者发生变化，而这种便可也是观察者所关心的，就会产生一个通知，这个通知将会被送出去，最后将会调用每个观察者的更新方法。当观察者不在对被观察者的状态感兴趣的时候，它们只需要简单的将自身剥离即可。

我们现在可以通过实现一个观察者模式来进一步扩展我们刚才所学的东西。这个实现包含以下组件：
 - 被观察者：维护一组观察者，提供用于增加和移除观察者的方法。
 - 观察者：提供一个更新接口，用于当被观察者状态变化时，得到通知。
 - 具体的被观察者：状态变化时广播通知给观察者，保持具体的观察者的信息。
 - 具体的观察者：保持一个纸箱具体被观察者的引用，实现一个更新接口，用于观察，以便保证自身状态总是和被观察者状态一致的。

首先，让我们对被观察者可能有的一组依赖其的观察者进行建模：
```javascript
  function ObserverList(){
    this.observerList = [];
  }

  ObserverList.prototype.Add = function(obj){
    return this.observerList.push(obj);
  }

  ObserverList.prototype.Empty = function(){
    this.observerList = [];
  }

  ObserverList.prototype.Count = function(){
    return this.observerList.length;
  }

  ObserverList.prototype.Get = function(index){
    if(index > -1 && index < this.observerList.length){
      return this.observerList[index]
    }
  }

  ObserverList.prototype.Insert = function(obj,index){
    var pointer = -1;

    if(index === 0){
      this.observerList.unshift(obj);
      pointer = index;
    }else if(index === this.observerList.length){
      this.observerList.push(obj);
      pointer = index;
    }

    return pointer;
  }

  ObserverList.prototype.IndexOf = function(obj,startIndex){
    var i = startIndex, pointer = -1;
    while(i< this.observerList.length){
      if(this.observerList[i] === obj){
        pointer = i;
      }
      i++;
    }

    return pointer;
  };

  ObserverList.prototype.RemoveAt = function(index){
    if(index === 0){
      this.observerList.shift();
    }else if(index === this.observerList.length -1){
      this.observerList.pop();
    }
  };

  //Extend an object with an extension
  function extend(extension, obj){
    for(var key in extension){
      obj[key] = extension[key];
    }
  }
```

接着，我们对被观察者以及其增加、删除，通知在观察者列表中的观察者的能力进行建模：
```javaScript
  function Subject(){
    this.observers = new ObserverList();
  }

  Subject.prototype.AddObserver = function(observer){
    this.observers.Add(observer);
  }

  Subject.prototype.RemoveObserver = function(observer){
    this.observers.RemoveAt(this.observers.IndexOf(observer,0));
  };

  Subject.prototype.Notify = function(context){
    var observerCount = this.observers.Count();
    for(var i=0;i< observerCount; i++){
      this.observers.Get(i).Update(context);
    }
  }
```

我们接着定义建立新的观察者的一个框架。这里的update函数之后会被具体的行为覆盖。
```javascript
  //The Observer
  function Observer(){
    this.update = function(){
      //...
    }
  }
```

在我们的样例应用里面，我们使用上面的观察者组件，现在我们定义：
 - 一个按钮，这个按钮用于增加新的充当观察者的选择框到页面上
 - 一个控制用的额选择框，充当一个被观察者，通知其它选择框是否用该被选中
 - 一个容器，用于放置新的选择框

我们接着定义具体被观察者和具体观察者，用于给页面增加新的观察者，以及实现更新接口。通过查看下面的内联的注释，搞清楚在我们的样例中的这些组件是如何工作的。

**HTML**
```javascript
  <button id="addNewObserver">Add New Observer checkbox</button>
  <input id="mainCheckbox" type="checkbox"/>
  <div id="observersContainer"></div>
```

**Sample script**
```javascript
// 我们DOM 元素的引用

var controlCheckbox = document.getElementById( "mainCheckbox" ),
addBtn = document.getElementById( "addNewObserver" ),
container = document.getElementById( "observersContainer" );

// 具体的被观察者

//Subject 类扩展controlCheckbox 类
extend( new Subject(), controlCheckbox );

//点击checkbox 将会触发对观察者的通知
controlCheckbox["onclick"] = new Function( "controlCheckbox.Notify(controlCheckbox.checked)" );

addBtn["onclick"] = AddNewObserver;

// 具体的观察者

function AddNewObserver(){

  //建立一个新的用于增加的checkbox
  var check  = document.createElement( "input" );
  check.type = "checkbox";

  // 使用Observer 类扩展checkbox
  extend( new Observer(), check );

  // 使用定制的Update函数重载
  check.Update = function( value ){
    this.checked = value;
  };

  // 增加新的观察者到我们主要的被观察者的观察者列表中
  controlCheckbox.AddObserver( check );

  // 将元素添加到容器的最后
  container.appendChild( check );
}
```
在这个例子里面，我们看到了如何实现和配置观察者模式，了解了被观察者，观察者，具体被观察者，具体观察者的概念。

**观察者模式和发布/订阅模式的不同**
观察者模式确实有用，但是在javaScript世界里面，通常我们使用一种叫做发布/订阅模式的变体来实现观察者模式。这两种模式很相似，但是也有一些值得注意的不同。

观察者模式要求想要接收相关通知的观察者必须到发起这个事件的被观察者上注册这个事件。

发布/订阅模式使用一个主题/事件频道，这个频道处于想要获取通知的订阅者和发起事件的发布者之间。这个事件系统允许代码定义应用相关的事件，这个事件可以传递特殊的参数，参数中包含有订阅者所需要的值。这种想法是为了避免订阅者和发布者之间的依赖性。

这种和观察者模式之间的不同，使订阅者可以实现一个合适的事件处理函数，用于注册和接受由发布者广播的相关通知。

这里给出一个关于如何使用发布者/订阅者模式的例子，这个例子中完整地实现了功能强大的publish(), subscribe() 和 unsubscribe()。
```javascript
  //一个非常简单的邮件处理器
  //接受的消息的计数器
  var mailCounter = 0;

  //初始化一个订阅者，这个订阅者监听名叫"indox/newMessage"的频道

  //渲染新消息的粗略信息
  var subscribe1 = subscribe("inbox/newMessage", function(topic,data){
    //日志记录主题，用于调试
    console.log("A new message was received: ", topic);

    //使用来自于被观察者的数据，用于给用户展示一个消息的粗略信息
    $(".messageSender").html(data.sender);
    $(".messagePriview").html(data.body);
  });

  //这是另外一个订阅者，使用相同的数据执行不同的任务
  //更新计数器，显示当前来自于发布者的新信息的数量
  var subscribe2 = subscribe("inbox/newMessage", function(topic,data){
    $(".newMessageCounter").html(mailCounter++);
  });

  publish("inbox/newMessage",[{
    sender: "hello@ceair.com",
    body: "Hello world!"
  }]);

  //在之后，我们可以让我们的订阅者通过下面的方式取消订阅来自于新主题的通知
  //unsubscribe(subscribe1);
  //unsubscribe(subscribe2);
```
这个例子的更广的意义是对松耦合的原则的一种推崇。不是一个对象直接调用另外一个对象的方法，而是通过订阅另外一个对象的一个特定的任务或者活动，从而在这个任务或者活动出现的时候得到通知。

**优势**
观察者和发布/订阅模式鼓励人们认真考虑应用不同部分之间的关系，同时帮助我们找出这样的层，该层中包含有直接的关系，这些关系可以通过一系列的观察者和被观察者来替换掉。这种方式可以有效地将一个应用程序切割成小块，这些小块耦合度低，从而改善代码的管理，以及用于潜在的代码复用。

使用观察者模式更深层次的动机是，当我们需要维护相关对象的一致性的时候，我们可以避免对象之间的紧密耦合。例如，一个对象可以通知另外一个对象，而不需要知道这个对象的信息。

两种模式下，观察者和被观察者之间都可以存在动态关系。这提供很好的灵活性，而当我们的应用中不同的部分之间紧密耦合的时候，是很难实现这种灵活性的。

尽管这些模式并不是万能的灵丹妙药，这些模式仍然是作为最好的设计松耦合系统的工具之一，因此在任何的JavaScript开发者的工具箱里面，都应该有这样一个重要的工具。

**缺点**
事实上，这些模式的一些问题实际上正是来自于它们所带来的一些好处。在发布/订阅模式中，将发布者共订阅者上解耦，将会在一些情况下，导致很难确保我们应用中的特定部分按照我们预期的那样正常工作。

例如，发布者可以假设有一个或者多个订阅者正在监听他们。比如我们基于这样的假设，在某些应用处理过程中来记录或者输出错误日志。如果订阅者执行日志功能崩溃了，因为系统本身的解耦本质，发布者没有办法感知到这些事情。

另外一个这种模式的缺点是，订阅者对彼此之间存在没有感知，对切换发布者的代价无从得知。因为订阅者和发布者之间的动态关系，更新依赖也很难去追踪。

**发布/订阅实现**
发布/订阅在JavaScript的生态系统中非常合适，主要是因为作为核心的ECMAScript实现是事件驱动的。尤其是在浏览器环境下更是如此，因为DOM使用事件作为其主要的用于脚本的交互API。

也就是说，无论是ECMAScript还是DOM都没有在实现代码中提供核心对象或者方法用于创建定制的事件系统。

幸运的是，流行的JavaScript库例如dojo, jQuery(定制事件)以及YUI已经有相关的工具，可以帮助我们方便的实现一个发布/订阅者系统。下面我们看一些例子。
```javascript
  // 发布

  // jQuery: $(obj).trigger("channel", [arg1, arg2, arg3]);
  $( el ).trigger( "/login", [{username:"test", userData:"test"}] );

  // Dojo: dojo.publish("channel", [arg1, arg2, arg3] );
  dojo.publish( "/login", [{username:"test", userData:"test"}] );

  // YUI: el.publish("channel", [arg1, arg2, arg3]);
  el.publish( "/login", {username:"test", userData:"test"} );

  // 订阅

  // jQuery: $(obj).on( "channel", [data], fn );
  $( el ).on( "/login", function( event ){...} );

  // Dojo: dojo.subscribe( "channel", fn);
  var handle = dojo.subscribe( "/login", function(data){..} );

  // YUI: el.on("channel", handler);
  el.on( "/login", function( data ){...} );

  // 取消订阅

  // jQuery: $(obj).off( "channel" );
  $( el ).off( "/login" );

  // Dojo: dojo.unsubscribe( handle );
  dojo.unsubscribe( handle );

  // YUI: el.detach("channel");
  el.detach( "/login" );
```

尤其对于jQuery 开发者来讲，他们拥有很多其它的选择，可以选择大量的良好实现的代码，从Peter Higgins 的jQuery插件到Ben Alman 在GitHub 上的（优化的）发布/订阅 jQuery gist。下面给出了这些代码的链接。
 - [Ben Alman的发布/订阅 gist(推荐)](https://gist.github.com/661855)
 - [Rick Waldron 在上面基础上修改的 jQuery-core 风格的实现](https://gist.github.com/705311)
 - [Peter Higgins 的插件](http://github.com/phiggins42/bloody-jquery-plugins/blob/master/pubsub.js)
 - [AppendTo 在AmplifyJS中的 发布/订阅实现](http://amplifyjs.com/)
 - [Ben Truyman的 gist](https://gist.github.com/826794)
