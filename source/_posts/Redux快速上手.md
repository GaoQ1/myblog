---
title: Redux快速上手
date: 2016-05-23 16:02:18
tags:
  - react
  - Redux
categories: 转载笔记
---
在实际的项目中，面对复杂业务逻辑的挑战，如何清晰高效的管理应用内的数据流动成为了关键。

Flux思想已经在提出后得到逐步推广，并广泛应用到实际项目中。facebook的flux实现，开源社区的reflux、redux等类库开始涌现并得到了广大开发者的认同和使用。

Redux以其简单易用、文档齐全易懂等优点在开源社区得到开发者的一直好评，所以接下来让我们一起走进Redux。

# 1. 基本介绍
React已经帮我们在视图层解决了禁止异步和直接操作DOM等问题，让页面的高效渲染和组件话开发成为了可能。美中不足的是，React依旧把处理state中数据的问题留给了你，那么，Redux的出现就是为了帮你解决这个问题

## 1.1 Fulx & Redux
最初，facebook官网提出了Flux思想管理数据流，同时也给了自己的实现方案flux来管理React应用。
![Flux](/images/react/flux.jpg)
1. 在view中触发action中的方法后
2. action发送dispatch
3. store接收新的数据进行合并，然后触发view中绑定在store上的方法
4. 最后通过修改局部state来改变view的展示

![Redux](/images/react/redux.jpg)
1. view直接触发dispatch
2. dispatch将action发送到reducer中后，根节点上会更新props，改变全局view
3. redux将view和store的绑定从手动编码中提取出来，放在了统一规范放在了自己的体系中

>相对于Flux而言，Redux的实现更简单，思路更清晰，写的代码也相对更少；只维护单一的 store.

## 1.2 对Redux的介绍
 - Redux是State容器，提供可预测化的状态管理
 - 它可以让你构建一致化的应用，运行于不同的环境(客户端、服务器、原生应用)，并且易于测试
 - 还提供了redux-devtools让开发者享受超爽的开发体验
 - 体小精悍(只有2kb)且没有任何依赖
 - 拥有丰富的生态圈：教程、开发者工具、路由、组件、中间件、工具集...

# 2. 快速上手
示例代码快速体验
```javascript
  //这是一个reducer，形式为(state,action) => state 的纯函数。描述了action如何把state转变成下一个state。
  //state的形式取决于你，可以是基本类型、数组、对象甚至是Immutable.js生成的数据结构。
  //唯一的要点是当state变化时需要返回全新的对象，而不是修改传入的参数。
  function counter(state=0,action){
    switch (action.type){
      case 'INCREMENT':
        return state + 1;
      case 'DECREMENT':
        return state - 1;
      default:
        return state;
    }
  }
  //创建Redux store来存放应用的状态。
  //API是{subscribe,dispatch,getState}
  let store = createStore(counter);
  //一个单纯渲染页面内容的函数
  const PureRender = () => {
    document.body.innerText = store.getState();
  }
  //可以手动订阅更新，也可以事件绑定到视图层
  store.subscribe(PureRender);
  //执行渲染函数
  PureRender();
  //改变内部state唯一方法是dispatch一个action。action可以被序列化，用日记记录和存储下来，后期还可以以回放的方式执行。
  document.addEventListener('click',function(e){
    //store dispatch 调度分发一个action(fire)
    store.dispatch({type:"DECREMENT"})
  })
```

# 3. 理解Redux的核心概念
## 3.1 Action & Action Creator
在Redux中，改变state只能通过action,它是store数据的唯一来源。一般来说你会通过store.dispatch()将action传到store。并且每个action都必须是javascript的简单对象，例如：
```
  {
    type:"ADD_TODO"，
    text:"Learn Redux"
  }
```

Redux要求action是可以被序列化的，这使得应用程序的状态保存、回放、Undo之类的功能可以被实现。因此，action中不能包含诸如函数调用这样的不可序列化的字段。
action的格式是有建议规范的，可以包含以下字段：
```
  {
    type:"ADD_TODO",
    payload:{
      text:"Do something"
    },
    `meta:{}`
  }
```

如果 action 用来表示出错的情况，则可能为：
```
  {
    type: 'ADD_TODO',
    payload: new Error(),
    error: true
  }
```
type 是必须要有的属性，其他都是可选的。完整建议请参考 Flux Standard Action(FSA) 定义。已经有不少第三方模块是基于 FSA 的约定来开发了。

**Action Creator**
事实上，创建 action 对象很少用这种每次直接声明对象的方式，更多地是通过一个创建函数。这个函数被称为Action Creator，例如：
```
  function addTodo(text) {
    return {
      type: ADD_TODO,
      text
    };
  }
```

Action Creator 看起来很简单，但是如果结合上 Middleware 就可以变得非常灵活，后面会专门讲 Middleware 。

## 3.2 Reducer
我们先来看一下javascript中Array.prototype.reduce的用法：
```javascript
  const initState = '';
  const actions = ['a','b','c'];
  //传入当前的state和action，返回新的state
  const newState = actions.reduce((curState,action) => curState + action);
  console.log(newState);
```

对应的理解，Redux中的reducer是一个纯函数，传入state和action，返回一个新的state tree,简单而纯粹的完成某一件具体的事情，没有依赖，**简单而纯粹是它的标签**
```javascript
  const counter = (state=0, action) => {
    switch(action.type){
      case 'INCREMENT':
        return state + 1;
      case 'DECREMENT':
        return state - 1;
      default:
        return state;
    }
  }
```

## 3.3 Store
Store就是用来维持应用所有的state树的一个对象。改变store内state的唯一途径是对它dispatch一个action。

Store是一个具有以下四个方法的对象：
 - getState()
 - dispatch(action)
 - subscribe(listener)
 - replaceReducer(nextReducer)

### 3.3.1 getState()
返回应用当前的state树。它与store的最后一个reducer返回值相同。
返回值：应用当前的state树。

### 3.3.2 dispatch(action)
dispatch分发action。**这是出发state变化的唯一途径。**
会使用当前getState()的结果和传入的action以同步方式的调用store的reduce函数。返回值会被作为下一个state。从现在开始，这就成为了getState()的返回值，同时变化监听器(change listener)会被触发。

>在Redux里，只会在根reducer返回新state结束后才会调用事件监听器，因此，你可以在事件监听器里再做dispatch。唯一使你不能在reducer中途diapatch的原因是要确保reducer没有副作用。如果action处理会产生副作用，正确的做法是使用异步action创建函数

示例:
```javascript
  import {createStore} from 'redux';
  //reducer
  const todos = (state = [''],action) => {
    switch (action.type){
      case 'ADD_TODO':
        return Object.assign([],state,[action.text]);
      default:
        return state;
    }
  }
  let store = createStore(todos,[ 'Use Redux'])
  //action Creator
  function addTodo(text){
    return {
      type: 'ADD_TODO',
      text
    }
  }
  //dispatch
  store.dispatch(addTodo('Read the docs'));
  store.dispatch(addTodo('Read about the middleware'));
```

### 3.3.3 subscribe(listener)
添加一个变化监听器。每当dispatch action的时候就会执行，state树中的一部分可能已经变化。这是一个底层API。多数情况下，你不会直接使用它，会使用一些React(或其他库)的绑定。

示例：
```javascript
  import {createStore} from 'redux';
  //reducer
  const todos = (state = [''],action) => {
    switch (action.type) {
      case 'ADD_TODO':
        return Object.assign([].state,[action.text]);
      default:
        return state;
    }
  }
  let store = createStore(todos,[ 'Use Redux'])
  //action Creator
  function addTodo(text){
    return{
      type:'ADD_TODO',
      text
    }
  }
  const handleChange = () => {
    console.log(store.getState());
  }
  let unsubscribe = store.subscribe(handleChange)
  handleChange()
  //dispatch
  store.dispatch(addTodo('Read the docs'));
  store.dispatch(addTodo('Read about the middleware'));
```

# 4. Redux的顶层API介绍
## 4.1 createStore
调用方式：createStore(reducer,[initialState])

创建一个Redux store来以存放应用中所有的state，应用中应有且仅有一个store。这个API返回一个store,这个store中保存了应用所有state的对象。改变state的唯一方法是dispatch action。你也可以subscrube监听state的变化，然后更新UI。我们来看一个示例。
**我们可以试着模拟createStore,深入了解其原理**
```javascript
  //reducer纯函数，具体的action执行逻辑
  const counter = (state = 0,action) =>{
    switch (action.type){
      case 'INCREMENT':
        return state + 1;        
      case 'DECREMENT':
        return state - 1;
      default:
        return state;
    }
  }
  //模拟createStore,了解其原理
  const createStore = (reducer) => {
    let state;
    let listeners = [];
    const getState = () => state;
    const dispatch = (action) => {
      state = reducer(state,action);
      listeners.forEach(listener => listener());
    }
    const subscribe = (listener) => {
      listeners.push(listener);
      return () => {
        listeners = listeners.filter(item => item !== listener);
      }
    }
    dispatch({});
    return {getState,dispatch,subscribe};
  }
  const store = createStore(counter);
  //view 对应到react里面的component
  const PureRender = () => {
    document.body.innerText = store.getState();
  }
  //store subscribe 订阅或是监听view(on)
  store.subscribe(PureRender);
  PureRender();
  document.addEventListener('click',function(e){
    //store dispatch 调度分发一个action(fire)
    store.dispatch({type:'DECREMENT'})；
  })
```

## 4.2 combineReducers
调用方式：combineReducers(reducers)

随着应用变得复杂，需要对reducer函数进行拆分，拆分后的每一块独立负责管理state的一部分。把一个由多个不同reducer函数作为value的object，合并成一个最终的reducer函数，然后就可以对这个reducer调用createStore.

示例如下：
代码清单：reducer/todos.js
```javascript
  export default function todos(state = [],action){
    switch(action.type){
      case 'ADD_TODO':
        return state.concat([action.text])
      default:
        return state
    }
  }
```

代码清单：reducer/counter.js
```javascript
  export default function counter(state = [],action){
    switch(action.type){
      case 'INCREMENT':
        return state + 1
      case 'DECREMENT':
        return state - 1
      default:
        return state
      }
    }
  }
```

代码清单：reducer/index.js
```javascript
  import { combineReducers } from 'redux'
  import todos from './todos'
  import counter from './counter'

  export default combineReducers({
    todos,
    counter
  })
```

代码清单：App.js
```javascript
  import {createStore} from 'redux'
  import reducer from './reducer/index.js'
  let store = createStore(reducer)
  console.log('当前的state: ',store.getState())
  store.dispatch({
    type:'ADD_TODO',
    text:'Use Redux'
  })
  store.dispatch({
    type:'INCREMENT'
  })
  console.log('改变后的state: ',store.getState());
```

## 4.3 applyMiddleware
调用方式: applyMiddleware(...middlewares)

使用包含自定义功能的middleware来扩展Redux是一种推荐的方式。Middleware可以让你包装store的dispatch方法来达到你想要的目的。同时，middleware还拥有“可组合”这一关键特性。多个middleware可以被组合到一起使用，形成middleware链。其中，每个middleware都不需要关心链中它前后的middleware的任何信息。

## 4.4 bindActionCreators
调用方式：bindActionCreators(actionCreators,dispatch)

唯一使用bindActionCreators的场景是当你需要把action creator往下传到一个组件上，却不想让这个组件察觉到Redux的存在，而且不希望把Redux store或dispatch传给它。

## 4.5 compose
调用方法：compose(...functions)

compose用来实现从右到左组合传入的多个函数，它做的只是让你不使用深度右括号的情况下来些深度嵌套的函数，仅此而已。

# 5. 使用React-redux连接react和redux
## 5.1 没有react-redux的写法
封装一个组件，将组件和Redux做基本的组合
```javascript
  import { createStore } from 'redux';
  import React, {Component} from 'react';
  import ReactDOM from 'react-dom';
  //reducer 纯函数，具体的action执行逻辑
  const counter = (state = 0,action) => {
    switch (action.type){
      case 'INCREMENT':
        return state + 1;
      case 'DECREMENT':
        return state - 1;
      default:
        return state;
    }
  }
  const store = createStore(counter);
  //Counter组件
  class Counter extends Component{
    render(){
      return (
        <div>
          <h1>{this.props.value}</h1>
          <button onClick={this.props.onIncrement}>点击加1</button>
          <button onClick={this.props.onDecrement}>点击减1</button>
        </div>
      )
    }
  }
  const PureRender = () => {
    ReactDOM.render(
      <Counter
        value={store.getState()}
        onIncrement={ () => store.dispatch({type: "INCREMENT"}) }
        onDecrement={ () => store.dispatch({type: "DECREMENT"}) }
      />, document.getElementById('app')
    )
  }
  // store subscribe 订阅或是监听view（on）
  store.subscribe(PureRender)
  PureRender()
```

## 5.2 React-redux提供的contect和Provider
<Provider store>使组件层级中的connect()方法都能够获得Redux store.正常情况下，你的根组件应该嵌套在'<Provider>'中才能使用connect()方法。
