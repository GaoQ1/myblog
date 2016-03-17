title: node_01http
date: 2015-10-24 20:53:18
tags:
- node
- http
- 学习笔记
categories: 笔记
---

这是**node第一天笔记**，关于客户端和服务器通信也就是http协议。


## 开门见山，先看如何实现node的请求
```
var http = require('http');
http.createServer(function(req,res){//req->请求 res->响应
    req.end('hello world');//向向应里写内容用end或是write方法
}).listen(8080);//监听8080端口
```

## 再看一个node如何获取页面里面的内容
### /index.html页面
```
<!DOCTYPE html>
<html>
<head lang="en">
  <meta charset="UTF-8">
  <title></title>
</head>
<body>
<%=type%>
</body>
</html>
```
### /server.js
```
var http = require('http');
var fs = require('fs');
http.createServer(function(req,res){
    var url = req.url;
    var urls = url.split('?');
    var pathname = urls[0];
    var query = urls[1];
    if(pathname == '/index.html'){
        var content = fs.readFileSync('/index.html');
        res.end(content);    
    }else{
        res.end('404');
    }
}).listen(8080);
```

## 当index.html页面中多了css文件和img文件，看node如何渲染
### /index.html
```
<!DOCTYPE html>
<html>
<head lang="en">
  <meta charset="UTF-8">
  <title></title>
  <link rel="stylesheet" href="style.css"/>
</head>
<body>
<%=type%>
<img src="cy.png" alt=""/>
</body>
</html>
```
### /server.js
```
var http = require('http');
var fs = require('fs');
http.createServer(function(req,res){
    var url = req.url;
    var urls = url.split('?');
    var pathname = urls[0];
    var query = urls[1];
    if(pathname == '/index.html'){
        var content = fs.readFileSync('./index.html');
        res.end(content.toString().replace("<%=type%>",query.split("=")[1]));
    }else if(pathname == '/style.css'){
        res.setHeader('Content-Type','text/css');
        var content = fs.readFileSync('./style.css');
        res.end(content);
    }else if(pathname == '/cy.png'){
        res.setHeader('Content-Type','image/png');
        var content = fs.readFileSync('./cy.png');
        res.end(content);
    }else{
        res.end('404');
    }
}).listen(8080);
```
### 简化后的/server-update.js
```
var http = require('http');
var fs = require('fs');
var mime = require('mime');
http.createServer(function(req,res){
    var url = req.url;
    var urls = url.split('?');
    var pathname = urls[0];
    var query = urls[1];
    var flag = fs.existsSync('.'+pathname);
    if(flag){
        var content = fs.readFileSync('.'+pathname);
        res.setHeader('Content-Type',mime.lookup(pathname));
        res.end(content);
    }else{
        res.end('404');
    }
}).listen(8080);
```

## end
