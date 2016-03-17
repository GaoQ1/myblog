title: node-13mongoose
date: 2015-11-12 20:36:35
tags:
- node
- mongoose
- 学习笔记
categories: 笔记
---

## mongoose 是对mongodb的原生方法的封装
```
var async = require('async');
var mongoose = require('mongoose');
var connection = mongoose.createConnection('mongodb://127.0.0.1/blog');
//模型定义
var Schema = mongoose.Schema;
var ObjectId = Schema.ObjectId;//自动生成的对象ID
var AuthorSchema = new Schema({
    name:String
});
var CommentSchema = new Schema({
    content:String,
    createAt:{type:Date,default:Date.now}
});
//作者
var Author = connection.model('Author',AuthorSchema);
//文章的模型
var ArticleSchema = new Schema({
    title:{type:String,unique:true},
    content:String,
    author:{type:ObjectId,ref:'Author'},
    stat:{
        favs:Number,
        visited:Number
    },
    comments:[CommentSchema]
});
//定义文章的模型,可以与数据库进行互动
var Article = connection.model('Article',ArticleSchema);

//保存用户
/*new Author({
    name:'张三'
}).save(function(err,author){
        if(err){
            console.log(err);
        }else{
            console.log(author);
        }
    });*/

/*
ArticleSchema.pre('save',function(next){
    this.stat.visited = Math.floor(Math.random()*1000);
    if(this.title){
        next();
    }else{
        console.log('标题不能为空');
    }
});

var tasks = [];
for(var i=1;i<=10;i++){
    (function(i){
        tasks.push(function(next){
            new Article({
                title:'title'+i,
                content:'content'+i,
                author:'5644832ba5d31db01adf4eb2',
                stat:{
                    favs:0,
                    visited:0
                },
                comments:[{content:'content'+i}]
            }).save(function(err,article){
                    if(err){
                        console.log(err);
                    }else{
                        console.log(article);
                    }
                    next();
                });
        })
    })(i);
}
//串行执行,一个任务完成后才能执行下一个任务
async.series(tasks,function(err,result){
    console.log('全部插入完成');
});
*/

//分页参数 pageNumber pageSize
//返回值   pages articles
/*var pageNumber = 1;
var pageSize = 5;
Article.count({},function(err,count){//返回总记录数
    var pages = Math.ceil(count/pageSize);//计算总页数
    var skip = (pageNumber-1)*pageSize;
    Article.find({}).skip(skip).limit(pageSize).sort({createAt:1}).populate('author').exec(function(err,result){
        console.log(pages,result);
    })
});*/

/*Article.update({},{$set:{'stat.favs':100}},function(err,result){
    console.log(result);
});*/

/*Article.update({},{$push:{'comments':{'comments':'i am a student'}}},function(err,result){
    console.log(result);
});*/
/*Article.find({},function(err,result){
 console.log(result);
 });*/
/*Article.findOne({},function(err,result){
    console.log(result);
});*/

/*Article.remove({_id:'5644835a2232594013cd77c8'},function(err,result){
 console.log(result);
 });*/
```