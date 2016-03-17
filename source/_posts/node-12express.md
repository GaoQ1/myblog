title: node-12express
date: 2015-11-11 11:01:12
tags:
- node
- express
- 学习笔记
categories: 笔记
---

## 中间件
```
/**
 中间件
 是处理HTTP请求，用来完成各种特定的任务
 比如检查用户是否登陆，增加工具方法等
 最大特点:
    一个中间件处理完毕后，才能将数据传递给下一个中间件
    app.use(路径,中间件函数);
 **/
var express = require('express');
var app = express();
var fs = require('fs');
app.use(function(req,res,next){
    res.print = function(content){
        this.end(content);
    }
    next();
});
/*app.use(function(req,res,next){
    console.log('next middleware');
});*/
//发表文章
app.get('/article',function(req,res){
    res.print('article');
});
//发表评论
app.get('/comment',function(req,res){
    res.print('article');
});
//登陆
app.get('/login',function(req,res){
    res.print('login');
});
//注册
app.get('/reg',function(req,res){
    res.print('login');
});
//匹配所有的路径和所有的请求方法
app.all('*',function(req,res){
    res.print('404');
});
app.listen(8080);
```

## 静态文件
```
var express = require('express');
var app = express();
var fs = require('fs');
//静态文件目录服务中间件 /index.html
// __dirname+/index.html
app.use(express.static(__dirname));
app.get('/',function(req,res){
    res.end('404');
});
app.listen(8080);
```

## 获取参数
```
var express = require('express');
var app = express();
var fs = require('fs');
/**
 * 请求方法method
 * 请求的路径 pathname
 * 查询字符串 query
 * 请求头 headers
 *
 */
var url = require('url');
app.use(function(req,res,next){
    req.path1 = url.parse(req.url,true).pathname;
    next();
});
app.get('/login',function(req,res){
    console.log(req.method);//请求的方法
    console.log(req.path1);//请求的pathname
    console.log(req.query);//请求的查询字符串对象
    console.log(req.headers);//请求头对象
    res.end('404');
});
//路径参数 http://localhost:8080/user/delete/1/zfpx
app.get('/user/delete/:id/:name',function(req,res){
    console.log(req.params.id);//
    console.log(req.params.name);//
    res.end('404');
});
app.listen(8080);
```

## 使用模板
```
var express = require('express');
var app = express();
var fs = require('fs');
var path = require('path');
var url = require('url');
//指定模板引擎 ejs handlerbar jade
// freemarker velocity
app.set('view engine','html');
app.set('views',path.join(__dirname,'tmpl'));
//对于html类型的模板调用__express来进行渲染
app.engine('html',require('ejs').__express);
app.get('/login',function(req,res){
    // __dirname/tmpl/login.ejs
  res.render('login',{username:'zhangsan',hello:'<h1>hello</h1>'});
});
app.listen(8080);
```

## 参数 req.body
```
var express = require('express');
var app = express();
var fs = require('fs');
/**
 * 获取请求体参数
 * post 请求中如何获取客户端提交的数据
 */
var path = require('path');
var url = require('url');
app.set('view engine','html');
app.set('views',path.join(__dirname,'tmpl'));
app.engine('html',require('ejs').__express);
var bodyParser = require('body-parser');
//处理文件上传的中间件 只解析类型是 multipart/form-dat的类型
var multer = require('multer');
var storage = multer.diskStorage({
  //指定保存的目的地
  destination: function (req, file, cb) {
    cb(null, 'uploads/')
  },
  //指定保存文件名
  filename: function (req, file, cb) {
    cb(null, Date.now()+'-'+file.originalname)
  }
})
var upload = multer({ storage: storage })
app.use(bodyParser.json());//一个是json字符串 req.body
app.use(bodyParser.urlencoded({extended:true}));//对象 req.body
app.get('/login',function(req,res){
  res.render('login');
});
app.use(express.static(__dirname));
// username=zfpx&password=123 querystring.parse() ->req.body
app.post('/login',upload.single('avatar'),function(req,res){
  console.log(req.body.username);
  console.log(req.body.password);
  console.log(req.file);
  res.setHeader('Content-Type',"text/html;charset=utf-8");
  //'uploads\\1445763597971-get_matrix_img.gif'
  //  /uploads\\1445763597971-get_matrix_img.gif
  res.end('<img src="'+req.file.path+'">')
});
app.listen(8080);
```

## 访问登录 demo
```
/**
  * 实现一个完整的访问控制
  * /login 当登陆成功之后会跳转到/home页
  * /home 访问/home页的时候要判断用户是否已经登陆，如果未登陆，跳回登陆页
  */
var express = require('express');
var app = express();
var fs = require('fs');
/**
  * 获取请求体参数
  * post 请求中如何获取客户端提交的数据
  */
var path = require('path');
var url = require('url');
app.set('view engine','html');
app.set('views',path.join(__dirname,'tmpl'));
app.engine('html',require('ejs').__express);
var session = require('express-session');
var bodyParser = require('body-parser');
// req.session
app.use(session({
    secret:'secret',
    resave:true,
    saveUninitialized:true
}));
app.use(express.static(__dirname));
app.use(bodyParser.urlencoded({extended:true}));//对象 req.body
app.get('/login',function(req,res){
    res.render('login');
});
app.post('/login',function(req,res){
    var user = req.body;
    req.session.user = user;
    res.redirect('/home');//重定各到/home
});
function checkAuth(req,res,next){
    if(req.session.user){
        next();
    }else{
        res.redirect('/login');//重定各到/home
    }
}
app.get('/home',checkAuth,function(req,res){
    res.render('home',req.session.user);
});
app.listen(8080);
```

## error处理机制
```
var express = require('express');
var app = express();
var fs = require('fs');
app.use(function(req,res,next){
    console.log('1');
    next();
});
app.use(function(req,res,next){
    console.log('2');
    //throw Error('wrong');
    next('I am wrong');
});
app.use(function(req,res,next){
    console.log('3');
    next();
});
app.use(function(req,res,next){
    console.log('4');
    res.end("4");
});
//错误处理中间件
app.use(function(err,req,res,next){
   res.end(err);
});
app.listen(8080);
```