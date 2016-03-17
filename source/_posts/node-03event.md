title: node-03event
date: 2015-10-28 21:11:51
tags:
- node
- event
- 学习笔记
categories: 笔记
---

## 自定义event事件
```
function Event(){
    this._events = {};
}

//注册事件
Event.prototype.on = function(eventName,listener){
    if(this._events[eventName]){

    }else{
        this._events[eventName] = [listener];
    }
}

//发射事件
Event.prototype.emit = function(eventname){
    for(var i=0;i<this._events[eventname].length;i++){
        this._events[eventname][i].call(this);
    }
}

var button = new Event();
button.on('click',function(){
    console.log('clicked');
});
button.emit('click');
```

## 自定义多个事件，且传递参数
```
function Event(name){
    this.name = name;
    this._events = {};
}

//注册事件
Event.prototype.on = function(eventName,listener){
    if(this._events[eventName]){
        this._events[eventName].push(listener);
    }else{
        this._events[eventName] = [listener];
    }
}

//发射事件
Event.prototype.emit = function(eventname){
    var handles = this._events[eventname];
    var args = Array.prototype.slice.call(arguments,1);
    for(var i=0;i<handles.length;i++){
        this._events[eventname][i].apply(this,args);
    }
}

var button = new Event("confirm");
function show1(words){
    console.log(this.name,'show1',words);
}
function show2(words){
    console.log(this.name,'show2',words);
}
button.on('click',show1);
button.on('click',show2);
button.emit('click','welcome');
```

## node封装的event
```
var EventEmitter = require('events').EventEmitter;
var util = require('util');
function Teacher(name){
    this.name = name;
}

util.inherits(Teacher,EventEmitter);

var coffee = new Teacher();
coffee.on('hungry',function(){
    console.log('吃饭');
});

coffee.emit('hungry');
```
