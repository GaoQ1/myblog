---
title: 函数组合-JS函数式编程
date: 2016-05-22T00:46:16.000Z
tags:
  - javascript
  - 函数式编程
categories: 转载笔记
---

> Experience is the mother of wisdom.

# 函数组合
> 在函数式编程中，我们希望一些都是函数，尤其希望是一元函数，如果可能的话。如果可以把多有的函数转换为一元函数，将发生神奇的事情。

 一元函数是只接受单个输入的函数。函数如果有多个输入就是多元的，不过我们一般把接受两个输入的叫二元函数，把接受三个输入的叫三元函数。有的函数接受的输入的数量并不确定，我们称它为可变的

> 操作函数及其可接受数量的输入可以极富表达力。在这一节，我们将探索如何把小的函数组合成新的函数：小的单元逻辑组合成的整个程序比这些函数本身之和还要大。

## 组合
组合函数使我们能够从简单的、通用的函数建立复杂的函数。通过把函数作为其它函数的构建单元，我们可以建立真正模块化的应用，使其具有很棒的可读性和维护性。

在我们定义compose()这个补充函数之前，我们先通过下面的例子看看她是怎么工作的：

```javascript
  var roundedSqrt = Math.round.compose(Math.sqrt);
  console.log(roundedSqrt(5)); //Returns:2
  var squaredDate = roundedSqrt.compose(Date.parse)
  console.log(squaredDate("January 1, 2014")); //Returns: 1178370
```

在函数里，函数f和g的组合定义为f(g(x)).在javascript里，可以写成这样：

```javascript
  var compose = function(f,g){
    return function(x){
      return f(g(x));
    }
  }
```

```
  compose = (f,g) -> (x) -> f g x
```

不过如果就写成这样的话，我们就失去了对this的跟踪。解决方法是使用call()和apply()。与柯里化相比，compose()这个补充函数相当简单：

```javascript
  Function.prototype.compose = function(prevFunc){
    var nextFunc = this;
    return function(){
      return nextFunc.call(this,prevFunc.apply(this,arguments));
    }
  }
```

为了展示他怎么用，来建个完整的例子，如下：

```javascript
  function function1(a){ return a + '1'}
  function function2(b){ return b + '1'}
  function function3(c){ return c + '1'}
  var composition = function3.compose(function2).compose(function1);
  console.log(composition('count'));
```

你是否注意到function1函数最先被应用？这很重要，函数是从右往左应用的。

```
  原文是“Did you notice that the function3 parameter was applied first?”。 意思应该是function3参数最先被应用，这个应该是作者弄错了，显然是funtion1最先被应用， 返回了“count 1”，而且这样顺序也是从右往左的。
```

## 序列--反向组合
由于很多人喜欢从左往右读东西，让函数也从左往右可以更通顺些。我们把这叫做序列而不是组合。为了让顺序相反，我们需要交换nextFunc和prevFunc参数。

```javascript
  Function.prototype.sequence = function(prevFunc){
    var nextFunc = this;
    return function(){
      return prevFunc.call(this,nextFunc.apply(this,arguments));
    }
  }
```

现在可以用更加自然的顺序调用这些函数

```javascript
  var sequences = function1.sequence(function2).sequence(function3);
  console.log(sequences('count')); //returns 'count 1 2 3'
```

## 组合 vs. 链
下面是五种实现floorSqrt()函数组合的方式。它们看起来差不多，但是需要仔细观察。

```javascript
  function floorSqrt1(num){
    var sqrtNum = Math.sqrt(num);
    var floorSqrt = Math.floor(sqrtNum);
    var stringNum = String(floorSqrt);
    return stringNum;
  }
  function floorSqrt2(num){
    return String(Math.floor(Math.sqrt(num)));
  }
  function floorSqrt3(num){
    return [num].map(Math.sqrt).map(Math.floor).toString();
  }
  var floorSqrt4 = String.compose(Math.floor).compose(Math.sqrt);
  var floorSqrt5 = Math.sqrt.sequence(Math.floor).sequence(String);
  //所有的函数都可以这样调用
  floorSqrt <N>(17); //Return:4
```

这里有些关键的区别需要仔细看：
- 第一种方法很明显冗长且低效。
- 第二种方法是个不错的一行代码，但是这种方式只要有几个函数应用就会变得可读性很差。
- 我们说代码越少越好其实没说到点上。代码在有效指令越简洁的时候可维护性越好。如果你减少屏幕上的字符数量却没有改变有效指令的实现，这只能得到相反的效果 -- 代码难以理解，并且真的更难维护了。比如，当我们使用嵌套在一起的三目运算符时，我们就把许多指令放到了一行里面。这种方式减少了屏幕上的代码总量，但是这并没有减少代码实际的具体步骤。所以其效果就是模糊不清难以理解。让代码易于维护的那种简洁是有效减少具体指令（比如使用更简单的算法靠更少和/或更简单的步骤完成同样的结果），或者只是简单地把代码替换为消息，比如调用一个具有良好文档的API的库。
- 第三种方式是一个数组函数的链，尤其是map函数。他工作很好，但并非数学正确的。
- 第四个使我们compose()函数的实际应用。所有的方法被强制为一元的，鼓励使用更好、更简单、更小函数的纯函数只做一件事情，并且做的很好。
- 最后一种实现使用compose()函数相反的顺序，同样有效。

## 使用组合来编程
组合最重要的一个方面是，除了应用的第一个函数以外，他们使用纯函数、只接受一个参数的一元函数效果最好。 执行的第一个函数的输出传递给第二个函数。也就是函数必须接受前一个函数所传给它的东西。类型签名对其有重要作用。

```
  类型签名用于明确地声明函数接受的输入类型是什么以及输出类型是什么。它首先被Haskell使用，实际上Haskell在函数定义时使用它们是为了编译器使用它们。但是，在javascript里，我们只能把个性签名放在代码注释里。它们看起来是这样：foo::arg1 -> argN -> output
  例如：
  // getStringLength :: String -> Int
  function getStringLength(s){return s.length};
  // concatDates :: Date -> Date -> [Date]
  function concatDates(d1,d2){return [d1, d2]};
  // pureFunc :: (int -> Bool) -> [int] -> [int]
  pureFunc(func, arr){return arr.filter(func)}
```

为了能真正尝到组合的甜头，所有应用都需要一个由一元纯函数组成的强大的集合。它们是更大的函数的结构单元，这些大的函数使应用非常模块化、可靠、易维护 来看个例子。首先，我们需要许多结构单元函数。它们中的一些需要依赖于其他函数：

```javascript
  //stringToArray :: String -> [Char]
  function stringToArray(s){
    return s.split('');
  }
  //arrayToString:: [Char] -> String
  function a.join('');
  //nextChar :: Char -> Char
  function nextChar(c){
    return String.fromCharCode(c.charCodeAt(0) + 1);
  }
  //previousChar :: Char -> Char
  function previousChar(c){
    return String.fromCharCode(c.charCodeAt(0) - 1);
  }
  //higherColorHex :: Char -> Char
  function higherColorHex(c){
    return c >= 'f' ? 'f' : c == '9' ? 'a' : nextChar(c)
  }
  // lowerColorHex :: Char -> Char
  function lowerColorHex(c) {
    return c <= '0' ? '0' :
      c == 'a' ? '9' :
      previousChar(c);
  }
  // raiseColorHexes :: String -> String
  function raiseColorHexes(arr) {
    return arr.map(higherColorHex);
  }
  // lowerColorHexes :: String -> String
  function lowerColorHexes(arr) {
    return arr.map(lowerColorHex);
  }
```

现在来把它们组合在一起

```javascript
  var lighterColor = arrayToString
  .compose(raiseColorHexes)
  .compose(stringToArray)
  var darkerColor = arrayToString
  .compose(lowerColorHexes)
  .compose(stringToArray)
  console.log(lighterColor('af0189')); // Returns: 'bf129a'
  console.log(darkerColor('af0189')); // Returns: '9e0078'
```

我们甚至可以混合使用compse()和curry()。实际上，它们一起工作得很好。我们来借助组合的例子来打造珂理化的例子。 首先我们需要一些前面的辅助函数。

```javascript
  //compose2hex :: Ints -> String
  function componentToHex(c){
    var hex = c.toString(16);
    return hex.length == 1 ? '0'+hex : hex;
  }
  //nums2hex :: Ints* -> String
  function nums2hex(){
    return Array.prototype.map.call(arguments,componentToHex).join('');
  }
```

首先我们需要建立柯里化和部分应用的函数，然后把它们组合成其它组合函数。

```javascript
var lighterColors = lighterColor
  .compose(nums2hex.curry());
var darkerRed = darkerColor
  .compose(nums2hex.partialApply(255));
var lighterRgb2hex = lighterColor
  .compose(nums2hex.partialApply());
console.log(lighterColors(123, 0, 22)); // Returns: 8cff11 [原书代码错误，实际返回是8c]
console.log(darkerRed(123, 0)); // Returns: ee6a00
console.log(lighterRgb2hex(123,200,100)); // Returns: 8cd975
```

我们完成了！这些函数易读且直观。我们被迫从只做一件事的小函数开始，然后就能够把函数放在一起形成更多功能。

我们来看最后一个例子。先有个函数根据一个可变的值来减淡RBG值，然后我们用组合根据它创建一个新函数。

```javascript
// lighterColorNumSteps :: string -> num -> string
function lighterColorNumSteps(color, n) {
for (var i = 0; i < n; i++) {
  color = lighterColor(color);
}
return color;
}
// 现在我们可以这样建立函数:
var lighterRedNumSteps =
lighterColorNumSteps.curry().compose(reds)(0, 0);
// 然后这样使用:
console.log(lighterRedNumSteps(5)); // Return: 'ff5555'
console.log(lighterRedNumSteps(2)); // Return: 'ff2222'
```

用同样的方式，我们可以轻松地创建更多的函数来建立更淡或更深的蓝色、绿色、灰色、紫色等等你所想要的。 这是建立API的一个极好的方式。

我们仅仅接触了函数组合能做的事情的一个表面。组合所做的是让控制脱离Javascript。一般Javascript是从左到右求值， 但是现在解释器会说"OK，有人来管它了，我来处理别的东西。"现在compose()函数控制了求值顺序！

这就是Lazy.js和Bacon.js等是如何能够实现惰性求值和无限序列这些东西的。下面我们会看看这些库怎么用。
