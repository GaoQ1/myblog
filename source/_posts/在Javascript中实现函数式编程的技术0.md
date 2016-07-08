---
title: 在Javascript中实现函数式编程的技术
date: 2016-05-19T15:03:40.000Z
tags:
  - javascript
  - 函数式编程
categories: 转载笔记
---

这章我们继续下面的内容：
- 把所有的核心概念放到一个集中的范式里
- 探索函数式编程之美
- 一步步跟踪函数式模式相互交织的逻辑
- 我们将贯穿整章建立一个简单的应用做一些很酷的事情

你可能已经注意到，在上一章我们介绍Javascript的函数式库的时候引入了一些概念， 而不是在第二章《函数式编程基础》里。呃，这是有原因的！组合、柯里化、不全调用...... 让我们来探索这些库为何以及如何实现这些概念。

函数式编程可以出现各种各样的模式，这章会覆盖很多不同风格的函数式编程：
- 数据泛型编程
- 基本上函数式的编程
- 函数式响应式编程等

然而这章将会避免偏向于任何风格。我们不会花费很大精力学完一个函数式编程风格再学另一个， 总的目标是展示写代码更好的方式，而不是被动接受唯一一个正确的选择。 如果你先人一步感觉到了什么是好的写代码的方法而什么不是，你就可以作任何想做的事情了。 当你像个孩子那样仅凭喜好放纵地写代码时，当你不关心如何去循规蹈矩的做事时，无限的可能就出现了。

## 部分函数应用和珂理化
> 许多语言支持可选参数，但是javascript不支持。javascript采用一种完全不同的模式，它允许任意数量的参数传给函数。这就是一些有趣且非同寻常的设计模式留下了门路。函数可以全部或部分应用。

部分应用在javascript中的处理方式是：给函数一个或多个参数绑定上值，然后返回另一个函数接受剩余的未绑定参数。同样，珂理化的处理方式是把一个有多个参数的函数转换为一个只接受一个参数的函数，它返回的函数接受剩余的参数。

这两者的差异现在看起来不是很明显，但最后会清楚的。

## 函数操作
在我们进一步解释如何实现部分应用和珂理化之前，我们需要进行一些回顾。如果我们想要扒掉javascript厚重的C风格语法外衣，暴露器函数是本质的话，我们需要理解原始函数、原型在javascript是如何工作的；而如果我们只是想设置一些cookie或验证一些表单的话则永远不用考虑这些。

## apply、call和this关键词
在纯函数式语言中，函数不会被唤起(invoke)，他们是被应用(apply)。javascript以同样的方式工作，甚至提供了手动调用(call)和应用(apply)函数的工具。这些都是与this关键词有关的，当然this指的是函数所属的那个对象。

call()函数把第一个参数作为this关键字。它是这样工作的：

```javascript
    console.log(['Hello','world'].join(' ')); //正常方式
    console.log(Array.prototype.join.call(['Hello','world'],' ')); //使用call
```

call()函数可以唤起匿名函数：

```javascript
    console.log((function(){console.log(this.length)}).call([1,2,3]));
```

apply()函数和call()函数很像，但是更有用一些：

```javascript
   console.log(Math.max(1,2,3)); // 返回3
   console.log(Math.max([1,2,3])); // 无法应用于数组
   console.log(Math.max.apply(null, [1,2,3])); // 这样就可以了
```

基本的区别是：call()函数接受一列参数，apply函数接受一个数组作为参数。

call()和apply()让你可以只写一次函数，其它对象可以继承它而无需再写一遍函数。 并且他俩都是Function对象的成员。
```javascript
  //当你对call()自己调用call()的时候，会发生有趣的事情。
  //这两行代码是等价的
  func.call(thisValue);
  Function.prototype.call.call(func,thisValue);
```

## 绑定函数
bind()函数让你能够调用一个对象的函数时this指向另一个对象。这跟call()函数差不多，不过他可以让方法链式调用，返回一个新的函数。

这对于回调非常有用，就像下面的代码那样：

```javascript
   function Drum() {
     this.noise = 'boom';
     this.duration = 1000;
     this.goBoom = function() {
       console.log(this.noise)
     };
   }
   var drum = new Drum();
   setInterval(drum.goBoom.bind(drum), drum.duration);
```

这解决了许多面向对象框架中的问题，比如Dojo，特别是对于那些有自己的handler函数的类处理状态维持的问题。 不过我们也可以用bind()来进行函数式编程。

`bind()函数实际上自己实现了部分应用，尽管是通过一种很有限的方式。`

## 函数工厂
> 还记得第二章《函数式编程基础》中关于闭包的那节吗？闭包使建立函数工厂这种Javascript编程模式成为可能。 它们使你能够手动绑定函数的参数。

首先我们需要一个为另一个函数绑定参数的函数：

```javascript
    function bindFirstArg(func,a){
        return function(b){
            return func(a,b);
        }
    }
```

现在我们可以用它来创建更多的泛型函数(generic function)

```javascript
    var powersOfTwo = bindFirstArg(Math.pow, 2);
    console.log(powersOfTwo(3)); // 8
    console.log(powersOfTwo(5)); // 32
```

也可以针对于其它参数：

```javascript
   function bindSecondArg(func, b) {
     return function(a) {
       return func(a, b);
     };
   }
   var squareOf = bindSecondArg(Math.pow, 2);
   var cubeOf = bindSecondArg(Math.pow, 3);
   console.log(squareOf(3)); // 9
   console.log(squareOf(4)); // 16
   console.log(cubeOf(3)); // 27
   console.log(cubeOf(4)); // 64
```

在函数式编程中，创建泛型函数的能力十分重要。然而还有更巧妙的方式可以更加一般化的完成这一过程。bindFirstArg()函数接受两个参数，第一个参数是这个函数。如果我们把bindFirstArg本身作为第一个参数的函数传给它自己，我们就可以创建绑定函数。最好用下面的例子来描述：

```javascript
    var makePowersOf = bindFirstArg(bindFirstArg, Math.pow);
    var powersOfThree = makePowersOf(3);
    console.log(powersOfThree(2)); // 9
    console.log(powersOfThree(3)); // 27
```

这就是为什么它被叫做函数工厂。

## 部分应用
注意我们函数工厂的例子里bindFirstArg()和bindSecondArg()函数只能有两个参数。我们可以写新的不同数量参数的函数，但是这就违背我们一般化的模型了。

我们需要部分应用

部分应用是这样一个过程：它给函数的一个或多个参数绑定上值，返回一个已经部分应用过的函数，这个函数仍然需要接收未绑定的参数。

与bind()函数等Function对象內建的方法不同，我们需要创建自己的函数来实现部分调用和柯里化。主要有两种方式：
- 作为一个单独的函数，也就是，var partial = function(func){...}
- 作为补充，也就是，Function.prototype.partial = function(func){...}

补充的方式视为原型增加新的函数，这会允许我们在为想要部分应用的函数调用我们的新函数的时候作为它的一个方法。就像这样：myfunction.partial(arg1,arg2,...)；

### 左端部分应用
这里Javascript的call()和apply()函数将对我们很有用。我们看看补充Function对象的方式：

```javascript
  Function.prototype.partialApply = function(){
    var func = this;
    args = Array.prototype.slice.call(arguments);
    return function(){
      func.apply(this,args.concat(
        Array.prototype.slice.call(arguments)
      ))
    }
  }
```

如你所见，它的工作方式是对arguments这个特殊的值调用slice

```
  每个函数又有一个特殊的内部变量叫做arguments,它是一个类似于数组的对象，包含传入函数的全部参数。从技术层面说，她不是数组，因此他没有slice和forEach这些数组的方法。这儿就是为什么我们需要使用Array的slice.call方法。
```

现在我们通过一个例子看看如何使用它。这次我们不做数学题，来搞点有用的东西。 我们来建立一个把数字转换为16进制的小应用。

```javascript
  function nums2hex(){
    function componentToHex(component){
      var hex = component.toString(16);
      //确保返回的数值是两位数字，比如0c或12
      if(hex.length == 1){
        return "0" + hex;
      }else{
        return hex;
      }
    }
    return Array.prototype.map.call(arguments,componentToHex).join('');
  }
  //这个函数对多少个数字有效
  console.log(nums2hex()); // ''
console.log(nums2hex(100, 200)); // '64c8'
console.log(nums2hex(100, 200, 255, 0, 123)); // '64c8ff007b'
// 不过我们可以用部分函数来对部分参数进行应用，比如mac地址的OUI
// ( OUI，“组织唯一标识符”，即网卡制造商的唯一标识符。)
var myOUI = 123;
var getMacAddress = nums2hex.partialApply(myOUI);
console.log(getMacAddress()); // '7b'
console.log(getMacAddress(100, 200, 2, 123, 66, 0, 1));
// '7b64c8027b420001'
// 我们还可以转换全红基础上的颜色rgb十六进制值
var shadesOfRed = nums2hex.partialApply(255);
console.log(shadesOfRed(123, 0)); // 'ff7b00'
console.log(shadesOfRed(100, 200)); // 'ff64c8'
```

这个例子展示出了我们可以应用部分参数而生成一个新的函数。它是左-右的，意思是我们只能部分应用从左边开始的若干参数。

### 右部分应用
为了从右边开始应用参数，我们可以再定义一个补充函数。

```javascript
Function.prototype.partialApplyRight = function() {
var func = this;
args = Array.prototype.slice.call(arguments);
return function() {
  return func.apply(
    this, [].slice.call(arguments, 0)
    .concat(args));
};
};
var shadesOfBlue = nums2hex.partialApplyRight(255);
console.log(shadesOfBlue(123, 0));   // '7b00ff'
console.log(shadesOfBlue(100, 200)); // '64c8ff'
var someShadesOfGreen = nums2hex.partialApplyRight(255, 0);
console.log(shadesOfGreen(123));   // '7bff00'
console.log(shadesOfGreen(100));   // '64ff00'
```

部分应用使我们能够创建非常一般化的函数，并从它提取出更多特殊化的函数。 但是这个方法最大的缺点在于参数传入的方式，也就是参数有多少个，是什么样的顺序，这些不太明确。 不明确性在编程中永远不是个好事儿。还有个更好的方式：珂理化。

## 柯里化(currying)
柯里化是这样一个过程：他把一个具有多个参数的函数转换为一个只有一个参数的函数并返回另一个函数，这个被返回的函数需要原函数剩余的参数。这是的说法是：一个具有N个参数的函数可以被转换为具有N个函数的函数链，其中每个函数只有一个参数。

一个普遍的问题是：部分应用和柯里化的区别是什么？实际就是部分应用立刻返回一个值，而柯里化只返回另一个柯里化的函数来获取下一个参数，本质的区别是柯里化可以更好的控制参数传入的方式。

这里我们再为Function的原型补充一个柯里化的方法：

```javascript
  Function.prototype.curry = function(numArgs){
    var func = this;
    numArgs = numArgs || func.length; //func.length是调用此方法的函数的形参个数
    //递归地获取参数
    function subCurry(prev){
      return function(arg){
        if(args.length < numArgs){
          //递归情形：仍需要更多的参数
          return subCurry(args);
        }else{
          //基准情形：执行函数
          return func.apply(this,args);
        }
      }
    }
    return subCurry([]);
  }
```

numArgs参数让我们可以在被珂理化的函数没有给出确切参数的时候指定参数的个数。
