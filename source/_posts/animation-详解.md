---
title: animation-详解
date: 2016-04-04 16:22:51
tags:
- animation
- css3
categories: 教程
---
>A glamorous life is quite different to a life of luxury.

## 简介 
animation属性是动画属性的简写，其中包括：
- animation-name
- animation-duration
- animation-timing-function
- animation-delay
- animation-iteration-count
- animation-direction
- animation-fill-mode

## 用法
animation可以定义多个动画属性，需要用逗号分隔。比如： 
```javascript
/*一个animation定义的时候*/
animation: [animation-name] [animation-duration] [animation-timing-function] [animation-delay] [animation-iteration-count] [animation-fill-mode];
/*多个animation定义的时候*/
animation: [animation-name] [animation-duration] [animation-timing-function] [animation-delay] [animation-iteration-count] [animation-fill-mode],
[animation-name] [animation-duration] [animation-timing-function] [animation-delay] [animation-iteration-count] [animation-fill-mode];
```
其中简写的animation属性用空格分隔，而且他们之间的排序没什么影响，除了animation-duration和animation-delay，他们需要顺序。也就是说当你使用两个time参数的时候，第一个时间指的是animation-duration第二个时间指的是animation-delay。

```javascript
animation: bounce 0.3s ease-in-out 1s infinite;
/*equivalent to*/
animation-name: bounce;
animation-duration: 0.3s;
animation-timing-function: ease-in-out;
animation-delay: 1s;
animation-iteration-count: infinite;
```

## animation-play-state
animation-play-state属性决定CSS animation是运行还是暂停。
```javascript
animation-play-state: running | paused
```