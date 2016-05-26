---
title: Redux快速上手
date: 2016-05-23 16:02:18
tags:
  - react
  - Redux
categories: 转载笔记
---
> 文章转载自[Redux快速上手](http://guoyongfeng.github.io/idoc/html/React%E8%AF%BE%E7%A8%8B%E4%B8%93%E9%A2%98/Redux%E5%BF%AB%E9%80%9F%E4%B8%8A%E6%89%8B.html)，仅供学习和参考

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
    dispatch();
    return { getState, dispatch, subscribe };
    const store = createStore(counter);
    // view 对应到React里面的component
    const PureRender = () => {
      document.body.innerText = store.getState();
    }
    // store subscribe 订阅或是监听view（on）
    store.subscribe(PureRender);
    PureRender();
    document.addEventListener('click', function( e ){
      // store dispatch 调度分发一个 action（fire）
      store.dispatch({ type: 'DECREMENT'});
    })
  }
```

## 4.2 combineReducers
调用方式：combineReducers(reducers)

随着应用变得复杂，需要对 reducer 函数进行拆分，拆分后的每一块独立负责管理 state 的一部分。把一个由多个不同 reducer 函数作为 value 的 object，合并成一个最终的 reducer 函数，然后就可以对这个 reducer 调用 createStore。

示例如下

代码清单：reducer/todos.js
```javascript
  export default function todos(state = [], action) {
    switch (action.type) {
    case 'ADD_TODO':
      return state.concat([action.text])
    default:
      return state
    }
  }
```

代码清单：reducer/counter.js
```javascript
  export default function counter(state = 0, action) {
    switch (action.type) {
    case 'INCREMENT':
      return state + 1
    case 'DECREMENT':
      return state - 1
    default:
      return state
    }
  }
```

代码清单：reducers/index.js
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
  import { createStore } from 'redux'
  import reducer from './reducer/index.js'

  let store = createStore(reducer)
  console.log('当前的 state :', store.getState())

  store.dispatch({
    type: 'ADD_TODO',
    text: 'Use Redux'
  })
  store.dispatch({
    type: 'INCREMENT'
  })
  console.log('改变后的 state :', store.getState())
```

## 4.3 applyMiddleware
调用方式：applyMiddleware(...middlewares)

使用包含自定义功能的 middleware 来扩展 Redux 是一种推荐的方式。Middleware 可以让你包装 store 的 dispatch 方法来达到你想要的目的。同时， middleware 还拥有“可组合”这一关键特性。多个 middleware 可以被组合到一起使用，形成 middleware 链。其中，每个 middleware 都不需要关心链中它前后的 middleware 的任何信息

## 4.4 bindActionCreators
调用方式：bindActionCreators(actionCreators, dispatch)

惟一使用 bindActionCreators 的场景是当你需要把 action creator 往下传到一个组件上，却不想让这个组件觉察到 Redux 的存在，而且不希望把 Redux store 或 dispatch 传给它。

## 4.5 compose
调用方式：compose(...functions)

compose 用来实现从右到左来组合传入的多个函数，它做的只是让你不使用深度右括号的情况下来写深度嵌套的函数，仅此而已。

# 5 使用React-redux连接react和redux
## 5.1 没有React-redux的写法
封装一个组件，将组件和Redux做基本的组合
```javascript
  import { createStore } from 'redux';
  import React, { Component } from 'react';
  import ReactDOM from 'react-dom';

  // reducer 纯函数，具体的action执行逻辑
  const counter = (state = 0, action) => {
    switch (action.type) {
        case 'INCREMENT':
          return state + 1;
        case 'DECREMENT':
          return state - 1;
        default:
          return state;
    }
  }

  const store = createStore(counter);

  // Counter 组件
  class Counter extends Component {
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
    );
  }

  // store subscribe 订阅或是监听view（on）
  store.subscribe(PureRender)
  PureRender()
```

## 5.2 React-redux提供的connect和Provider
<Provider store>使组件层级中的connect()方法都能够获得Redux store.正常情况下，你的根组件应该嵌套在<Provider>中才能使用connect()方法。
```javascript
  ReactDOM.render(
    {/*  使组件层级中的 connect() 方法都能够获得 Redux store */}
    <Provider store={store}>
      {/* 这里传入的组件MyRootComponent是组件层级的根组件 */}
      <MyRootComponent />
    </Provider>
  )
```

connect([mapStateToProps],[mapDispatchToProps],[mergeProps],[options]) connect方法是来连接React组件与Redux store,连接操作不会改变原来的组件类，反而返回一个新的已与Redux store连接的组件类。

使用React-redux的一个简单完整示例
```javascript
  import React, { Component, PropTypes} from 'react';
  import ReactDOM from 'react-dom';
  import { createStore } from 'redux';
  import { Provider, connect} from 'react-redux';
  //这是一个展示型组件counter
  class Counter extends Component {
    render(){
      const { value, onIncrementClick} = this.props;
      return (
        <div>
          <span>{value}</span>
          <button onClick={onIncrementClick}>点我加一</button>
        </div>
      )
    }
  }
  Counter.propTypes = {
    value: PropTypes.number.isRequired,
    onIncrementClick: PropTypes.func.isRequired
  }
  //Action
  const increaseAction = {type: 'increase'}
  //Reducer
  function counter(state={count:0},action){
    let count = state.count;
    switch(action.type){
      case 'increase':
        return {count:count + 1}
      default:
        return count
    }
  }
  //store
  let store = createStore(counter);
  //Map Redux state to component props
  function mapStateToProps(state){
    console.log(state);
    //这里拿到的state就是store里面给的state
    return {
      value: state.count
    }
  }

  //Map Redux actions to component props
  function mapDispatchToProps(dispatch){
    //dispatch
    return {
      onIncrementClick: () => dispatch(increaseAction);
    }
  }

  class App extends Component{
    render(){
      //store里的state经过connect连接后给了根组件的props
      console.log(this.prps);
      return (
        <div>
          <h1>react-redux</h1>
          <Counter {...this.props} />
        </div>
      )
    }
  }
  //Connected Component
  let RootApp = connect(
    mapStateToProps,
    mapDispatchToProps
  )(App)
  ReactDOM.render(
    <Provider store={store}>
      <RootApp />
    </Provider>,
    document.getElementById('app')
  )
```
实际应用中，connect这个部分会比较复杂。

# 6. 一步步开发一个TODO应用
## 6.1 入口文件
index.js
```javascript
  import React from 'react';
  import {render} from 'react-dom';
  import {createStore} from 'redux';
  import {Provider} from 'react-redux';
  import App from './containers/App';
  import todoApp from './reducers';

  let store = createStore(todoApp);

  let rootElement = document.getElementById('app');
  render(
    <Provider store={store}>
      <App />
    </Provider>,
    rootElement
  )
```

## 6.2 Action创建函数和常量
action.js
```javascript
  //action类型
  export const ADD_TODO = 'ADD_TODO';
  export const COMPLETE_TODO = 'COMPLETE_TODO';
  export const SET_VISIBILITY_FILTER = 'SET_VISIBILITY_FILTER';
  //其他常量
  export const VisibilityFilter = {
    SHOW_ALL: 'SHOW_ALL',
    SHOW_COMPLETED: 'SHOW_COMPLETED',
    SHOW_ACTIVE: 'SHOW_ACTIVE'
  }
  //action创建函数
  export function addTodo(text){
    return {type: ADD_TODO,text}
  }
  export function completeTodo(index){
    return {type: COMPLETE_TODO,index}
  }
  export function setVisibilityFilter(filter){
    return {type: SET_VISIBILITY_FILTER,filter}
  }
```

## 6.3 Reducers
reducers.js
```javascript
  import {combineReducers} from 'redux';
  import {ADD_TODO, COMPLETE_TODO, SET_VISIBILITY_FILTER, VisibilityFilter} from './actions';
  const {SHOW_ALL} = VisibilityFilter;

  function visibilityFilter(state = SHOW_ALL, action){
    switch(action.type){
      case SET_VISIBILITY_FILTER:
        return action.filter
      default:
        return state
    }
  }

  function todos(state=[],action){
    switch(action.type){
      case ADD_TODO:
        return [
          ...state,
          {
            text:action.text,
            completed: false
          }
        ]
      case COMPLETE_TODO:
        return [
          ...state.slice(0,action.index),
          Object.assign({},state[action.index],{
            completed:true
          }),
          ...state.slice(action.index + 1)
        ]
      default:
        return state
    }
  }

  const todoApp = combineReducers({
    visibilityFilter,
    todos
  })

  export default todoApp
```

## 6.4 容器组件
containers/App.js
```javascript
  import React, {Component, PropTypes} from 'react';
  import {connect} from 'react-redux';
  import {addTodo, completeTodo, setVisibilityFilter, VisibilityFilter} from '../actions';
  import AddTodo from '../components/AddTodo';
  import TodoList from '../components/TodoList';
  import Footer from '../components/Footer';

  class App extends Component {
    render(){
      //Injected by connect() call
      const {dispatch, visibleTodos, visibilityFilter } = this.props;
      return (
        <div>
          <AddTodo onAddClick={text =>
            dispatch(addTodo(text))
          } />
          <TodoList
            todos={visibleTodos}
            onAddClick={index =>
              dispatch(completeTodo(index))
          } />
          <Footer
            filter={visibilityFilter}
            onFilterChange={nextFilter =>
              dispatch(setVisibilityFilter(nextFilter))
          } />
        </div>
      )
    }
  }

  App.propTypes = {
    visibleTodos: PropTypes.arrayOf(PropTypes.shape({
      text: PropTypes.string.isRequired,
      completed: PropTypes.bool.isRequired
    }).isRequired).isRequired,
    visibilityFilter: PropTypes.oneOf({
      'SHOW_ALL',
      'SHOW_COMPLETED',
      'SHOW_ACTIVE'
    }).isRequired
  }

  function selectTodos(todo,filter){
    switch(filter){
      case visibilityFilters.SHOW_ALL:
        return todos
      case VisibilityFilters.SHOW_COMPLETED:
        return todos.filter(todo => todo.completed)
      case VisibilityFilters.SHOW_ACTIVE:
        return todos.filter(todo => !todo.completed)
    }
  }

  function select(state){
    return {
      visibleTodos: selectTodos(state.todos,state.visibilityFilter),
      visibilityFilter: state.visibilityFilter
    }
  }
  // 包装 component ，注入 dispatch 和 state 到其默认的 connect(select)(App) 中；
  export default connect(select)(App)
```

## 6.5 展示组件
components/AddTodo.js
```javascript
  import React, {Component, PropTypes} from 'react';

  export default class AddTodo extends Component {
    render() {
      return (
        <div>
          <input type='text' ref='input' />
          <button onClick={(e) => this.handleClick(e)}>
            Add
          </button>
        </div>
      )
    }

    handleClick(e) {
      const node = this.refs.input
      const text = node.value.trim()
      this.props.onAddClick(text)
      node.value = ''
    }
  }

  AddTodo.propTypes = {
    onAddClick: PropTypes.func.isRequired
  }
```

components/Footer.js
```javascript
  import React, { Component, PropTypes } from 'react'

  export default class Footer extends Component {
  renderFilter(filter, name) {
    if (filter === this.props.filter) {
      return name
    }

    return (
      <a href='#' onClick={e => {
        e.preventDefault()
        this.props.onFilterChange(filter)
      }}>
        {name}
      </a>
    )
  }

  render() {
    return (
      <p>
        Show:
        {' '}
        {this.renderFilter('SHOW_ALL', 'All')}
        {', '}
        {this.renderFilter('SHOW_COMPLETED', 'Completed')}
        {', '}
        {this.renderFilter('SHOW_ACTIVE', 'Active')}
        .
      </p>
    )
  }
}

  Footer.propTypes = {
    onFilterChange: PropTypes.func.isRequired,
    filter: PropTypes.oneOf([
      'SHOW_ALL',
      'SHOW_COMPLETED',
      'SHOW_ACTIVE'
    ]).isRequired
  }
```

components/Todo.js
```javascript
  import React, { Component, PropTypes } from 'react'

  export default class Todo extends Component {
    render() {
      return (
        <li
          onClick={this.props.onClick}
          style={{
            textDecoration: this.props.completed ? 'line-through' : 'none',
            cursor: this.props.completed ? 'default' : 'pointer'
          }}>
          {this.props.text}
        </li>
      )
    }
  }

  Todo.propTypes = {
    onClick: PropTypes.func.isRequired,
    text: PropTypes.string.isRequired,
    completed: PropTypes.bool.isRequired
  }
```

components/TodoList.js
```javascript
  import React, { Component, PropTypes } from 'react'
  import Todo from './Todo'

  export default class TodoList extends Component {
    render() {
      return (
        <ul>
          {this.props.todos.map((todo, index) =>
            <Todo {...todo}
                  key={index}
                  onClick={() => this.props.onTodoClick(index)} />
          )}
        </ul>
      )
    }
  }

  TodoList.propTypes = {
    onTodoClick: PropTypes.func.isRequired,
    todos: PropTypes.arrayOf(PropTypes.shape({
      text: PropTypes.string.isRequired,
      completed: PropTypes.bool.isRequired
    }).isRequired).isRequired
  }
```
