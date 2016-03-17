title: node-10http
date: 2015-11-10 13:45:47
tags:
- node
- http
- 学习笔记
categories: 笔记
---

## http-server
```
var http = require('http');
var url = require('url');// ctrl+ alt +下箭头
var util = require('util');
var querystring = require('querystring');
var server = http.createServer(function(req,res){
    console.log(req.method);//获取请求的方法
    console.log(req.headers);//获取请求头
    console.log(req.url);//获取请求的url
    //http://zfpx:12345@localhost:8080/index/index.html?name=zfpx#top
    var urlObj = url.parse("http://zfpx:12345@localhost:8080"+req.url,true);//url转成对象
    var result = '';
    req.on('data',function(data){
        // console.log(data);
        result+=data;
    });
    req.on('end',function(){
        var contentType = req.headers['content-type'];//全部小写
        if(contentType == 'json'){
            var resObj = JSON.parse(result);
        }else{
            var resObj = querystring.parse(result);
        }
        res.statusCode = 200;//响应的状态码
        res.setHeader('name','zfpx');
        res.writeHead(200,{'name':'zfpx'});
        res.end(JSON.stringify(resObj));
    });

    //res.end(util.inspect(urlObj.query));
    //res.end('end');
});
server.listen(8088);
server.on('connection',function(){
    console.log('connection');
});
server.on('error',function(err){
    console.log('error');
});
server.on('close',function(){
    console.log('close');
});
server.on('error',function(){
    console.log('close');
});
server.setTimeout(3000,function(){
    console.log('timeout');
});
```

## http-client
```
var http = require('http');
//http://zfpx:12345@localhost:8080/index/index.html?name=zfpx#top
var options = {
    method:'post',
    protocol: 'http:',
    slashes: true,
    auth: 'zfpx:12345',
    host: 'localhost:8088',
    port: '8088',
    hostname: 'localhost',
    hash: null,
    search: '?name=zfpx',
    query: { name: 'zfpx' },
    pathname: '/index/index.html',
    path: '/index/index.html?name=zfpx',
    headers:{'connection':'close','Content-Type':'application/x-www-form-urlencoded'},//当提交表单的时候会把表单序列化成这种类型 url query查询字符是一样的
    headers:{'Content-Type':'application/json'},
    headers:{'Content-Type':'application/xml'},
    href: 'http://zfpx:12345@localhost:8080/index/index.html?name=zfpx' }
//res 是一个流对象 可读可写的流
/**
 * 可读流
 *  data end事件
 * 可写流
 *  write end方法
 */
var request = http.request(options,function(res){
    console.log(res.statusCode);
    console.log(res.headers);
    //读取可读流里的数据 readstream readable
    res.setEncoding('utf8');
    var result = '';
    res.on('data',function(data){
        // console.log(data);
        result+=data;
    });
    res.on('end',function(){
        console.log(result);
    });
})
/*request.on('error',function(err){
 console.log(err);
 });*/
request.write('age=7&name=zf');//向请求体里写数据
//request.write(JSON.stringify({"name":"zfpx"}));//向请求体里写数据
request.end();//真正发起向服务器的请求
```

## 注册登录并支持上传图片
```
var http = require('http');
var url = require('url');
var querystring = require('querystring');
var fs = require('fs');
var formidable = require('formidable');
var mime = require('mime');
var useList = [];

http.createServer(function(req,res){
    var pathname = req.url;
    if(pathname == '/favicon.ico'){
        res.end('404');
    }else if(pathname == '/'){
        fs.createReadStream('./reg.html').pipe(res);
    }else if(pathname == '/reg'){
        var parser = new formidable.IncomingForm();
        parser.parse(req,function(err,fields,files){
            var userObj = {uname:fields.uname,upasswd:fields.upasswd};
            useList.push(userObj);
            fs.createReadStream(files.avatar.path).pipe(fs.createWriteStream('./upload/'+files.avatar.name));
            //console.log(fields);
            //console.log(files);
            res.writeHead(200,{'Content-Type':'text/html;charset=utf-8'});
            res.write(userObj.uname);
            res.write(userObj.upasswd);
            res.end('<img src="/upload/'+files.avatar.name+'" />')
        })
    }else if(pathname == '/login'){
        var result = '';
        req.on('data',function(data){
            result+=data;
        });
        req.on('end',function(){
            var userObj = querystring.parse(result);
            for(var i=0;i<useList.length;i++){
                if(useList[i].uname == userObj.uname&&useList[i].upasswd == userObj.upasswd){
                    res.setHeader('Content-Type','text/html;charset=utf-8');
                    res.end('登录成功');
                    return;
                }
            }
            fs.createReadStream('./reg.html').pipe(res);
        })
    }else if(pathname.indexOf('/upload/')!=-1){
        res.writeHead(200,{'Content-Type':mime.lookup(pathname)})
        fs.createReadStream('.'+pathname).pipe(res);
    }
}).listen(8080);
```