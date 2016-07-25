---
title: 这个API很迷人
date: 2016-07-25 15:25:12
tags:
- javascript
- protocol
categories: 笔记
---
> 文章转载自[这个API很“迷人”——(新的Fetch API)](http://www.w3ctech.com/topic/854)，仅供学习和参考

## 原标题是This API is so Fetching, Fetching也可以表示迷人的意思
Javascript通过XMLHttpRequest(XHR)来执行异步请求，这个方式已经存在了很长一段时间。虽说他很有用，但他不是最佳API。他在设计上不符合职责分离原则，将输入、输出和用事件来跟踪的状态混杂在一个对象里。而且，基于事件的模型与最近Javascript流行的Promise以及基于生成器的异步编程模型不太搭。

新的[Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) API打算修正上面提到的那些缺陷。他向JS中引入和HTTP协议中同样的原语。具体而言，它引入一个使用的函数fetch()用来捕捉从网络上检索一个资源的意图。

[Fetch规范](https://fetch.spec.whatwg.org/)的API明确了用户代理获取资源的语义。它结合ServiceWorkers，尝试达到以下优化：
 1. 改善离线体验
 2. 保持可扩展性

## 特性检测
要检查是否支持Fetch API，可以通过检查Headers、Request、Response或者fetch在window或者worker作用域中是否存在。
```javascript
  if(self.fetch){
    //run my fetch request here
  }else{
    // do something with XMLHttpRequest
  }
```

### 简单的fetching示例
在Fetch API中，最常用的就是fetch()函数。它接收一个URL参数，返回一个promise来处理response。response参数带着一个Response对象。
```javascript
  fetch('./data.json').then(function(res){
    // res instanceof response == true
    if(res.ok){
      res.json().then(function(data){
        console.log(data);
      });
    }else{
      console.log('Looks like the response',res.status);
    }
  },function(e){
    console.log('Fetch failed',e);
  })
```
如果是提交一个POST请求，代码如下：
```javascript
  fetch('http://www.example.org/submit.php',{
    method: 'POST',
    headers: {
      'Content-Type': 'application/x-www-form-urlencoded'
    },
    body: 'firstName=Nikhil&favColor=blue&password=easytoguess'
  }).then(function(res){
    if(res.ok){
      alert('Perfect! Your settings are saved.');
    }else if(res.status == 401){
      alert('Oops! You are not authorized.');
    }
  },function(e){
    alert('Error submitting form!')
  })
```
fetch()函数的参数和传给Request()构造函数的参数保持完全一致，所以你可以直接传任意复杂的request请求给fetch().

### Headers
Fetch引入了3个接口，它们分别是Headers、Request以及Response。他们直接对应了相应的HTTP概念，但是基于安全考虑，有些区别，例如支持CORS规则以及保证cookies不能被第三方获取。
Headers接口是一个简单的多映射的名-值表
```javascript
  var content = 'Hello World';
  var reqHeaders = new Headers();
  reqHeaders.append('Content-Type','text/plain');
  reqHeaders.append('Content-Length',content.length.toString());
  reqHeaders.append('X-Custom-Header','ProcessThisImmediately');
```
也可以传一个多维数组或者json：
```javascript
  reqHeaders = new Headers({
    "Content-Type": "text/plain",
    "Content-Length": content.length.toString(),
    "X-Custom-Header": "ProcessThisImmediately"
  })
```
Headers的内容可以被检索：
```javascript
  console.log(reqHeaders.has('Content-Type'));
  console.log(reqHeaders.has("Set-Cookie"));
  reqHeaders.set("Content-Type", "text/html");
  reqHeaders.append("X-Custom-Header", "AnotherValue");

  console.log(reqHeaders.get("Content-Length"));
  console.log(reqHeaders.getAll("X-Custom-Header"));

  reqHeaders.delete("X-Custom-Header");
  console.log(reqHeaders.getAll("X-Custom-Header"));
```
一些操作不仅仅对ServiceWorkers有用，本身也提供了更方便的操作Headers的API。

由于Headers可以在request请求中被发送或者在response请求中被接收，并且规定了哪些参数是可写的，Headers对象有一个特殊的guard属性。这个属性没有暴露给Web，但是它影响到哪些内容可以在Headers对象中被改变。
可能的值如下：
 - "none": 默认值。
 - "request": 从Request中获得的Headers只读。
 - "request-no-cors": 从不同域的Request中获得的Headers只读。
 - "response": 从Response获得的Headers只读.
 - "immutable": 在ServiceWorkers中最常用，所有的Headers都只读。

哪一种guard作用于Headers导致什么行为，详细定义在这个[规范](https://fetch.spec.whatwg.org/)中。例如，你不可以添加或者修改一个guard属性是"request"的Request Headers的"Content-Length"属性。同样地，插入"Set-Cookie"属性到一个Response headers是不允许的，因此ServiceWorkers是不能给合成的Request的headers添加一些cookies。

如果使用了一个不合法的HTTP Header属性名，那么Headers的方法通常都抛出TypeError异常。如果不小心写入了一个不可写的属性，也会抛出一个 TypeError 异常。除此以外的情况，失败了并不抛出异常。例如：
```javascript
  var res = Response.error();
  try{
    res.headers.set('Origin', 'http://myback.com')
  }catch(e){
    console.log('Cannot pretend to be a bank!')
  }
```

### Requset
http://www.w3ctech.com/topic/854
