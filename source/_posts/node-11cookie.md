title: node-11cookie
date: 2015-11-10 18:44:37
tags:
- node
- cookie
- session
- 学习笔记
categories: 笔记
---

## cookie
请求路径 查询字符串 请求体
很多业务是需要状态,比如购物车,登录
但HTTP是无状态的,无法区分用户的身份
为了标识和认证一个用户,就需要用到cookie

## 步骤
1.客户端先请求服务器端,服务器端向客户端发送cookie
2.客户端保存cookie
3.以后每次请求服务器的时候会将此cookie发送回服务器端

## 原理
网站为了辨别用户的身份,进行会话跟踪而要在客户端上存储数据

## cookie问题
cookie过多会引起请求头过大
1.浪费带宽
2.容易被篡改所以使用前要先校验
3.不要信任cookie里的数据
4.设置正确的domain和path

## cookie的建议
1.存储的内容不宜过多
2.正确设置domain
3.有些web服务器对请求头的大小有限制,比如apache 8k
4.敏感数据不能放到cookie中isAdmin 支付宝的账户余额

## 会话 session
敏感数据应该存在什么地方?

## cookie demo
```
var http = require('http');
var url = require('url');
var querystring = require('querystring');
/**
 * Domain 域名
 * Path 路径 控制访问哪些路径的时候发送cookie
 * Expires/Max-Age 设置cookie的过期时间或有效期
 * HTTP 只能通过HTTP查看,不能通过js操作
 * Secure 只限于https服务器使用
 */
http.createServer(function(req,res){
    var urlObj = url.parse(req.url);
    var pathname = urlObj.pathname;
    if(pathname=='/favicon.ico'){
        res.end('404');
    }else if(pathname == '/write'){
        res.setHeader('Set-Cookie','name=coffee; Expires='+new Date(new Date().getTime()+10000).toGMTString());//设置一个header,名称固定为Set-Cookie
        res.end('write');
    }else if(pathname == '/read'){
        var cookie = req.headers.cookie;
        var cookieObj = querystring.parse(cookie,'; ');
        if(cookieObj && cookieObj.name){
            res.end('old');
        }else{
            res.end('new');
        }
    }
}).listen(8080);
```

## session demo
```
/**
 1 用户第一次访问服务器的时候，服务器生成一个卡号给用户。 卡号默认100余额
 2 以后每次用户再次请求服务器的时候，把卡号发回服务器。然后服务器把此卡号对应的余额减掉10
 **/
var http = require('http');
var url = require('url');
var querystring = require('querystring');
var CARD_NUM = 'card_num';
var sessions = {};
/**
 * 1.先访问 / 显示登陆页面。
 * 2.用户填 写用户名进行提交，提交到/login  把用户名和余额写到session里
 * 3.访问 /home页面，显示用户名和余额，每访问一次余额-10.
 *
 */
http.createServer(function(req,res){
    var urlObj = url.parse(req.url);
    var pathname = urlObj.pathname;
    if (pathname == '/favicon.ico') {
        res.end('404');
    }else if(pathname == '/'){
      //返回一个登陆页面 只需要用户名username
      //  用户输入用户名之后进行登陆。
    }else if(pathname == '/login'){
        var cookie = req.headers.cookie;
        var cookieObj = querystring.parse(cookie,'; ');
        var no = new Date().getTime()+Math.random();
        res.setHeader('Set-Cookie',CARD_NUM+'='+no);
        sessions[no] = 100;
        res.setHeader('Content-Type','text/html;charset=utf-8');
        res.end('欢迎新朋友，给你一张卡，卡号是'+no+',余额是100');
    }else if(pathname == '/home'){
        if(cookieObj[CARD_NUM]){
            var no = cookieObj[CARD_NUM];
            sessions[no] = sessions[no] - 10;
            res.setHeader('Content-Type','text/html;charset=utf-8');
            res.end('欢迎老朋友，,余额是'+sessions[no]);
        }else{
            res.setHeader('Content-Type','text/html;charset=utf-8');
            res.end('你没有登陆过');
        }
    }
}).listen(8080);
```