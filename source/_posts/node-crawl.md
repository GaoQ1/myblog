title: node-crawl
date: 2015-12-01 16:03:21
tags:
- request
- cheerio
- 爬虫
categories: 笔记
---

## 项目介绍
> request+cheerio+mysql

## read
```
var request = require('request');
var cheerio = require('cheerio');
var util = require('util');
var debug = require('debug')('crawl:read');
exports.classList = function(url,callback){
    debug('读取文章的分类信息:%s',url);
    request(url,function(err,res){
        if(err)
            return callback(err);
        var $ = cheerio.load(res.body.toString());
        var classList = [];
        $('.panel_body li a').each(function(){
            var $this = $(this);
            var item = {
                name:$this.text().trim(),
                url:"http://blog.csdn.net"+$this.attr('href')
            };
            var s = item.url.match(/\/article\/category\/(\d+)/);
            if(Array.isArray(s)){
                item.id = s[1];
                classList.push(item);
            }
        });
        callback(err,classList);
    })
};
exports.classArticles = function(url,callback){
    debug('读取分类下面的文章列表:%s',url);
    request(url,function(err,res){
        if(err)
            return callback(err);
        var articleList = [];
        var $ = cheerio.load(res.body.toString());
        $('#article_list .article_item').each(function(){
            var $this = $(this);
            var $title = $this.find('.link_title a');
            var $time = $this.find('.article_manage .link_postdate');
            var item = {
                title:$title.text().trim(),
                url: "http://blog.csdn.net/"+$title.attr('href'),
                time:$time.text().trim()
            }
            var s = item.url.match(/\/article\/details\/(\d+)/);
            if(Array.isArray(s)){
                item.id = s[1];
                articleList.push(item);
            }
        });
        callback(err,articleList);
    })
};
exports.articleDetail = function(url,callback){
    debug('读取文章的详情:%s',url);
    request(url,function(err,res){
        if(err)
            return callback(err);
        var tags = [];
        var $ = cheerio.load(res.body.toString());
        $('.tag2box a').each(function(){
            var tag = $(this).text().trim();
            if(tag){
                tags.push(tag);
            }
        });
        var content = $('.article_content').html().trim();
        callback(err,{tags:tags,content:content});

    })
};
```

## write
```
var mysql = require('mysql');
var debug = require('debug')('crawl:write');
var async = require('async');
var connection = mysql.createConnection({
    host:'127.0.0.1',
    user:'root',
    password:'',
    database:'coffee'
});
connection.connect();
exports.classList = function(classList,callback){
    debug('保存文章的分类列表');
    async.eachSeries(classList,function(item,next){
        connection.query('replace into class_list(id,name,url) value(?,?,?)',[item.id,item.name,item.url],next);
    },callback)
};
exports.classArticles = function(class_id,list,callback){
    debug('保存分类下面的文章列表%d',class_id);
    async.eachSeries(list,function(item,next){
        connection.query('replace into article_list(id,title,url,postdate,class_id) value(?,?,?,?,?)',[item.id,item.title,item.url,item.time,class_id],next);
    },callback)
};
exports.articleDetail = function(id,title,tags,content,callback){
    debug('保存文章的详情%d',id);
    connection.query('replace into article_detail(id,title,tags,content) values(?,?,?,?)',[id,title,tags.join(','),content],callback);
}
```

## app
```
var read = require('./read');
var write = require('./write');
var debug = require('debug')("craw:app");
var async = require('async');
var classList;
var classArticles = {};
var articles = [];//所有的文章的ID列表
var url = 'http://blog.csdn.net/hongqishi';
async.series([function(next){
    read.classList(url,function(err,list){
        classList = list;
        next(err);
    })
},function(next){
    write.classList(classList,next);
},function(next){
    //读取每个分类下面的文章列表
    async.eachSeries(classList,function(cls,done){
        read.classArticles(cls.url,function(err,list){
            classArticles[cls.id] = list;//key=分类ID value=文章列表
            done(err);
        });
    },next);
},function(next){
    //保存每一个分类和分类下面的文章
    async.eachSeries(Object.keys(classArticles),function(class_id,done){
        write.classArticles(class_id,classArticles[class_id],done)
    },next);
},function(next){
    debug('消除重复的文章');
    var article = {};
    Object.keys(classArticles).forEach(function(class_id){
        classArticles[class_id].forEach(function(item){
            article[item.id] = item;
        });
    });
    Object.keys(article).forEach(function(id){
        articles.push(article[id]);
    });
    next();
},function(next){
    debug('保存文章详情');
    async.eachSeries(articles,function(item,done){
        read.articleDetail(item.url,function(err,ret){
            write.articleDetail(item.id,item.title,ret.tags,ret.content,done);
        })
    },next);
}],function(err,result){
    if(err)
        console.error(err);
    console.log('done');
});
```

## crontab
```
var crontab = require('cron').CronJob;
var job = new crontab("* * * * * * *",function(){
    console.log('每秒一次');
});
job.start();
/**
 * 秒 分 小时 日 月 dayofweek
 *  * 每xx一次
 *  a-b 第几秒到第几秒
 *  星/n 每隔多少秒
 *  a-b/n 第几秒到第几秒,每隔多少秒
 **/
```