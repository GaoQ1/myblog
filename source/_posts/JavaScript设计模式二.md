---
title: JavaScript设计模式二
date: 2016-08-18 14:06:04
tags:
- javascript
- 设计模式
categories: 笔记
---
> Comedy is acting out optimism.

## 中介者模式
字典中中介者的定义是，一个中立方，在谈判和冲突解决过程中起辅助作用。在我们的世界，一个中介者是一个行为设计模式，使我们可以导出统一的接口，这样系统不同部分就可以彼此通信。

如果系统组件之间存在大量的直接关系，就可能是时候，使用一个中心的控制点，来让不同的组件通过它来通信。中介者通过将组件之间显示的直接的引用替换成通过中心点来交互的方式，做到松耦合。这样可以帮助我们解耦和改善组件的重用性。

在现实世界中，类似的系统就是，飞行控制系统。一个航站塔(中介者)处理哪个飞机可以起飞，哪个可以着陆，因为所有的通信(监听的通知或者广播的通知)都是飞机和控制塔之间进行的，而不是飞机和飞机之间进行的额。一个中央集权的控制中心是这个系统成功的关键，也正是中介者在软件设计领域中所扮演的角色。

从实现角度来讲，中介者模式是观察者模式中的共享被观察者对象。在这个系统中的对象之间直接的发布/订阅关系被牺牲掉了，取而代之的是维护一个通信的中心节点。

也可以认为是一种补充-用于应用级别的通知，例如不同子系统之间的通信，子系统本身很复杂，可能需要使用发布/订阅模式来做内部之间的解耦。

另外一个类似的例子是DOM的事件冒泡机制，以及事件代理机制。如果系统中所有的订阅者都是对文档订阅，而不是对独立的节点订阅，那么文档就充当一个中介者的角色。DOM的这种做法，不是将事件绑定到独立节点上，而是用一个更高级别的对象负责通知订阅者关于交互事件的信息。

### 基础的实现
中介者模式的一种简单的实现可以在下面找到，publish()和subscribe()犯法都被暴露出来使用：
```javascript
var mediator = (function(){

  // Storage for topics that can be broadcast or listened to
  var topics = {};

  // Subscribe to a topic, supply a callback to be executed
  // when that topic is broadcast to
  var subscribe = function( topic, fn ){

      if ( !topics[topic] ){
        topics[topic] = [];
      }

      topics[topic].push( { context: this, callback: fn } );

      return this;
  };

  // Publish/broadcast an event to the rest of the application
  var publish = function( topic ){

      var args;

      if ( !topics[topic] ){
        return false;
      }

      args = Array.prototype.slice.call( arguments, 1 );
      for ( var i = 0, l = topics[topic].length; i < l; i++ ) {

          var subscription = topics[topic][i];
          subscription.callback.apply( subscription.context, args );
      }
      return this;
  };

  return {
      publish: publish,
      subscribe: subscribe,
      installTo: function( obj ){
          obj.subscribe = subscribe;
          obj.publish = publish;
      }
  };
}());
```

### 高级的实现
更高级的实现方式，首先，让我们实现认购的概念，我们可以考虑一个中介者主题的注册。
通过生成对象实体，我们稍后能够简单的更新认购，而不需要去取消注册然后重新注册它们。认购可以写成一个使用被称作一个选项对象或者一个上下文环境的函数。
```javascript
  //Pass in a context to attach our Mediator to.
  //By default this will be the window object
  (function(root){
    function guidGenerator(){
      //Our Subscriber constructor
      function Subscriber(fn, options, context){
        if(!(this instanceof Subscriber)){
          return new Subscriber(fn, context, options);
        }else{

          //guidGenerator() is a function that generates
          //GUIDs for instances of our Mediators Subscribers so we can easily reference them later on. We're going to skip its implementation for brevity

          this.id = guidGenerator();
          this.fn = fn;
          this.options = options;
          this.context = context;
          this.topic = null;
        }
      }
    }
  })();
```
在我们的中介者模式中包含了一长串的回调和子主题，当中间人发布在我们中间人实体上被调用的时候被启动。它也包含操作数据列表方法。
```javascript
  //Let's model the topic
  //JavaScript lets us use a Function object as a conjunction of a prototype for use with the new object and a constructor function to be invoked.
  function Topic(namespace){
    if(!(this instanceof Topic)){
      return new Topic(namespace);
    }else{
      this.namespace = namespace || "";
      this._callbacks = [];
      this._topics = [];
      this.stopped = false;
    }
  }

  //Define the prototype for our topic, including ways to add new subscribers or retrieve existing ones.
  Topic.prototype = {
    //Add a new subscriber
    AddSubscriber: function(fn,options,context){
      var callback = new Subscriber(fn, options, context);
      this._callbacks.push(callback);
      callback.topic = this;
      return callback;
    }
  }
```

我们的主题实体被当做中间人调用的一个参数被传递。使用一个方便实用的calledStopPropagation()方法，回调就可以进一步被传播开来：
```javascript
  StopPropagation: function(){
    this.stopped: true;
  }
```

我们也能够使得当提供一个GUID的标识符的时候检索订购用户更加容易:
```javascript
  GetSubscriber: function(identifier){
    for(var x=0,y=this._callbacks.length; x<y ; x++){
      if(this._callbacks[x].id == identifier || this._callbacks[x].fn == identifier){
        return this._callbacks[x];
      }
    }

    for(var z in this._topics){
      if(this._topics.hasOwnProperty(z)){
        var sub = this._topics[z].GetSubscriber(identifier);
        if(sub !== undefined)(
          return sub;
        )
      }
    }
  }
```

接着，在我们需要它们的情况下，我们也能够提供添加新主题，检查现有的主题或者检索主题的简单方法：
```javascript
  AddTopic: function( topic ){
    this._topics[topic] = new Topic( (this.namespace ? this.namespace + ":" : "") + topic );
  },

  HasTopic: function( topic ){
    return this._topics.hasOwnProperty( topic );
  },

  ReturnTopic: function( topic ){
    return this._topics[topic];
  }
```

如果我们觉得不再需要它们了,我们也可以明确的删除这些订购用户.下面就是通过它的其子主题递归删除订购用户的代码:
```javascript
  RemoveSubscriber: function( identifier ){
    if( !identifier ){
      this._callbacks = [];

      for( var z in this._topics ){
        if( this._topics.hasOwnProperty(z) ){
          this._topics[z].RemoveSubscriber( identifier );
        }
      }
    }

    for( var y = 0, x = this._callbacks.length; y < x; y++ ) {
      if( this._callbacks[y].fn == identifier || this._callbacks[y].id == identifier ){
        this._callbacks[y].topic = null;
        this._callbacks.splice( y,1 );
        x--; y--;
      }
    }
  }
```

接着我们通过递归子主题将发布任意参数的能够包含到订购服务对象中:
```javascript
  Publish: function( data ){

      for( var y = 0, x = this._callbacks.length; y < x; y++ ) {

          var callback = this._callbacks[y], l;
            callback.fn.apply( callback.context, data );

        l = this._callbacks.length;

        if( l < x ){
          y--;
          x = l;
        }
      }

      for( var x in this._topics ){
        if( !this.stopped ){
          if( this._topics.hasOwnProperty( x ) ){
            this._topics[x].Publish( data );
          }
        }
      }

      this.stopped = false;
    }
  };
```

接着我们暴露我们将主要交互的调节实体.这里它是通过注册的并且从主题中删除的事件来实现的
```javascript
  function Mediator() {

    if ( !(this instanceof Mediator) ) {
      return new Mediator();
    }else{
      this._topics = new Topic( "" );
    }

  };
```

想要更多先进的用例,我们可以看看调解支持的主题命名空间,下面这样的asinbox:messages:new:read.GetTopic 返回基于一个命名空间的主题实体。
```javascript
  Mediator.prototype = {

    GetTopic: function( namespace ){
      var topic = this._topics,
          namespaceHierarchy = namespace.split( ":" );

      if( namespace === "" ){
        return topic;
      }

      if( namespaceHierarchy.length > 0 ){
        for( var i = 0, j = namespaceHierarchy.length; i < j; i++ ){

          if( !topic.HasTopic( namespaceHierarchy[i]) ){
            topic.AddTopic( namespaceHierarchy[i] );
          }

          topic = topic.ReturnTopic( namespaceHierarchy[i] );
        }
      }

      return topic;
    },  
```

这一节我们定义了一个Mediator.Subscribe方法，它接受一个主题命名空间,一个将要被执行的函数,选项和又一个在订阅中调用函数的上下文环境.这样就创建了一个主题,如果这样的一个主题存在的话
```javascript
  Subscribe: function( topiclName, fn, options, context ){
    var options = options || {},
        context = context || {},
        topic = this.GetTopic( topicName ),
        sub = topic.AddSubscriber( fn, options, context );

    return sub;
  },
```

根据这一点,我们可以进一步定义能够访问特定订阅用户,或者将他们从主题中递归删除的工具
```javascript
  // Returns a subscriber for a given subscriber id / named function and topic namespace

  GetSubscriber: function( identifier, topic ){
    return this.GetTopic( topic || "" ).GetSubscriber( identifier );
  },

  // Remove a subscriber from a given topic namespace recursively based on
  // a provided subscriber id or named function.

  Remove: function( topicName, identifier ){
    this.GetTopic( topicName ).RemoveSubscriber( identifier );
  },
```

我们主要的发布方式可以让我们随意发布数据到选定的主题命名空间，这可以在下面的代码中看到。

主题可以被向下递归.例如,一条对inbox:message的post将发送到inbox:message:new和inbox:message:new:read.它将像接下来这样被使用:Mediator.Publish( "inbox:messages:new", [args] );
```javascript
  Publish: function( topicName ){
      var args = Array.prototype.slice.call( arguments, 1),
          topic = this.GetTopic( topicName );

      args.push( topic );

      this.GetTopic( topicName ).Publish( args );
    }
  };
```

最后，我们可以很容易的暴露我们的中间人，将它附着在传递到根中的对象上：
```javascript
  root.Mediator = Mediator;
    Mediator.Topic = Topic;
    Mediator.Subscriber = Subscriber;

    // Remember we can pass anything in here. I've passed inwindowto
    // attach the Mediator to, but we can just as easily attach it to another
    // object if desired.
  })( window );
```

### 优点&缺点
中间人模式最大的好处就是，它节约了对象或者组件之间的通信信道，这些对象或者组件存在于从多对多到多对一的系统之中。由于解耦合水平的因素，添加新的发布或者订阅者是相对容易的。

也许使用这个模式最大的缺点是它可以引入一个单点故障。在模块之间放置一个中间人也可能会造成性能损失，因为它们经常是间接地的进行通信的。由于松耦合的特性，仅仅盯着广播很难去确认系统是如何做出反应的。

这就是说，提醒我们自己解耦合的系统拥有许多其它的好处，是很有用的——如果我们的模块互相之间直接的进行通信，对于模块的改变（例如：另一个模块抛出了异常）可以很容易的对我们系统的其它部分产生多米诺连锁效应。这个问题在解耦合的系统中很少需要被考虑到。

在一天结束的时候，紧耦合会导致各种头痛，这仅仅只是另外一种可选的解决方案，但是如果得到正确实现的话也能够工作得很好。

### 中间人Vs观察者
中间人模式最大的好处就是，它节约了对象或者组件之间的通信信道，这些对象或者组件存在于从多对多到多对一的系统之中。由于解耦合水平的因素，添加新的发布或者订阅者是相对容易的。

也许使用这个模式最大的缺点是它可以引入一个单点故障。在模块之间放置一个中间人也可能会造成性能损失，因为它们经常是间接地的进行通信的。由于松耦合的特性，仅仅盯着广播很难去确认系统是如何做出反应的。

这就是说，提醒我们自己解耦合的系统拥有许多其它的好处，是很有用的——如果我们的模块互相之间直接的进行通信，对于模块的改变（例如：另一个模块抛出了异常）可以很容易的对我们系统的其它部分产生多米诺连锁效应。这个问题在解耦合的系统中很少需要被考虑到。

在一天结束的时候，紧耦合会导致各种头痛，这仅仅只是另外一种可选的解决方案，但是如果得到正确实现的话也能够工作得很好。

### 中间人Vs门面
不久我们的描述就将涵盖门面模式，但作为参考之用，一些开发者也想知道中间人和门面模式之间有哪些相似之处。它们都对模块的功能进行抽象，但有一些细微的差别。

中间人模式让模块之间集中进行通信，它会被这些模块明确的引用。门面模式却只是为模块或者系统定义一个更加简单的接口，但不添加任何额外的功能。系统中其他的模块并不直接意识到门面的概念，而可以被认为是单向的。
