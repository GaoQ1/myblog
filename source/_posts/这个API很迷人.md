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
Request接口定义了通过HTTP请求资源的request格式。参数需要URL、method和headers,同时Requset也接受一个特定的body、mode、credentials以及cache hints.
最简单的Request当然是一个URL，可以通过URL来GET一个资源。
```javascript
  var req = new Request('./index.html');
  console.log(req.method); //"GET"
  console.log(req.url);
```

你也可以将一个建好的Request对象传给构造函数，这样将复制出一个新的Request。
```javascript
  var copy = new Request(req);
  console.log(copy.method);
  console.log(copy.url);
```

这种用法通常见于ServiceWorkers.
URL以外的其他属性的初始值能够通过第二个参数传给Request构造函数。这个参数是一个json：
```javascript
  var uploadReq = new Request('/uploadImage',{
    method: 'POST',
    headers: {
      'Content-Type': 'image/png',
    },
    body: 'image data'
  })
```
mode属性用来决定是否允许跨域请求，以及哪些response属性可读。可选的mode属性值为`same-origin`,`no-cors`以及`cors`。

`same-origin`模式很简单，如果一个请求是跨域的，那么返回一个简单的error，这样确保所有的请求遵守同源策略。
```javascript
  var arbitraryUrl = document.getElementById('url-input').value;
  fetch(arbitraryUrl, {mode:'same-origin'}).then(function(res){
    console.log('Response succeeded ', res.ok);
  },function(e){
    console.log('Please enter a same-orgin URL!');
  })
```

`no-cors`模式允许来自CDN的脚本、其他域的图片和其他一些跨域的资源，但是首先有个前提条件，就是请求的method只能是'HEAD'、'GET'或者'POST'。此外，任何ServiceWorkers拦截了这些请求，它不能随意添加或者改写任何headers，除了[这些](https://fetch.spec.whatwg.org/#simple-header)。第三，javascript不能访问Response中的任何属性，这保证了ServiceWorkers不会导致任何跨域下的安全问题而隐私信息泄露。

`cros`模式我们通常用作跨域请求来从第三方提供的API获取数据。这个模式遵守[CORS协议](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS).只有有限的[一些](https://fetch.spec.whatwg.org/#concept-filtered-response-cors)headers被暴露给Response对象，但是body是可读的。例如，你可以获得一个Flickr的最感兴趣的照片的清单:
```javascript
  var u = new URLSearchParams();
  u.append('method', 'flickr.interestingness.getList');
  u.append('api_key', '<insert api key here>');
  u.append('format', 'json');
  u.append('nojsoncallback', '1');

  var apiCall = fetch('https://api.flickr.com/services/rest?' + u);

  apiCall.then(function(response) {
    return response.json().then(function(json) {
      // photo is a list of photos.
      return json.photos.photo;
    });
  }).then(function(photos) {
    photos.forEach(function(photo) {
      console.log(photo.title);
    });
  });
```
你无法从Headers中读取'Date'属性，因为Flickr在Access-Control-Expose-Headers中设置了不允许读取它。
```javascript
  response.headers.get('Date'); //null
```

`credentials`枚举属性决定了cookies是否能跨域得到。这个属性与XHR的withCredentials标志相同，但是只有三个值，分别是'omit(默认)'、'same-origin'以及'include'。
Request对象也可以提供caching hints给用户代理。这个属性还在安全复审阶段。Firefox提供了这个属性，但是它目前还不起作用。
Request还有两个只读的属性与ServiceWorkers拦截有关。其中一个是referrer，表示Request的来源，可能为空。另外一个是context，是一个非常大的枚举集合定义了获得的资源的种类，它可能是image比如请求来自于img标签，可能是worker如果是一个worker脚本，等等。如果使用fetch()函数，这个值是fetch。

### Response
Response实例通常在fetch()的回调中获得。但是它们也可以用JS构造，不过通常这招只用于ServiceWorkers。
Response中最常见的成员是status(一个整数默认值是200)和status Text(默认值是'OK')，对应HTTP请求的status和reason。还有一个'ok'属性，当status为2xx的时候它是true.
headers属性是Response的Headers对象，它是只读的，url属性是当前Response的来源URL.
Response也有一个type属性，它的值可能是'basic'、'cors'、'default'、'error'或者'opaque'。
 - 'basic': 正常的，同域的请求，包含所有的headers除开'Set-Cookie'和'Set-Cookie2'.
 - 'cros': Response从一个合法的跨域请求获得，[一部分headers和body](https://fetch.spec.whatwg.org/#concept-filtered-response-cors)可读.
 - 'error': 网络错误。Response的status是0，Headers是空的并且不可写。当Response是从Response.error()中得到时，就是这种类型。
 - 'opaque': Response从'no-cros'请求了跨域资源。依靠Server端来做限制。

'error'类型会导致fetch()函数的Promise被reject并回调一个TypeError.
还有一些属性只在ServiceWorkers作用域下有效。以正确的方式返回一个Response针对一个被ServiceWorkers拦截的Request，可以像下面这样写：
```javascript
  addEventListener('fetch', function(event){
    event.responseWith(new Response('Response body', {
      headers: { 'Content-Type': 'text/plain' }
    }))
  })
```
如你所见，Response有个接收两个可选参数的构造器。第一个参数返回的body，第二个参数是一个json,设置status、statusText以及headers.
静态方法Response.error()简单返回一个错误的请求。类似的，Response.redirect(url,status)返回一个跳转URL的请求。

### 处理body
无论Request还是Response都可能带着body。由于body可以是各种类型。body可以是以下任何一种类型的示例:
 - [ArrayBuffer](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer)
 - [ArrayBufferView](https://developer.mozilla.org/en-US/docs/Web/API/ArrayBufferView)(Unit8Array and friends)
 - [Blob](https://developer.mozilla.org/en-US/docs/Web/API/Blob)/[File](https://developer.mozilla.org/en-US/docs/Web/API/File)
 - 字符串
 - [URLSearchParams](https://url.spec.whatwg.org/#interface-urlsearchparams)
 - [FormData](https://developer.mozilla.org/en-US/docs/Web/API/FormData)

此外，Request和Response都为他们的body提供了以下方法，这些方法都返回一个Promise对象。
 - arrayBuffer()
 - blof()
 - json()
 - text()
 - formData()
在使用非文本的数据方面，Fetch API和XHR相比提供了极大的便利。

可以通过传body参数来设置Request的body：
```javascript
  var form = new FormData(document.getElementById('login-form'));
  fetch('/login', {
    method: 'POST',
    body: form
  })
```
Response的第一个参数是body:
```javascript
  var res = new Response(new File(['chunk', 'chunk'], 'archive.zip',{ type: 'application/zip' }));
```
Request和Response(通过fetch()方法)都能够[自动识别自己的content type](https://fetch.spec.whatwg.org/#concept-bodyinit-extract)，Request还可以自动设置'Content-Type'header,如果开发者没有设置它的话。

### 流和克隆
非常重要的一点要说明，那就是Request和Response的body只能被读取一次！它们有一个属性叫bodyUsed,读取一次之后设置为true，就不能再读取了。
```javascript
  var res = new Response('one time use');
  console.log(res.bodyUsed); //false
  res.text().then(function(){
    console.log(res.bodyUsed); //true
  });
  console.log(res.bodyUsed); //true

  res.text().catch(function(e){
    console.log('Tried to read already consumed Response')
  })
```

这样设计的目的是为了之后兼容基于流的API，让应用一次消费data，这样就允许了JavaScript处理大文件例如视频，并且可以支持实时压缩和编辑。
有时候，我们希望多次访问body，例如，你可能想用即将支持的[Cache API](http://slightlyoff.github.io/ServiceWorker/spec/service_worker/index.html#cache-objects)去缓存Request和Response,以便于可以离线使用，Cache要求body能被再次读取。

所以，我们该如何让body能经得起多次读取呢？API提供了一个clone()方法。调用这个方法可以得到一个克隆对象。不过要记得，**clone()必须要在读取之前调用，也就是先clone()在读取**。
```javascript
  addEventListener('fetch', function(evt){
    var sheep = new Response('Dolly');
    console.log(sheep.bodyUsed); //false
    var clone = sheep.clone();
    console.log(clone.bodyUsed); //false

    clone.text();
    console.log(sheep.bodyUsed); //false
    console.log(clone.bodyUsed); //true

    evt.responseWith(cache.add(sheep.clone()).then(fucntion(e){
      return sheep;
    }))
  })
```

### 未来的改进
为了支持流，Fetch最终将提供可以中断执行读取资源的能力，并且提供可以得到读取进度的API。这些能力在XHR中有，但是想要实现成Promise-based的Fetch API有些麻烦。
