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

## 参数
 - [mapStateToProps(state,[ownProps]): stateProps](Function): 如果定义该参数，组件将会监听Redux store的变化。任何时候，只要Redux store发生变化，mapStateToProps函数就会被调用。该回调函数必须返回一个纯对象，这个对象会与组件的props合并。如果你省略了这个参数，你的组件将不会监听Redux store。如果指定了该回调函数中的第二个参数ownProps，则该参数的值为传递到组件的props，而且只要组件接收到新的props，mapStateToProps也会被调用。
 - [mapDispatchToProps(dispatch,[ownProps]): dispatchProps](Object or Function)：如果传递的是一个对象，那么每个定义在该对象的函数都将被当作Redux action creator，而且这个对象会与Redux store绑定在一起，其中所定义的方法名将作为属性名，合并到组件的props中。如果传递的是一个函数，该函数将接收一个dispatch函数，然后由你来决定如何返回一个对象，这个对象通过dispatch函数与action creator以某种方式绑定在一起（提示：你也许会用到Redux的辅助函数bindActionCreators()）.如果你省略这个mapDispatchToProps参数，默认情况下，dispatch会注入到你的组件props中。如果你指定了该回调函数中第二个参数ownPros，该参数的值未传递到组件的props，而且只要组件接收到新props，mapDispatchToProps也会被调用。
 - [mergeProps(stateProps,dispatchProps,ownPros): props](Function):如果指定了这个参数，mapStateToProps()与mapDispatchToProps()的执行结果和组件自身的props将传入到这个回调函数中。该回调函数返回的对象将作为props传递到被包装的组件中。你也许可以用这个回调函数，根据组件的props来筛选部分的state数据，或者把props中的某个特定变量与action creator绑定在一起。如果你省略这个参数，默认情况下返回Object.assign({},ownPros,stateProps,dispatchProps)的结果。
 - [options](Object) 如果指定这个参数，可以定制connector的行为。
  - [pure = true](Boolean): 如果为true,connector将执行shouldComponentUpdate并且浅对比mergeProps的结果，避免不必要的更新，前提是当前组件是一个"纯组件"，它不依赖于任何的输入或state而只依赖于props和Redux store的state。默认值为true。
  - [withRef = false](Boolean): 如果为true，connector会保存一个对被包装组件实例的引用，该引用通过getWrappedInstance()方法获得。默认值为false

## 返回值
根据配置信息，返回一个注入了state和action creator的React组件。

## 静态属性
 - WrappedComponent(Component): 传递到connect()函数的原始组件类。

## 静态方法
组件原来的静态方法都被提升到被包装的React组件。

## 实例方法
getWrappedInstance(): ReactComponent
仅当connect()函数的第四个参数options设置了{withRef: true}才返回被包装的组件示例。

## 备注
 - 函数将被调用两次。第一次是设置参数，第二次是组件与React store连接：connect(mapStateToProps,mapDispatchToProps,mergeProps)(MyComponent).
 - connect函数不会修改传入的React组件，返回的是一个新的已与Redux store连接的组件，而且你应该使用这个新组件。
 - mapStateToProps函数接收整个Redux store的state作为props，然后返回一个传入到组件props的对象。该函数被称之为selector。参考使用reselect高效地组合多个selector，并对收集到的数据进行处理。

## Example例子
**只注入dispatch，不监听store**
`export default conenct(){TodoApp};`

**注入dispatch和全局state**
> 不要这样做！这会导致每次action都触发整个TodoApp重新渲染，你做的所有性能优化都将付之东流。最好在多个组件上使用connect()，每个组件只监听它所关联的部分state.  

`export default connect(state => state)(TodoApp)`

**注入dispatch和todos**
```javascript
  function mapStateToProps(state){
    return {
      todos: state.todos
    };
  }

  export default conenct(mapStateToProps)(TodoApp);
```

**注入todos和所有action creator(addTodo,completeTodo,...)**
```javascript
  import * as actionsCreators from './actionsCreators';

  function mapStateToProps(state){
    return {
      todos: state.todos
    }
  }

  export default connect(mapStateToProps,actionsCreators)(TodoApp);
```

**注入todos并把所有action creator作为actions属性也注入组件中**
```javascript
  import * as actionsCreators from './actionsCreators';
  import { bindActionCreators } from 'redux';

  function mapStateToProps(state){
    return {todos: state.todos};
  }

  function mapDispatchToProps(dispatch){
    return {actions: bindActionCreators(actionsCreators,dispatch)};
  }

  export dfault connect(mapStateToProps,mapDispatchToProps)(TodoApp)
```

**注入todos和指定的action creator(addTodo)**
```javascript
  import { addTodo } from './actionsCreators';
  import { bindActionCreators } from 'redux';

  function mapStateToProps(state){
    return {
      todo: state.todos
    }
  }

  function mapDispatchToProps(dispatch){
    return bindActionCreators({addTodo},dispatch);
  }

  export default connect(mapStateToProps,mapDispatchToProps)(TodoApp);
```

**注入todos并把todoActionCreators作为todoActions属性、counterActionCreators作为counterActions属性注入到组件中**
```javascript
  import * as todoActionCreators from './todoActionCreators';
  import * as counterActionCreators from './counterActionCreators';
  import { bindActionCreators } from 'redux';

  function mapStateToProps(state){
    return {todos: state.todo};
  }

  function mapDispatchToProps(dispatch){
    return {
      todoActions: bindActionCreators(todoActionCreators,dispatch),
      counterActions: bindActionCreators(counterActionCreators,dispatch)
    };
  }

  export default connect(mapStateToProps, mapDispatchToProps)(TodoApp);
```

**注入todos并把todoActionCreators与counterActionCreators一同作为actions属性注入到组件中**
```javascript
  import * as todoActionCreators from './todoActionCreators';
  import * as counterActionCreators from './counterActionCreators';
  import { bindActionCreators } from 'redux';

  function mapStateToProps(state){
    return { todos: state.todos };
  }

  function mapDispatchToProps(dispatch){
    return {
      actions: bindActionCreators(Object.assign({},todoActionCreators,counterActionCreators),dispatch)
    }
  }

  export default connect(mapStateToProps,mapDispatchToProps)(TodoApp);
```

**注入todos并把所有的todoActionCreators和counterActionCreators作为props注入到组件中**
```javascript
  import * as todoActionCreators from './todoActionCreators';
  import * as counterActionCreators from './counterActionCreators';
  import { bindActionCreators } from 'redux';

  function mapStateToProps(state){
    return {todos: state.todo}
  }

  function mapDispatchToProps(dispatch){
    return bindActionCreators(Object.assign({}, todoActionCreators,counterActionCreators),dispatch);
  }

  export default connect(mapStateToProps,mapDispatchToProps)(TodoApp);
```

**根据组件的props注入特定用户的todos**
```javascript
  import * as actionsCreators from './actionsCreators';

  function mapStateToProps(state,ownProps){
    return {todos: state.todos[ownProps.userId]};
  }

  export default connect(mapStateToProps)(TodoApp);
```

**根据组件的props注入特定用户的todos并把props.userId传入到action中**
```javascript
  import * as actionsCreators from './actionsCreators';

  function mapStateToProps(state){
    return {todos: state.todos};
  }

  function mergeProps(stateProps,dispatchProps,ownProps){
    return Object.assign({},ownProps,{
      todos: stateProps.todos[ownProps.userId],
      addTodo: (text) => dispatchProps.addTodo(ownProps.userId,text)
    });
  }

  export default connect(mapStateToProps,actionsCreators,mergeProps)(TodoApp);
```

# 排错
这里会列出常见的问题和对应的解决方案。
