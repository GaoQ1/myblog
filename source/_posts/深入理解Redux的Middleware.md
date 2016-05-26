---
title: 深入理解Redux的Middleware
date: 2016-05-26 14:22:35
tags:
  - react
  - Redux
categories: 转载笔记
---
> 文章转载自[深入理解Redux的Middleware](http://guoyongfeng.github.io/idoc/html/React%E8%AF%BE%E7%A8%8B%E4%B8%93%E9%A2%98/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3Redux%E7%9A%84Middleware.html)，仅供学习和参考

# 1. 中间件
Redux最有趣的一个概念是它允许你通过自定义的中间件来影响你store的dispatch逻辑。
> “中间件”这个词听起来很恐怖，但它实际一点都不难。想更好的了解中间件的方法就是看一下那些已经实现了的中间件是怎么工作的，然后尝试自己写一个。函数嵌套写法看起来很恐怖，但是大多数你能找到的中间件，代码都不超过十行，但是它们的强大来自于它们的可嵌套组合性。

区区十行的中间件很容易写，但是你要想明白它们是如何放入中间件调用链，又是如何影响store的dispatch方法的，还真需要一些经验。首先让我们来简单定义一下中间件到底是个啥，并且找一些简单的中间件看一下他们的具体实现方式。关于中间件的定义我能找到的最简单的描述是：
> 中间件主要被用于分离那些不属于你应用的核心业务逻辑，可以被组合起来使用的代码

Redux的中间件主要用于store的dispatch的函数上。dispatch函数的作用是发送actions给一个或多个reducer来影响应用状态。中间件可以增强默认的dispatch函数，我们来看一下Redux 1.0.1版本的applyMiddleware源码：
```javascript
  export default function applyMiddleware(...middlewares){
    return (next) => (reducer, initialStare) => {
      var store = next(reducer, initialStare);
      var dispatch = store.dispatch;
      var chain = [];

      var middlewareAPI = {
        getState: store.getState,
        dispatch: (action) => dispatch(action)
      };

      chain = middlewares.map(middleware => middleware(middlewareAPI));
      dispatch = compose(...chain, store.dispatch);
      return {
        ...store,
        dispatch
      }
    }
  }
```

就这几行代码，跟着敲一下也就一会儿时间，但是什么鬼，完全看不懂。因为里面用了很多的函数式编程的思想，包括：高阶函数、复合函数、珂理化和ES6语法。关于函数式编程思想，我也转载了不少文章，但理解起来需要花点时间。

# 2. 函数式编程概念
在开始阅读Redux中间件源码之前，你需要先掌握一些函数式编程知识。
## 2.1 复合函数
函数式编程是非常理论和非常数学化的。用数学的视角来解释复合函数，如下：
```javascript
  given:
    f(x) = x^2 + 3*x + 1
    g(x) = 2*x
  then:
     (f ∘ g)(x) = f(g(x)) = f(2*x) = 4*x^2 + 6*x + 1
```
你可以将上面的方式扩展到组合两个或更多个函数这都是可以的。我们再来看个例子，演示组合两个函数并返回一个新的函数：
```javascript
  var greet = function(x){ return `Hello, ${x}` };
  var emote = function(x){ return `${x}` };

  var compose = function(f,g){
    return function(x){
      return f(g(x));
    }
  }

  var happyGreeting = compose(green, emote);
  //happyGreeting("Mark") -> Hello,Mark
```

## 2.2 珂理化
珂理化是这样一个过程：它把一个具有多个参数的函数转换为一个只有一个参数的函数并返回另一个函数，这个被返回的函数需要原函数的参数。正式的说法是：一个具有N个参数的函数可以被转换为具有N个函数的函数链，其中每一个函数只有一个参数。
```javascript
  var curriedAdd = function(a){
    return function(b){
      return a + b;
    }
  };

  var addTen = curriedAdd(10);
  addTen(10); //20
```
通过珂理化来组合你的函数，你可以创建一个强大的数据处理管道。

# 3. Redux的dispatch函数
Redux的Store有一个dispatch函数，它关注你的应用实现的业务逻辑。你可以用它指派actions到你定义的reducer函数，用以更新你的应用状态。Redux的reducer函数接受一个当前状态参数和一个action参数，并返回一个新的状态对象：
`
  reducer:: state -> action -> state
`
指派给action很像发送消息，如果我们假设要从一个列表中删除某个元素，action结构一般如下：
`
  {type: types.DELETE_ITEM, id:1 }
`
store会指派这个action对象到它所拥有的所有reducer函数来影响应用的状态，然而只有关注删除逻辑的reducer会真的修改状态。在此期间没人会关注到底是谁修改了状态，花了多长时间，或者记录一下变更前后的状态数据镜像。这些非核心关注点都可以交给中间件来完成。

# 4. Redux Middleware
Redux中间件被设计成可组合的，会在dispatch方法之前调用的函数。让我们来创建一个简单的日志中间件，他会简单的输出dispatch前后的应用状态。Redux中间件的签名如下：
`
Middleware:: next -> action -> retVal
`
我们的logger中间件实现如下：
```javascript
  export default function createLogger({getState}){
    return (next) => (action) => {
      const console = window.console;
      const prevState = getState();
      const returnValue = next(action);
      const nextState = getState();
      const actionType = String(action.type);
      const message = `action ${actionType}`;

      console.log(`%c prev state`, `color: #9E9E9E`, prevState);
      console.log(`%c action`, `color: #03A9F4`, action);
      console.log(`%c next state`, `color: #4CAF50`, nextState);
      return returnValue;
    }
  }
```
注意，我们的createLogger接受的getState方法是由applyMiddleware.js注入进来的。使用它可以在内部的闭包中得到应用的当前状态。最后我们返回调用next创建的函数作为结果。next方法用于维护中间件调用链和dispatch，它返回一个接受action对象的柯里化函数，接受的action对象可以在中间件中被修改，再传递给下一个被调用的中间件，最终dispatch会使用中间件修改后的action来执行。
