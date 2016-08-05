---
title: Immutable详解及React中实践
date: 2016-05-09 23:52:22
tags:
- immutable
- react
categories: 教程
---
> Shared mutable state is the root of all evil

有人说Immutable可以给React应用带来数十倍的提升，也有人说Immutable的引入是近期Javascript中伟大的发明，因为同期React太火，它的光芒被掩盖了。这些至少说明Immutable是很有价值的。

Javascript中的对象一般是可变的(mutable)，因为使用了引用赋值，新的对象简单的引用了原始对象，改变新的对象将影响到原始对象。虽然这样做可以节约内存，但是当应用复杂后，这就造成了非大的隐患，Mutable带来的优点变得得不偿失。为了解决这个问题，一般的做法是使用shallowCopy(浅拷贝)或deepCopy(深拷贝)来避免被修改，但这样做造成了CPU和内存的浪费。

而Immutable可以很好的解决这些问题。

##什么是Immutable Data
Immutable Data就是一旦创建，就不能再被更改的数据。对Immutable对象的任何修改或添加删除操作都会返回一个新的Immutable对象。Immutable实现的原理是Persistent Data Structure(持久化数据结构)，也就是使用旧的数据创建新的数据时，要保证旧数据同时可用且不变。同时为了避免deepCopy把所有的节点的都复制一遍带来的性能损耗，Immutable使用了Structure Sharing(结构共享)，即如果对象树中一个节点发生变化，只修改这个节点和受它影响的父节点，其他节点则进行共享。

目前流行的Immutable库有两个：

### immutable.js
Facebook工程师Lee Byron花费3年时间打造，与React同期出现，但没有被默认放到React工具库里(React提供了简化的Helper)。它内部实现了一套完整的Persistent Data Structure，还有很多易用的数据类型。像Collection、List、Map、Set、Record、Seq.有非常全面的map、filter、groupBy、reduce `find`函数式操作方法。同事API也尽量与Object或Array类似。
其中有3种最重要的数据结构：
- Map:键值对集合，对应于Object,ES6也有专门的Map对象
- List: 有序可重复的列表，对应于Array
- Set: 无序且不可重复的列表

### seamless-immutable
与 Immutable.js 学院派的风格不同，seamless-immutable 并没有实现完整的 Persistent Data Structure，而是使用 Object.defineProperty（因此只能在 IE9 及以上使用）扩展了 JavaScript 的 Array 和 Object 对象来实现，只支持 Array 和 Object 两种数据类型，API 基于与 Array 和 Object 操持不变。代码库非常小，压缩后下载只有 2K。而 Immutable.js 压缩后下载有 16K。

下面是例子
```javascript
    //原来的写法
    let foo = {a:{b:1}};
    let bar = foo;
    bar.a.b = 2;
    console.log(foo.a.b); //2
    console.log(foo === bar); //true
    //使用immutable.js后
    import Immutable from 'immutable';
    foo = Immutable.fromJS({a:{b:1}});
    bar = foo.setIn(['a','b'],2); // 使用setIn赋值
    console.log(foo.getIn(['a','b'])); //1
    console.log(foo === bar); // false
    //使用seamless-immutable.js后
    import SImmutable from 'seamless-immutable';
    foo = SImmutable({a:{b:1}});
    bar = foo.merge({a:{b:2}}); //使用merge赋值
    console.log(foo.a.b); //1
    console.log(foo === bar); //false
```

## Immutable优点
1. Immutable降低了Mutable带来的复杂度
可变(Mutable)数据耦合了Time和Value的概念，造成了数据很难被回溯。
比如下面的一段代码：

```javascript
    function touchAndLog(touchFn){
        let data = {key:'value'};
        touchFn(data);
        console.log(data.key);
    }
```

在不查看touchFn的代码的情况下，因为不确定它对data做了什么，你是不可能知道会打印什么的。但是如果data是Immutable，你可以知道打印的是value。

2. 节省内存
Immutable.js使用了Structure Sharing 会尽量复用内存，甚至以前使用的对象也可以再次被复用。没有被引用的对象会被垃圾回收。

```javascript
    import {Map} from 'immutable';
    let a = Map({
        select: 'users',
        filter: Map({name:'Tom'})
    })
    let b = a.set('select','people');
    a===b; //false
    a.get('filter') === b.get('filter') //true
```

上面的a和b共享了没有变化的filter节点。

3. Undo/Redo, Copy/Paste, 甚至时间旅行这些功能做起来小菜一碟
因为每次数据都是不一样的，只要把这些数据放到一个数组里存储起来，想回退到哪里就拿出对应的数据即可，很容易开发出撤销重做这种功能。

4. 并发安全
传统的并发非常难做，因为要处理各种数据不一致问题，因此有人就发明了各种锁来解决。但是使用了Immutable之后，数据天生是不可变的，并发锁就不需要了。

然而现在并没有什么卵用，因为Javascript是单线程运行的，但未来可能会加入。

5. 拥抱函数式编程
Immutable本身就是函数式编程中的概念，纯函数式编程比面向对象更适用于前端开发。因为只要输入一致，输出必然一致，这样开发的组件更易于调试和组装。

像ClojureScript,Elm等函数式编程语言中的数据类型天生都是Immutable的，这也是为什么ClojureScript基于React的框架，OM性能比React还要好的原因。

## Immutable的缺点
容易与原生的对象混淆
这点使我们使用Immutable.js过程中遇到的最大的问题。写代码要做思维上的转变。

虽然Immutable.js尽量尝试把API设计的原生对象类似，有的时候还是很难区别到底是Immutable对象还是原生对象，容易混淆操作。

Immutable中的Map和List虽然对应原生Object和Array，但操作非常不同，比如你要用map.get('key')而不是map.key，array.get(0)而不是array[0]。另外Immutable每次修改都会返回新对象，很容易忘记赋值。

当使用外部库的时候，一般需要使用原生对象，也很容易忘记转换。

下面给出了一些办法来避免类似问题发生：
1. 使用Flow或TypeScript这类有静态类型检查的工具。
2. 约定变量命名规则：如所有Immutable类型对象以$$开头。
3. 使用Immutable.fromJS而不是Immutable.Map或Immutable.List来创建对象，这样可以避免Immutable和原生对象间的混用。

## 更多认识
两个immutable对象可以使用 === 来比较，这样是直接比较内存地址，性能最好。但即使两个对象的值是一样的，也会返回false

```javascript
    let map1 = Immutable.Map({a:1,b:1,c:1});
    let map2 = Immutable.Map({a:1,b:1,c:1});
    map1 === map2; //false
```

为了直接比较对象的值，immutable.js提供了Immutable.js来做值比较：

```javascript
    Immutable.is(map1,map2); //true
```

Immutable.is比较的是两个对象的hashCode或valueOf(对于javascript对象)。由于immutable内部使用了Tree数据结构来存储，只要两个对象的hashCode相等，值就是一样的。这样的算法避免了深度遍历比较，性能非常好。

后面会使用Immutable.js来减少React重复渲染，提高性能。

与Object.freeze、const比较
ES6中新加入的Object.freeze和const都可以达到防止对象被篡改的功能，但是它们是shallowCopy的。对象层级一深就要特殊处理了。

Cursor的概念
这个Cursor和数据库中的游标是完全不同的而概念。

由于Immutable数据一般嵌套非常深，为了便于访问深层数据，Cursor提供了直接访问这个深层数据的引用。

```javascript
    import Immutable from 'immutable';
    import Cursor from 'immutable/contrib/cursor';
    let data = Immutable.fromJS({a:{b:{c:1}}});
    //让cursor指向{c:1}
    let cursor = Cursor.from(data,['a','b'],newData => {
        //当cursor或其子cursor执行update时调用
        console.log(newData);
    });
    cursor.get('c'); //1
    cursor = cursor.update('c',x => x+1);
    cursor.get('c'); //2
```

## 实践
### 与React搭配使用，Pure Render

熟悉React的都知道，React做性能优化时有一个避免重复渲染的大招，就是使用shouldComponentUpdate(),但它默认返回true，即始终会执行render()方法，然后做Virtual DOM比较，并得到是否需要做真实DOM更新，这里往往会带来很多无必要的渲染并成为性能瓶颈。

当然我们也可以在shouldComponentUpdate()中使用deepCopy和deepCompare来避免无必要的render(),但deepCopy和deepCompare一般都是非常耗性能。

Immutable则提供了简洁高效的而判断数据是否变化的方法，只需 === 和is比较就能知道是否需要执行render(),而这个操作几乎0成本，所以可以极大提高性能。修改后的shouldComponentUpdate是这样的：

```javascript
    import {is} from 'immutable';
    shouldComponentUpdate: (nextProps,nextState) => {
        return !(this.props === nextProps || is(this.props,nextProps)) || !(this.state === nextState || is(this.state,nextState));
    }
```

当然你也可以借助React.addons.PureRenderMixin 或支持class语法的pure-render-decorator来实现。

setState的一个技巧

React建议把this.state当作Immutable的，因此修改前需要做一个deepCopy,显得麻烦：

```javascript
    import '_' from 'lodash';
    const Component = React.createClass({
        getInitialState(){
            return{
                data:{times:0}
            }
        },
        handleAdd(){
            let data = _.cloneDeep(this.state.data);
            data.times = data.times + 1;
            this.setState({data:data});
            //如果上面不做cloneDeep，下面打印的结果会是已经加1后的值。
            console.log(this.state.data.times);
        }
    })
```

使用Immutable后：

```javascript
    getInitialState(){
        return {
            data: Map({times:0})
        }
    },
    handleAdd(){
        this.setState({data:this.state.data.update('times',v => v+1)});
        //这时的times并不会改变
        console.log(this.state.data.get('times'));
    }
```

上面的handleAdd可以简写成：

```javascript
    handleAdd(){
        this.setState(({data}) => ({
            data: data.update('times', v => v+1)})
        });
    }
```

## 与Flux搭配使用
由于Flux并没有限定Store中的数据类型，使用Immutable非常简单。
下面是实现一个类似带有添加和撤销功能Store:

```javascript
    import {Map, OrderedMap} from 'immutable';
    let todos = OrderedMap();
    let history = []; //普通数组，存放每次操作后产生的数据
    let TodoStore = createStore({
        getAll(){
            return todos;
        }
    });
    Dispatcher.register(action => {
        if(action.actionType === 'create'){
            let id = createGUID();
            history.push(todos); //记录当前操作前的数据，便于撤销
            todos = todos.set(id,Map({
                id: id,
                complete: false,
                text: action.text.trim()
            }));
            TodoStore.emitChange();
        }else if(action.actionType === 'undo'){
            if(history.length > 0){
                todos = history.pop();
            }
            TodoStore.emitChange();
        }
    })
```

## 与Redux搭配使用
Redux是目前最流行的Flux衍生库。它简化了Flux中多个Store的概念，只有一个Store，数据操作通过Reducer中实现；同时它提供了更简洁和清晰的单项数据流(View -> Action -> Middleware -> Reducer),也更易于开发同构应用。

由于 Redux 中内置的 combineReducers 和 reducer 中的 initialState 都为原生的 Object 对象，所以不能和 Immutable 原生搭配使用。

幸运的是，Redux 并不排斥使用 Immutable，可以自己重写 combineReducers 或使用 redux-immutablejs 来提供支持。

上面我们提到 Cursor 可以方便检索和 update 层级比较深的数据，但因为 Redux 中已经有了 select 来做检索，Action 来更新数据，因此 Cursor 在这里就没有用武之地了。

## 总结
Immutable可以给应用带来极大的性能提升，但是否使用还要看项目情况。由于侵入性较强，新项目引入比较容易，老项目迁移需要评估迁移。对于一些提供给外部使用的公共组件，最好不要把Immutable对象直接暴露在对外接口中。
