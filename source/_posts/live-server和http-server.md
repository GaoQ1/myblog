---
title: live-server和http-server
date: 2016-10-26 11:04:05
tags:
- live-server
- http-server
- nodejs
categories: 教程
---
> I find that the harder I work, the more luck I seem to have.

## 概括
前端开发的过程中，总有这样的需求。想快速在移动端，PC端看看页面的效果，看看交互的效果。通常我们会自己搭建一个服务器，或简单或复杂。
这里推荐的两个live-server和http-server是基于nodejs的快速搭建服务器的插件。

## 开始使用
当然首先在用这两个live-server和http-server的前提是你要安装nodejs。
然后起个terminal窗口，全局安装live-server和http-server。
```javascript
  npm install -g live-server
  npm install -g http-server
```
安装完毕后，你就可以在你需要其服务的目录下直接输入`live-server`或者`http-server`

## [live-server](https://github.com/tapio/live-server)和[http-server](https://github.com/indexzero/http-server)的区别

`live-server`提供的功能是：
 1. 由于安全限制，AJAX的请求不能是`file://`的协议。因此你需要创建一个server来调用AJAX请求
 2. `live-server`另一个用途是，当所要监控的文件发生变化，页面会自动刷新，且十分方便简单。

`http-server`功能就略微单一，只是一个简单的无须配置的http服务器。

## live-server和http-server的命令行的使用
关于live-server和http-server命令行的使用，比如如何设置port、host等等，具体的请查看对应的github进行查看。
