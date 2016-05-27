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

我们来看看上面的logger中间件的业务流程：
1. 得到当前的应用状态
2. 将action指派给下一个中间件
3. 调用链下游的中间件全部被执行
4. store中的匹配reducer被执行
5. 此时得到新的应用状态

# 5. 剖析applyMiddleware.js
下面让我们一步步分析applyMiddleware.js源码，applyMiddleware可能应该起一个更好一点的名字，applyMiddlewareToStore。
首先我们看一下方法的签名：
`
  export default function applyMiddleware(...middlewares)
`
注意这里有个很有趣的写法，参数：...middlewares，这么定义允许我们调用时传入任意个数的中间件函数作为参数。接下来函数将返回一个接受next作为参数的函数：
`
  return (next) => (reducer,initialStare) => {...}
`
next参数是一个被用来创建store的函数，你可以看一下createStore.js源码的实现细节。最后这个函数返回一个类似createStore的函数，不同的是它包含一个有中间件加工过的dispatch实现。

接下来我们通过调用next拿到store对象。我们用一个变量保存原始的dispatch函数，最后我们申明一个数组来存储我们创建的中间件链：
```javascript
  var store = next(reducer, initialStare);
  var dispatch = store.dispatch;
  var chain = [];
```

接下来的代码将getState 和调用原始的dispatch函数注入给所有的中间件：
```javascript
  var middlewareAPI = {
    getState: store.getState,
    dispatch: (action) => dispatch(action)
  };

  chain = middlewares.map(middleware => middleware(middlewareAPI));
```

然后我们根据中间件链创建一个加工过的dispatch实现：
```javascript
  dispatch = compose(...chain, store.dispatch);
```

最精妙的地方就是上面这行，Redux提供的compose工具函数组合了我们的中间件链，compose实现如下：
```javascript
  export default function compose(...funcs){
    return funcs.reduceRight((compose, f) => f(compose));
  }
```

碉堡了！上面的代码展示了中间件调用链是如何创建出来的。中间件调用链的顺序很重要，调用链类似下面这样：
```javascript
  middlewareI(middlewareJ(middlewareK(store.dispatch)))(action)
```

现在我们知道为啥我们要掌握复合函数和珂理化概念了。最后我们只需要将新的store和调整过的dispatch函数返回即可：
```javascript
  return {
    ...store,
    dispatch
  };
```

上面这种写法的意思是返回一个对象，该对象拥有store的所有属性，并增加一个dispatch函数属性，store里自带的那个原始dispatch函数会被覆盖。这种写法会被Babel转化成：
```javascript
  return _extends({}, store, {dispatch : _dispatch});
```

现在让我们将我们的logger中间件注入到dispatch中：
```javascript
  import { createStore, applyMiddleware } from 'redux';
  import loggerMiddleware from 'logger';
  import rootReducer from '../reducers';

  const createStoreWithMiddleware = applyMiddleware(loggerMiddleware)(createStore);

  export default function configureStore(initialStare){
    return createStoreWithMiddleware(rootReducer, initialStare);
  }

  const store = configureStore();
```

# 6. 异步中间件
我们已经会写基础的中间件了，我们就要玩更高深的，整个能处理异步action的中间件咋样？让我们来看一下redux-thunk的更多细节。我们假设有一个包含异步请求的action，如下：
```javascript
  function fetchQuote(symbol){
    requestQuote(symbol);
    return fetch(`http://www.google.com/finance/info?q=${symbol}`)
            .then(req => req.json())
            .then(json => showCurrentQuote(symbol,json));
  }
```

上面的代码并没有明显的调用dispatch来分派一个返回promise的action，我们需要使用redux-thunk中间件来延迟dispatch的执行：
```javascript
  function fetchQuote(symbol){
    return dispatch => {
      dispatch(requestQuote(symbol));
      return fetch(`http://www.google.com/finance/info?${symbol}`)
              .then(req => req.json())
              .then(json => dispatch(showCurrentQuote(symbol, json)));
    }
  }
```

注意这里的dispatch和getState是由applyMiddleware函数注入进来的。现在我们就可以分派最终得到的action对象到store的reducers了。下面是类似redux-thunk的实现：
```javascript
  export default function thunkMiddleware({ dispatch, getState}){
    return next => action => typeof action === 'function' ? action(dispatch,getState) : next(action);
  }
```

这个和你之前看到的中间件没什么太大不同。如果得到的action是个函数，就用dispatch和getState当作参数来调用它，否则就直接分派给store。你可以看一下Redux提供的更详细的异步示例。另外还有一个支持promise的中间件是redux-promise。

# 7. 使用middleware实现异步action和异步数据流
redux的生态在持续的完善，其中就有不少的middleware供开发者使用，同时大家也可以实现自己的middleware。
现在让我们来使用以下两个中间件来完成一个示例：

 - redux-thunk -- Redux-Thunk可以让你的action creator返回一个function而不是action。这可以用于延迟dispatch一个action或是在特定条件下dispatch才触发。他的内部函数接受store的dispatch和getState方法作为参数。
 - redux-logger -- Redux-Logger的用处很明显，就是用于记录所有action和下一次state的日志。
 - isomorphic-fetch: 用于ajax请求数据

# 总结
希望你已经了解了关于Redux中间件的足够信息，我也希望你掌握了更多的关于函数式编程的知识。我不断的尝试更多更好的函数式编程方法，尽管一开始并不容易，你需要不断的学习和尝试来参悟它的精髓。如果你完全掌握了这篇文章交给你的，那么你已经拥有了足够的信心去投入更多的学习当中。

最后，千万别使用那些你还没有搞明白的第三方类库，你必须确定它会给你的项目带来好处。掌握它的一个好方法就是去阅读它的源码，你将会学到新的编程技术，淘汰那些老的解决方案。将一个工具引入你的项目前，你有责任搞清楚它的细节。
