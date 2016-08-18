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

































---------------
