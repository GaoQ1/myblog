---
title: webapck直接上手
date: 2016-03-22 14:26:43
tags:
- webpack
- 快速上手
categories: 教程
---
>True mastery of any skill takes a lifetime.

写在前面的话，花了一天一夜学得的webpack，并且都把webapck.config.js配置完成了，leader不让用，还说不要盲目跟风。无语，遂把学得的东西贡献出来。

## Getting started

首先，全局下载[webpack](https://www.npmjs.com/package/webpack)和[live-server](https://www.npmjs.com/package/live-server)。  
***ps***：这边可自行选择，[live-server](https://www.npmjs.com/package/live-server)或是[webpack-dev-server](https://www.npmjs.com/package/webpack-dev-server)，我用的是live-server。

```javascript
$ npm install -g webpack live-server 
```

然后，git直接clone本仓库以及依赖包

```javascript
$ git clone https://github.com/GaoQ1/webpack-start.git
$ cd webpack-scaffold
$ npm install
```

现在你可以试试webpack的效果了。

```javascript
$ webpack
$ live-server
```

浏览器会自动打开，然后点击src，访问路径为[http://127.0.0.1:8080/src/](http://127.0.0.1:8080/src/)。 

## 什么是webpack
至于什么是webpack，网上有很多资料，我就不copy过来了，详细的可参考
 - [Webpack中文指南](http://zhaoda.net/webpack-handbook/)
 - [Webpack官网](http://webpack.github.io/docs/)

还有关于webpack的常用的命令行有：
```javascript
 $ webpack //用来一次性执行开发环境webpack
 $ webpack -p //用来一次性执行生产环境webpack
 $ webpack --watch //用来持续的执行webpack
 $ webpack -d //包含source maps文件
 $ webpack --colors //用来让wenbpack执行后的文件更优美
```

## 目录结构
```javascript
--- dist (webpac打包后的文件)
    |--- ...
--- images (图片文件)
    |--- ...
--- src (webpack打包前源文件)
    |--- js (源文件组件文件夹)
    |    |--- client.js (javascript文件)
    |    |--- image.js (image文件)
    |    |--- style.js (css文件)
    | 	 |...
    |--- sass (sass文件夹)          
    |    |--- app.scss (sass文件)
    |    |---
    |--- index.html (主页面)
    |--- package.json (配置文件)
    |--- webpack.config.js (webpack配置文件) 
```

## 参考文章
关于webpack.config.js的文件配置参考了
 - 阮大神的[webpack-demos](https://github.com/ruanyf/webpack-demos)
 - Petehunt大神的[webpack-howto](https://github.com/petehunt/webpack-howto)
 - [Webpack documents](https://webpack.github.io/docs/)