---
title: React-Redux文档
date: 2016-05-31 17:58:31
tags:
  - react
  - Redux
categories: 转载笔记
---
> It's an imperfect world, but it's the only one we've got.  

# 快速开始
在应用中，只有最顶层组件是对Redux可知（例如路由处理）。所有它们的子组件都应该是“笨拙的”，并且 通过props获取数据。

|  | **容器组件** | **展示组件** |
| ------ | :-------: | :----------- |
| **位置**	| 最顶层，路由处理 | 中间和子组件 |
| **使用Redux**	| 是 | 否 |
| **读取数据**	| 从Redux获取state | 从props获取数据 |
| **修改数据**	| 向Redux发起actions | 从props调用回调函数 |

## 不使用Redux的展示组件
让我们看下，我们拥有一个<Counter />的展示组件，它有一个通过props传过来的值，和一个函数onIncrement，当你点击"Increment"按钮时就会调用这个函数：
```javascript
  import { Component } from 'react';

  export default class Counter extends Component {
    render(){
      return {
        <button onClick={this.props.onIncrement}>
          {this.props.value}
        </button>
      }
    }
  }
```

## 容器组件使用connect() 方法连接Redux
我们用react-redux提供的connect()方法将“笨拙”的Counter转化成容器组件。connect()允许你从Redux store中指定准确的state到你想要获取的组件中。这让你能获取到任何级别颗粒度的数据。

`containers/CounterContainer.js`

```javascript
  import { Compoennt } from 'react';
  import { connect } from 'react-redux';

  import Counter from '../components/Counter';
  import { increment } from '../actionsCreators';

  //哪些Redux全局的state是我们组件想要通过props获取的？
  function mapStateToProps(state){
    return {
      value: state.counter
    }
  }

  //哪些action创建函数是我们想要通过props获取的？
  function mapDispatchToProps(dispatch){
    return {
      onIncrement: () => dispatch(increment())
    };
  }

  export default connect(
    mapStateToProps,
    mapDispatchToProps
  )(Counter);

  //你可以传递一个对象，而不是定义一个`mapDispatchToProps`;
  //export default connect(mapStateToProps, CounterActionCreators)(Counter);
  //或者如果你想省略`mapDispatchToProps`,你可以通过传递一个`dispatch`作为一个props:
  //export default connect(mapStateToProps)(Counter);

```
作为一个展示组件，无论是在同一个文件中调用connect()方法，还是分开调用，都取决于你。你应该考虑的是，是否重用这个组件半丁不同数据。

## 嵌套
在你的应用任何层次中，你可以拥有很多使用connect()的组件，甚至你可以把它们嵌套使用。的确如此，但是更好的做法是只在最顶层的组件中使用connect(),例如路由处理，这使得应用中的数据流是保持可预知的。

## 修饰器的支持
你可能会注意到，我们在调用connect()方法的时候使用了两个括号。这个叫做局部调用，并且这允许开发者使用ES7提供的修饰语法：
```javascript
  //这是还不稳定的写法！可能在实际的应用中被修改或摒弃
  @conenct(mapStateToProps)
  export default class CounterContainer {...}
```
不要忘了修饰器还在实验中！在以下的示范中，它们被提取出来，作为一个可以在任何地方调用的函数例子。

## 额外的灵活性
这是最基础的用法，但connect()也支持很多其他的模式：通过传递一个普通的dispatch()方法，绑定多个action创建函数，把它们传递到一个action prop中，选择一部分state和绑定的action创建函数依赖到props中。

## 注入Redux store
最后，我们实际上是怎么连接到 Redux store 的呢？我们需要在根组件中创建这个 store。对于客户端应用而言，根组件是一个很好的地方。对于服务端渲染而言，你可以在处理请求中完成这个。

关键是从 React Redux 将整个视图结构包装进 <Provider>。
```javascript
  import ReactDOM from 'react-dom';
  import { Component } from 'react';
  import { Provider } from 'react-redux';

  class App extends Component {
    render() {
      // ...
    }
  }

  const targetEl = document.getElementById('root');

  ReactDOM.render(
    <Provider store={store}>
      <App />
    </Provider>,
    targetEl
  );
```

# API
## <Provider store>
<Provider store>是组件层级中connect()方法都能获得Redux store。正常情况下，你的根组件应该嵌套在<Provider>中才能使用connect()方法。

如果你真的不想把根组件嵌套在<Provider>中，你可以把store作为props传递到每一个被connect()包装的组件，但是我们只推荐您在单元测试中对store进行伪造(stub)或者在非完全基于React的代码中才这样做。正常情况下，你应该使用<Provider>

## 属性
 - store(Redux Store): 应用程序中唯一的Redux store对象
 - children(ReactElement) 组件层级的根组件

## 例子
**React**
```javascript
  ReactDOM.render(
    <Provider store={store}>
      <MyRootComponent />
    </Provider>,
    rootEL
  )
```

**React Router 1.0**
```javascript
  ReactDOM.render(
    <Provider store={store}>
      <Router history={history}>...</Router>
    </Provider>,
    targetEl
  )
```

`connect([mapStateToProps],[mapDispatchToProps],[mergeProps],[options])`
连接React组件与Redux store.

连接操作不会改变原来的组件类，反而返回一个新的已与 Redux store 连接的组件类。

http://cn.redux.js.org/docs/react-redux/api.html
