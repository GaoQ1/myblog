---
title: 在Javascript中实现函数式编程的技术
date: 2016-05-19 15:03:40
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
 
你可能已经注意到，在上一章我们介绍Javascript的函数式库的时候引入了一些概念， 而不是在第二章《函数式编程基础》里。呃，这是有原因的！组合、柯里化、不全调用…… 让我们来探索这些库为何以及如何实现这些概念。

函数式编程可以出现各种各样的模式，这章会覆盖很多不同风格的函数式编程：
 - 数据泛型编程
 - 基本上函数式的编程
 - 函数式响应式编程等
 
然而这章将会避免偏向于任何风格。我们不会花费很大精力学完一个函数式编程风格再学另一个， 总的目标是展示写代码更好的方式，而不是被动接受唯一一个正确的选择。 如果你先人一步感觉到了什么是好的写代码的方法而什么不是，你就可以作任何想做的事情了。 当你像个孩子那样仅凭喜好放纵地写代码时，当你不关心如何去循规蹈矩的做事时，无限的可能就出现了。
 
## 部分函数应用和珂理化
许多函数支持可选参数，但是javascript不支持。javascript采用一种完全不同的模式，它允许任意数量的参数传给函数。这就是一些有趣且非同寻常的设计模式留下了门路。函数可以全部或部分应用。

部分应用在javascript中的处理方式是：给函数一个或多个参数绑定上肢，然后返回另一个函数接受剩余的未绑定参数。同样，珂理化的处理方式是把一个有多个参数的函数转换为一个只接受一个参数的函数，它返回的函数接受剩余的参数。

这两者的差异现在看起来不是很明显，但最后会清楚的。

### 函数操作
在我们进一步解释如何实现部分应用和珂理化之前，我们需要进行一些回顾。如果我们想要扒掉javascript厚重的C风格语法外衣，暴露器函数是本质的话，我们需要理解原始函数、原型在javascript是如何工作的；而如果我们只是想设置一些cookie或验证一些表单的话则永远不用考虑这些。
 
*apply、call和this关键词*
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

### 绑定函数
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
 
### 函数工厂
还记得第二章《函数式编程基础》中关于闭包的那节吗？闭包使建立函数工厂这种Javascript编程模式成为可能。 它们使你能够手动绑定函数的参数。

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

### 部分应用
 
 http://www.cnblogs.com/tolg/p/4729696.html
 
 
