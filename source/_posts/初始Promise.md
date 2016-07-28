---
title: 初始Promise
date: 2016-07-28 11:21:39
tags:
- javascript
- promise
categories: 笔记
---
> Don't let yesterday take up too much of today.

## 概述
Promise对象用于异步(asynchronous)计算。一个promise对象代表着一个还未完成，但预期将来会完成的操作。

**语法**
```javascript
  new Promise(executor);
  new Promise(function(resolve,reject){ ... });
```

**参数**
executor,带有resolve、reject两个参数的函数对象。第一个参数用在处理执行成功的场景，第二个参数则用在处理执行失败的场景。一旦我们的操作完成即可调用这些函数。

## 描述
Promise对象是一个返回值的代理，这个返回值在promise对象创建时未必已知。它允许你为异步操作的成功或失败指定处理方法。这使得异步方法可以像同步方法那样返回值：异步方法会返回一个包含了原返回值的promise对象来替代原返回值。

Promise对象有以下几种状态：
 - pedding: 初始状态，既不是fulfilled也不是rejected.
 - fulfilled: 成功的操作
 - rejected: 失败的操作

pending状态的promise对象即可转换为带着一个成功值的fulfilled状态，也可变为带着一个失败信息的rejected状态。当状态发生转化时，promise.then绑定的方法(函数句柄)就会被调用。(当绑定方法时，如果promise对象已经处在fulfilled或rejected状态，那么相应的方法将会被立刻调用，所以在异步操作的完成情况和它的绑定方法之间不存在竞争条件)。

因为Promise.prototype.then和Promise.prototype.catch方法返回promises对象，所以他们它们可以被链式调用--一种被称为composition的操作。

![Promise](/images/promise/promises.png)

`注意：如果一个promise对象处在fulfilled或rejected状态而不是pedding状态，那么它也可以被称为settled转态。你可能也会听到一个术语resolved，它表示promise对象处于settled状态，或者promise对象被锁定在了调用链中。`

## 属性
Promise.length
长度属性，其值为1(构造器参数的数目)
Promise.prototype
表示Promise构造器的原型

## 方法
Promise.all(iterable)
返回一个promise对象，当iterable参数里的所有的promise都被完成后，该promise也会被完成。
Promise.race(iterable)
当iterable参数里的任意一个子promise被成功或失败后，父promise马上也会用子promise的成功返回值或失败详情作为参数调用父promise绑定的相应句柄，并返回该promise对象。
Promise.reject(reason)
调用Promise的rejected句柄，并返回这个Promise对象
Promise.resolve(value)
用成功值value完成一个Promise对象。如果该value为可继续的(thenable，即带有then方法)，返回的Promise对象会“跟随”这个value，采用这个value的最终状态；否则的话返回值会用这个value满足(fullfil)返回的Promise对象。

## Promise原型
**原型**
Promise.prototype.constructor
返回创建了实例原型的函数。默认为Promise函数。

**方法**
Promise.prototype.catch(onRejected)
添加一个否定(rejection)回调到当前promise，返回一个新的promise。如果这个回调被调用，新promise将以它的返回值来resolve，否则如果当前promise进入fulfilled状态，则以当前promise的肯定结果作为新promise的肯定结果。

Promise.prototype.then(onFulfilled.onRejected)
添加肯定和否定回调到当前promise，返回一个新的promise，将以回调的返回值来resolve.

## 示例
**创建Promise**
这个小例子展示了Promise的机制。每当<button>被按下时，testPromise()函数就会被执行。该函数会创建一个用window.setTimeout在1秒到3秒(随机)后用‘result’字符串完成的promise。
这里通过p1.then方法满足回调，简单的输出了promise的满足过程，这些输出显示了该方法的同步部分是如何和promise的异步完成解耦的。
```javascript
  <div id="log"></div>
  <script>
    'use strict'
    var promiseCount = 0;
    function testPromise(){
      var thisPromiseCount = ++promiseCount;
      var log = document.getElementById('log');
      log.insertAdjacentHTML('beforeend',thisPromiseCount + ') 开始(同步代码开始)<br/>');

      //我们创建一个新的promise:然后用'result'字符串完成这个promise(3秒后)
      var p1 = new Promise(function(resolve,reject){
        //完成函数带着完成(resolve)或拒绝(reject) promise的能力被执行
        log.insertAdjacentHTML('beforeend',thisPromiseCount + ') Promise开始(异步代码开始)<br/>');

        //这只是个创建异步完成的示例
        window.setTimeout(function(){
          //我们满足(fullfil)了这个promise！
          resolve(thisPromiseCount)
        },Math.random() *2000 + 1000);
      });

      //定义当promise被满足时应做什么
      p1.then(function(val){
        //输出一段信息和一个值
        log.insertAdjacentHTML('beforeend',val + ') Promise被满足了(异步代码结束)<br/>');
      });

      log.insertAdjacentHTML('beforeend',thisPromiseCount + ') 建立了Promise(同步代码结束)<br/><br/>');
    }
    testPromise();
    testPromise();
    testPromise();
  </script>
```
这个例子在按钮被按下后执行。你需要一个支持Promise的浏览器。你能通过短时间内按多次按钮看到不同的promise一个接一个的被满足。

## Example using new XMLHttpRequest()
**创建一个promise**
这个例子展示了如何用promise报告一个XMLHttpRequest的成功或失败。
```javascript
  'use strict'
  function $http(url){
    var core = {
      ajax: function(method,url,args){
        var promise = new Promise(function(resolve,reject){
          var client = new XMLHttpRequest();
          var uri = url;

          if(args && (method === 'POST' || method === 'PUT')){
            uri += '?';
            var argcount = 0;
            for(var key in args){
              if(args.hasOwnProperty(key)){
                if(argcount++){
                  uri += '&';
                }
                uri += encodeURIcomponent(key) + '=' + encodeURIcomponent(args[key]);
              }
            }
          }

          client.open(method, uri);
          client.send();

          client.onload = function(){
            if(this.status >= 200 && this.status <300){
              resolve(this.response);
            }else{
              reject(this.statusText);
            }
          };
          client.onerror = function(){
            reject(this.statusText);
          }
        });

        return promise;
      }
    };

    //Adapter patten
    return {
      'get': function(args){
        return core.ajax('GET',url,args);
      },
      'post': function(args){
        return core.ajax('POST',url,args);
      },
      'delete': function(args){
        return core.ajax('DELETE',url,args);
      }
    }
  }
  //End A

  var mdnAPI = 'https://developer.mozilla.org/en-US/search.json';
  var payload = {
    'topic': 'js',
    'q': 'Promise'
  };

  var callback = {
    success: function(data){
      console.log(1,'success',JSON.parse(data));
    },
    error: function(data){
      console.log(2,'error',JSON.parse(data));
    }
  }
  //End B

  //Executes the method call
  $http(mdnAPI)
    .get(payload)
    .then(callback.success)
    .catch(callback.error);

  $http(mdnAPI)
    .get(payload)
    .then(callback.success,callback.error);

  $http(mdnAPI)
    .get(payload)
    .then(callback.success)
    .then(undefined, callback.error);
```
