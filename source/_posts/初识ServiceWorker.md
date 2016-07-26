---
title: 初识ServiceWorker
date: 2016-07-26 23:50:01
tags:
- javascript
- service worker
categories: 笔记
---
> The best preparation for tomorrow is doing your best today.

`Service worker本质上作为一个代理坐落于web应用和浏览器与网络(可用时)之间。他们旨在(包括其他事情)建立高效的离线体验，拦截网络请求，并且会根据当前的网络是否可用、服务器的内容是否有所更新来采取合适的策略。他们还允许通知推送和后台同步API.`

## Service worker的概念和用法
serviced worker是一个被注册在一个源和路径下的事件驱动的worker。它允许javascript文件控制与其相关的页面或者网站，拦截或者修改导航和资源请求，并且运用十分细化的方式来缓解资源。这让你可以完全控制自己的网页APP无论是在什么情况下(最明显的情况就是当前网络不可用时)。

service worker运行在一个worker的环境中：因此，他不会用dom的访问权，并且是在主线程之外的另一个线程中运行来加速你的APP，所以它不会造成阻塞。它的设计是完全异步的，因此，同步的XHR请求和localStorage都不能应用在service worker中。

为了安全起见，service workers只通过HTTPS运行。在被篡改的网络连接中被第三者攻击可就不好了。

`注意：Service Workers之所以优于以前如AppCache的同类尝试，是因为他不会对你想要做的事情做一些假设并且在那些假设不完全准确的时候失去作用。你可以更细致地去控制每一件事情。`

`注意：service workers大量使用了promise，因此通常他们会等待响应后才继续下一步。在此之后，他们会返回一个成功或者失败的响应。promises架构十分适合这种场景。`

## 注册
使用`ServiceWorkerContainer.register()`方法来进行service worker的第一次注册。一旦注册成功，你的service worker就会被下载到客户端并且试着安装/激活，用于整个域内用户可以访问的urls，或者在你规定的子集内。

## 下载、安装和激活
此时，你的service worker会遵守以下的生命周期。
 1. 下载
 2. 安装
 3. 激活

当用户首次访问被service worker控制的网站/页面时，service worker会立刻被下载。

之后它会至少每24小时被下载一次。它**可能**被更频繁得下载，但它肯定会每24小时被下载一次，以免不良脚本长时间生效。

当下载的文件被发现是新的，就会尝试进行安装--无论是与现存的service worker不同(字节方式的对比)，或者是第一次有service worker到达这个页面/网站。

如果这是第一次service worker可以被使用，页面首先会尝试安装，安装成功后它会被激活。

如果有现存的可用的service worker，新版本会被安装在后台，但是并不会被激活--此时我们称这个worker处在等待状态。知道再也没有已加载的页面使用旧的service worker的时候，它才会被激活。一旦没有了依赖旧service worker得已加载页面，新的service worker就会被激活(成为活动的worker)。

你可以监听InstallEvent,有一个标准动作是让你的service worker在触发时为使用做好准备，例如利用内置的storage API来创建缓存，并且放置你离线的时候期望你APP运行的内容。

同样的，会有一个activate事件。这个事件触发的时候，是一个很好的时间点去清理旧缓存和其他与旧service worker相关的东西.

service worker可以通过FetchEvent事件去响应请求。通过使用FecthEvent.respondWith方法，你可以任意修改对于这些请求的响应。

你的service worker可以用FetchEvent事件响应请求。你可以使用FetchEvent.respondWith任意修改对这些请求的响应。

`注意：因为oninstall/onactivate可能需要一些时间去完成，service worker标准提供了一个waitUtil方法，当oninstall或者onactivate时被调用，接受一个promise.在这个promise被成功地resolve前，这个service worker的functional events并不会被触发。`

## 其他使用场景
Service workers也可以被用来做这些事情：
 - 后台数据同步
 - 响应其他源的资源请求
 - 集中获取不易计算的数据的更新，比如地理位置和陀螺仪信息，这样多个页面就可以利用同一组数据了
 - 为了开发目的，在客户端进行coffeeScript、less、CJS/AMD模块等的编译和依赖管理
 - 为监控后台服务提供钩子
 - 针对特定URL的个性化模板
 - 增强性能，比如预取用户将来可能会需要的资源，比如一个相册中的几张新图片

在将来，service workers能够用来做更多使web平台接近原生应用的事。有趣的是，其他一些标准也能并且即将使用service worker作为环境，比如：
 - 后台同步：启动一个service worker即使没有用户访问特定站点，这样就可以更新缓存。
 - 对推送消息作出响应：启动一个service worker来向用户发送一条消息告诉他们有新的可用内容
 - 对时间或日期作出响应
