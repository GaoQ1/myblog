---
title: 范畴论-JS函数式编程
date: 2016-05-23 09:32:54
tags:
  - javascript
  - 函数式编程
categories: 转载笔记
---
托马斯.沃森(时任IBM董事长)说过一句著名的话，“我想全世界只有五台计算机的市场”。那是1948年，当时，每个人都认为计算机只会被用于两件事情：数学和工程。 即使是技术上最大胆的预想也不会认为有一天计算机能够把西班牙语翻译成英语， 或者模拟整个天气系统。在那时最快的计算机是IBM的SSEC，每秒能计算50次，显示终端要在15年后才出现， 多任务处理意味着多个用户终端共享一个单线程。晶体管改变了一切，然而对技术的远见没有跟上。 在1977年，Ken Olson（DEC创始人）说过另一个愚蠢的预言：“任何人都没有理由想在家里拥有个计算机”。

对于我们来说很明显计算机不只是为科学家和工程师所用，但这只是事后诸葛。在70年前认为计算机只能作数学计算是很正常的。 沃森不仅没有意识到计算机会改变社会，他还没有意识到数学的改革和演进的能力。

不过计算机和数学的潜力并非被所有人忽视。约翰·麦卡锡在1958年发明了Lisp，这是一个革命性的基于算法的语言， 它把计算机带入了新纪元。从那开始，Lisp对使用抽象层的思想（编译、解释、虚拟化）起到重要作用， 这促使了计算机从一个只能用于数学的机器变成了今天这样。

从Lisp到Scheme，一个JavaScript的直接原型。现在它给我们带来了一个轮回。如果计算机在核心上只是一个做数学的机器， 那么它在以数学为基础编程范式上具有优越性是合理的

这里所说的“数学”并不是指计算机明显能做的数字运算，而是要描述为离散数学：对于离散的、 对于诸如逻辑上的声明或者计算机语言命令的数学结构的研究。 通过把代码作为离散的数学结构来对待，我们可以把概念和想法应用到数学上。 这也就是为什么函数式编程在人工智能、图谱搜索、模式识别以及其它计算机科学中具有挑战性的领域里具有如此重要的地位。

这一章我们将针对日常编程中的问题对一些概念和它们的应用进行试验，包括：

 - 范畴论（Category theory）
 - 态射（Morphisms）
 - 函子（Functors）
 - Maybes
 - Promises
 - Lenses
 - 函数组合

利用这些概念我们可以轻松安全地写出整个库和API。并且我们要从对范畴论的解释开始一直到它在JavaScript里的实现。

# 范畴论
> 范畴论是用于函数组合的理论性概念。范畴论和函数组合他们俩在一起就像发动机排量和马力，像NASA和空间穿梭， 像好酒和装它的瓶子。基本上讲，你不能让它们中的一个脱离另一个而独立存在。

## 范畴论概览
范畴论实际并不是一个很难的概念。在数学上它大到能够填满一个本科课程，但是在计算机编程中它可以很容易地被总结出来。

爱因斯坦曾说过：“如果你不能把它解释给一个六岁的孩子听，那你自己也没有理解”。这样，按照给六岁孩子解释的说法， 范畴论只不过是一些连接的圆点。也许这过分简化了范畴论，不过这从直观的方式上很好的解释了我们所需要知道的东西。

首先你需要了解一些术语。范畴(category,也可以说是种类)只是一些同样类型的集合。在javascript里，它们是数组或对象，包含了明确指定为数字、字符串、布尔、日期或节点等类型的变量。太射(Morphism)是一些纯函数，当给定一系列输入时总会返回相同的输出。多态操作可以操作多个范畴，而同态操作限制在一个单独的范畴中。例如，同态函数“乘”只能作用于数字，而多态函数“加”还能作用于字符串。

下图展示了三个范畴--A、B、C，以及两个太射--f和g

![范畴论](/images/范畴论/1.png)
范畴论告诉我们，当第一个态射的范畴是另一个态射所需的输入时，它们就可以像下图所示这样组合：
![范畴论](/images/范畴论/2.png)
fog符号代表态射f和g的组合。现在我们就可以连接这些原点。
![范畴论](/images/范畴论/3.png)
真的是这样，只是连接圆点。

## 类型安全
我们来连接一些圆点。范畴包含两样东西：
1. 对象Object(在javascrip中是类型)
2. 态射Morphisms(在javascrip中是只作用于类型的纯函数)。

这是数学赋予范畴论的术语，所以不幸与我们的javascrip的术语集有些冲突。范畴论中的对象更像是代表一个指定数据类型的变量，而不是像JavaScript所定义的对象那样具有一系列属性和值。 态射只是使用这些类型的纯函数。
所以在javascrip应用范畴论很简单。在javascrip中使用范畴论意味着每个范畴只使用一个特定的数据类型。数据类型是指数字、字符串、数组、日期、对象、布尔等等。但是javascrip没有严格的类型系统，很容易出岔子。所以我们不得不实现我们自己的方法来保证数据的正确。
javascrip中有四种原始类型：number、string、Boolean、function.我们可以创建类型安全函数，返回变量或者抛出一个错误。这符合范畴论的对象定理。
```javascript
  var str = function(s) {
    if (typeof s === "string") {￼￼
      return s;
    } else {
      throw new TypeError("Error: String expected, " + typeof s + "given.");
    }
  }
  var num = function(n) {
    if (typeof n === "number") {
      return n;
    } else {
      throw new TypeError("Error: Number expected, " + typeof n + "given.");
    }
  }
  var bool = function(b) {
    if (typeof b === "boolean") {
      return b;
    } else {
      throw new TypeError("Error: Boolean expected, " + typeof b + "given.");
    }
  }
  var func = function(f) {
    if (typeof f === "function") {
      return f;
    } else {
      throw new TypeError("Error: Function expected, " + typeof f +
        " given.");
    }
  }
```
然而这里重复代码太多，并且不是很函数式。我们可以创建一个函数，它返回一个类型安全的函数。
```javascript
  var typeOf = function(type){
    if(typeof x === type){
      return x;
    }else{
      throw new TypeError("Error: " + type + " exceped, " + typeof x + "given.");
    }
  }
  var str = typeOf('string');
  var num = typeOf('number');
  var func = typeOf('function');
  var bool = typeOf('boolean');
```
现在，我们可以利用这些函数让我们的函数像预期那样运行。
```javascript
  //未受保护的方法
  var x = '24';
  x + 1; //会返回‘241’，而不是25
  //受保护的方法
  //plus :: Int -> Int
  function plus(n){
    return num(n) + 1;
  }
  plus(x); //抛出错误，防止出现意外的结果
```
再来看个有点肉的例子。我们想检查Unix时间戳的长度，由于javascript函数Date.parse()返回的值是数字而不是字符串，我们得用str()函数。
```javascript
  //timestampLength :: String -> Int
  function timestampLength(t){ return num(str(t).length);}
  timestampLength(Date.parse('12/31/1999')); // 抛出错误
  timestampLength(Date.parse('12/31/1999').toString()); // 返回12
```
像这样把明确地一个类型转换为另一个类型（或者是相同的类型）的函数叫做态射。这符合范畴论的态射定理。这里强迫通过类型安全函数进行类型声明，利用了这个机制的态射是我们在javascript中展示范畴概念所需的一切。

## 对象识别
另外还有一个重要的数据类型：对象。
```javascript
  var obj = typeOf('object');
  obj(123); //抛出错误
  obj({x:'a'}); //返回{x:'a'}
```
然而，对象各不相同。它们可以被继承。任何非原始类型(number、string、boolean、functin)的东西都是对象，包括数组、日期、元素等等。
没有办法知道一个对象是个什么类型，也就是说没法通过typeof关键字知道javascript的对象的子类型是什么，所以我们得想办法。Object有个toString()函数，我们可以通过它变通实现这个目的。
```javascript
  var obj = function(o){
    if(Object.prototype.toString.call(o) === "[object object]"){
      return o;
    }else{
      throw new TypeError("Error: Object expected, something else given.");
    }
  }
```
同样，对于各种对象，我们要实现代码重用：
```javascript
  var objectTypeOf = function(name){
    return function(o){
      if(Object.prototype.toString.call(o) === "[object " + name + "]"){
        return o;
      }else{
        throw new TypeError(
          "Error: '+name+' expected, something else given."
        )
      }
    }
  }
  var obj = objectTypeOf('Object');
  var arr = objectTypeOf('Array');
  var date = objectTypeOf('Date');
  var div = objectTypeOf('HTMLDivElement');
```

# 函子(Functiors)
态射是类型之间的映射；函子是范畴之间的映射。可以认为函子是这样一个函数，它从一个容器中取到值，并将其加工，然后放到一个新的容器中。这个函数的第一个输入的参数是类型的态射，第二个输入的参数是容器。
> 函子的函数签名是这样子 //myFunctor :: (a -> b) -> fa -> fb
> 意思是“给我一个传入a返回b的函数和一个包含a(一个或多个)的容器，我会返回一个包含b(一个或多个)的容器”

## 创建函子
要知道我们已经有了一个函子：map()，它攫取包含一些值的容器(数组)，然后把一个函数作用于它。
```javascript
  [1,4,9].map(Math.sqrt); //returns: [1,2,3]
```

然而我们要写成一个全局函数，而不是数组对象的方法。这样我们后面就可以写出简洁、安全的代码。
```javascript
  //map :: (a -> b) -> [a] -> [b]
  var map = function(f,a){
    return arr[a].map(func(f));
  }
```

这个例子看起来像是故意弄的封装，因为我们只是把map()函数换了个形式。但这有它的目的。它为映射其他类型提供了一个模板。
```javascript
  //strmap :: (str -> str) -> str -> str
  var strmap = function(f,s){
    return str(s).split('').map(func(f)).join('');
  }
```

## 数组和函子
数组是函数式javascript使用数据最好的方式。

是否有一种简单的方法来创建已经分配了态射的函子？有，它叫做arrayOf。当你传入一个以整数为参数、返回数组的态射时，你会得到一个以整数数组为参数返回数组的数组的态射。

它自己本身不是函子，但是它让我们能够用态射建立函子。
```javascript
  //arrayOf :: (a -> b) -> ([a] -> [b])
  var arrayOf = function(f){
    return function(a){
      return map(func(f),arr(a));
    }
  }
```
下面是如何用态射创建函子
```javascript
  var plusplusall = arrayOf(plusplus); //plusplus是函子
  console.log(plusplusall([1,2,3])); //返回[2,3,4]
  console.log(plusplusall([1,'2',3])); //抛出错误
```

## 函数组合，重访(revisitd)
函数也是一种我们能够用函子来创建的原始类型，这个函子叫做“fcompose”.我们对函子是这样定义的：它从容器中取一个值，并对其应用一个函数。如果这个容器是一个函数，我们只需要调用它并获取里面的值。

我们已经知道了什么事函数组合，不过让我们来看看在范畴论驱动的环境里它们能做些什么。

函数组合就是结合(associative)，如果你的高中代数老师也像我这样的话那她只告诉了你函数组合的定律有什么，而没有没教你用它能做些什么。 在实践中，组合就是结合律所能够做的。
```
  (a*b)*c = a*(b*c)
  (f o g)o h = f o (g o h)
```

```
  f o g ≠ g o f
```

我们可以任意进行内部组合，无所谓怎样分组。交换律也没有什么可迷惑的。f o g 不总等于 g o f。比如说，一个句子的第一个单词被反转并不等同于一个被反转的句子的第一个单词。

总的来说意思就是哪个函数以什么样的顺序被执行是无所谓的，只要每个函数的输入来源于上一个函数的输出。不过，等等，如果右边的函数依赖于左边的函数，不就是只有一个固定的求值顺序吗？从左到右？是的，如果把它封装起来，我们就可以按照我们感觉合适的方式来控制它。这就使得在JavaScript中可以实现惰性求值。
```
  (a*b)*c = a*(b*c)
  (f o g)o h = f o (g o h)
```

我们来重写函数组合，不作为函数原型的扩展，而是作为一个单独的函数，这样我们就可以的到更多的功能。基本的形式是这样的：

```javascript
  var fcompose = function(f,g){
    return function(){
      return f.call(this,g.apply(this,arguments));
    }
  }
```

不过我们还得让它能接受任意数量的输入。
```javascript
  var fcompose = function(){
    //首先确保所有的参数都是函数
    var funcs = arrayOf(func)(Array.prototype.slice.call(arguments));
    return function(){
      var args = arguments;
      for(var i=func.length-1;i>=0;i--){
        args = [funcs[i].apply(this,args)];
      }
      return args[0];
    }
  }
```

现在我们封装好了这些函数并可以控制它们了。我们重写了组合函数使得每一个函数接受另一个函数作为输入， 存储起来，并同样返回一个对象。这里并不是接受一个数组作为输入处理它，而是对每一个操作返回一个新的数组， 我们可以在源头上让每一个元素接受一个数组，把所有操作合到一起执行（所有map、filter等等组合到一起）， 最终把结果存到一个新数组里。这就是通过函数组合实现的惰性求值。这里我们没有理由重新造轮子， 许多库对于这个概念都有很好的实现，包括Lazy.js、Bacon.js以及wu.js等库。

利用这一不同模式的结果，我们可以做更多事情：异步迭代、异步事件处理、惰性求值甚至自动并行。

# 单子(Monad)
单子是帮助你组合函数的工具。

像原始类型一样，单子是一种数据结构，它可以被当做装载让函子取东西的容器使用。函子取出了数据，进行处理，然后放到一个新的单子中并将其返回。

我么要关注的三种单子：
 - Maybes
 - Promises
 - Lenses

除了用于数组的map和函数的compose以外，我们还有三种函子(maybe、promise和lens).这仅仅是另一些函子和单子。

## Maybe
Maybe可以让我们优雅地使用有可能为空并且有默认值的数据。maybe是一个可以有值也可以没有值的变量，并且对于调用者来说无所谓。
就他自己来说，这看起来不是什么大问题。所有人都知道空值检查可以通过一个if-self语句很容易实现
```javascript
  if(getUsername() == null){
    username = 'Anonymous';
  }else{
    username = getUsername();
  }
```

但是用函数式编程，我们要打破过程、一行接一行的做事方式，而应该用函数和数据的管道方式。如果我们不得不从链的中间断开来检查是否存在，我们就的创建临时变量并写更多的代码。maybe仅仅是帮助我们保持逻辑跟随管道的工具。

要实现maybe，我们首先要创建一些构造器。
```javascript
  //Maybe单子构造器，目前是空的
  var Maybe = function(){}

  //None实例，对一个没有值的对象的包装
  var None = function(){};
  None.prototype = Object.create(Maybe.prototype);
  None.prototype.toString = function(){return 'None'};

  //现在可以写`none`函数
  //这让我们不用总写`new None()`
  var none = function(){return new None()};

  //Just实例，对一个有一个值的对象的包装
  var Just = function(x){return this.x = x};
  Just.prototype = Object.create(Maybe.prototype);
  Just.prototype.toString = function(){return "Just" + this.x};
  var just = function(x){return new Just(x)};
```
-----------------
