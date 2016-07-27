---
title: 使用ServiceWorkers
date: 2016-07-27 23:02:50
tags:
- javascript
- service worker
categories: 笔记
---
> Don't let yesterday take up too much of today.

`这是一个实验中的功能，此功能某些浏览器尚在开发中，请参考[浏览器兼容性表格](https://developer.mozilla.org/zh-CN/docs/Web/API/Service_Worker_API/Using_Service_Workers#Browser_compatibility)以得到在不同浏览器中适合使用的前缀。由于该功能对应的标准文档可能被重新修订，所以在未来版本的浏览器中该功能的语法和行为可能随之改变。`

本文提供了使用service workers所需要的相关知识。包括它的基本结构、注册一个service worker、一个新的service worker的安装和激活流程、更新你的service worker、缓存管理和自定义响应内容。所有这些功能点都是基于一个场景：离线APP。

## Service workers出现的背景
有一个困扰web用户多年的难题：网络不可连接(离线)。即使是世界上最好的web app，如果你下载不了它，用户体验基本是毁的。已经有很多种技术尝试，来解决这一问题。随着离线页面的出现，一些问题已经得到了解决。但是，最重要的问题是，仍然没有一个好的统筹机制，来对缓存和网络请求进行控制。

之前的尝试--APPCache看起来是个不错的方法，因为它可以很容易地指定需要离线缓存的资源。但是，这个方法假定了你使用时会遵循很多规则，如果你不严格遵循这些规则，它会把你的APP搞的一团糟。

`注意：从FireFox44起，当使用APPCache来提供离线页面支持时，会提示一个警告消息，来建议开发者使用service workers来实现离线页面。`

Service workers应该最终解决了这些问题。Service Worker的语法比APPCache更加复杂，但带来的效果是你可以使用javascript，更加灵活和细粒度地控制你的应用的缓存资源。有了它，你可以解决目前离线应用的问题，同时也可以做更多的事。使用Service worker可以使你的应用先访问本地缓存，所以在离线状态时，在没有通过网络接收到更多数据前，仍可以提供基本的功能检验(一般称之为Offline First)。这是原生APP本来就支持的功能，这也是相比于web app，原生app更受青睐主要原因。

## 使用Service workers前的设置
在已经支持service workers的浏览器的较新版本中，很多service workers的特性默认没有开启支持。如果你发现示例代码当前版本的浏览器中怎么样都无法正常运行，你可能需要开启一下浏览器的相关配置：
 - FireFox Nightly：访问about:config 并设置dom.ServiceWorkers.enabled的值为true;重启浏览器；
 - Chrome Canary：访问chrome://flags并开启experimetal-web-platform-features;重启浏览器(注意：有些特性在Chrome中没有默认开启支持)；
 - Opera：访问opera://flags并开启ServiceWorker的支持；重启浏览器。

另外，你需要通过HTTPS来访问你的页面代码--出于安全原因，Service Workers严格要求要在HTTPS下才能运行。Github是个用来测试的好地方，因为它就支持HTTPS.

## 基本架构
使用service workers，通常遵循以下基本步骤：
 1. service worker,通过serviceWorkerContainer.register()来加载和注册(一个脚本URL)。
 2. 如果注册成功，service worker在ServiceWorkerGlobalScope环境中运行；这是一个特殊的worker上下文运行环境，与脚本的运行线程相独立，同时没有访问DOM的能力。
 3. service worker现在可以处理事件了。
 4. 受service worker控制的页面打开后，service worker尝试安装。最先发送给service worker的事件，是安装事件(install event在这个事件里，可以开始IndexDB和Cache的相关操作流程)。这个流程同原生APP或者FireFox OS APP是一样的--让所有资源可离线访问。
 5. 当oninstall事件的处理流程执行完毕后，可以认为service worker安装完成了。
 6. 下一步是激活。当service worker安装完成后，会接收到一个激活事件(activate event)。激活事件的处理函数中，主要操作是清理旧版本的service worker脚本中使用资源。
 7.Service Worker现在可以控制页面了，但是只针对在成功注册(register())了service worker后打开的页面。也就是说，页面打开是有没有service worker,决定了接下来页面的生命周期内受不受service worker控制。所以，只有当页面刷新后，之前不受service worker控制的页面才有可能被控制起来。
 ![Worker lifecycle](/images/serviceWorker/sw-lifecycle.png)

 下图表示了service worker所有支持的事件：
 ![Events](/images/serviceWorker/sw-events.png)

 ## Promises
 [Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)是一种非常适用于异步操作的机制，一个操作依赖于另一个操作的成功执行。这是service worker的核心工作机制。

 Promises可以做很多事情。但现在，你只需要知道，如果有什么返回了一个Promise，你可以在后面加上.then()来传入成功和失败的回调函数。或者，你可以在后面加上.catch()如果你想添加一个操作失败的回调函数。

 接下来，让我们对比一下传统的同步回调结构，和异步promise结构，两者在功能上是等效的：
 **同步**
 ```javascript
  try{
    var value = myFunction();
    console.log(value);
  } catch(err){
    console.log(err);
  }
 ```
**异步**
```javascript
  myFunction().then(function(value){
    console.log(value);
  }).catch(function(err){
    console.log(err);
  })
```

在上面第一个例子中，我们必须等待 myFunction( ) 执行完成，并返回 value值，在此之前，后续其它的代码无法执行。在第二个例子中，myFunction( ) 返回一个promise对象，下面的代码可以继续执行。当promise成功resolves后，then( ) 中的函数会异步地执行。

现在来举下实际的例子 — 如果我们想动态地加载图片，而且要在图片下载完成后再展示到页面上，要怎么实现呢？这是一个比较常见的场景，但是实现起来会有点麻烦。我们可以使用 .onload 事件处理程序，来实现图片的加载完成后再展示。但是如果图片的 onload事件发生在我们监听这个事件之前呢？我们可以使用 .complete来解决这个问题，但是仍然不够简洁，如果是多个图片该怎么处理呢？并且，这种方法仍然是同步的操作，会阻塞主线程。

```javascript
  function imgLoad(url){
    return new Promise(function(resolve,reject){
      var request = new XMLHttpRequest();
      request.open('Get',url);
      request.responseType = 'blob';

      request.onload = function(){
        if(request.status == 200){
          resolve(request.response);
        }else{
          reject(Error('Image didn\'t load successfully; error code:' + request.statusText));
        }
      }

      request.onerror = function(){
        reject(Error('There was a network error.'));
      }

      request.send();
    })
  }
```
我们使用 Promise( ) 构造函数返回了一个新的promise对象，构造函数接收一个回调函数作为参数。这个回调函数包含两个参数，第一个为成功执行(resolve)的回调函数，第二个为执行失败(reject)的回调函数。我们将这两个回调函数在对应的时机执行。在这个例子中，resoleve会在请求返回状态码200的时候执行，reject会在请求返回码为非200的时候执行。上面代码的其余部分基本都是XHR的相关操作，现在不需要过多关注。

当我们调用 imgLoad( ) 函数时，传入要加载的图片url作为参数。然后，后面的代码与同步方式会有点不同：
```javascript
  var body = document.querySelector('body');
  var myImage = new Image();

  imgLoad('myLittleVader.jpg').then(function(response) {
  var imageURL = window.URL.createObjectURL(response);
  myImage.src = imageURL;
  body.appendChild(myImage);
  }, function(Error) {
  console.log(Error);
  });
```
