title: node-04module
date: 2015-10-29 21:46:13
tags:
- node
- module
- 学习笔记
categories: 笔记
---

# 模块
## JS不足
- js没有模块系统，不支持封闭的作用域和依赖管理
- 没有标准库，没有文件系统和IO流API
- 也没有包管理系统
## commonjs规范
### 优点
封装功能，封闭作用域，可解决依赖问题
工作效率更高，重构方便
## commonjs in node
在node.js里，模块划分所有的功能，每个JS都是一个模块
实现require方法
npm 实现了模块的自动加载和安装依赖
## 定义模块
```
(function(exports,require,module,__filename,__dirname){
    ...
    exports = module.exports;
    return module.exports;
})
```

## 模块的加载策略
### 分类
- 原生模块 http path fs util events 编译成二进制，加载速度最快
- 文件模块 在硬盘的某个位置 非常慢

## 如何查找文件模块
### 文件模块分类
- js 脚本文件 需要先读入内存再运行
- json 文件 fs 读入内存 转化为JSON对象
- node 二进制文件 可以直接使用
### 查找的过程
- 如果参数是相对路径的话如何查找
- 如果不是路径
### 从全局路径进行加载
NODE_PATH 用;分隔的路径串
$HOME/.node_modules

```
console.log(module);
/**
* id ID
* exports 导出对象
* parent 它依赖的模块
* filename 模块的文件名
* loaded 是否加载成功
* children 被哪些模块所依赖
* path 
* cache 缓存
```

# 路径

```
var path = require('path');
var fs = require('fs');
/**
 * normalize 将非标准化的路径转化成标准化的路径
 * 1.解析. 和 ..
 * 2.多个斜杠会转成一个斜杠
 * 3.window下的斜杠会转成正斜杠
 * 4.如果以斜杠结尾会保留
 */
```

```
console.log(path.join(__dirname,'a','b'));

/**
 * resolve
 * 以应用程序为根目录，做为起点，根据参数解析出一个绝对路径
 * 1.以应用程序为根起点
 * 2. ..
 * 3.普通字符串代表子目录
 * 4. /代表绝对路径根目录
 */
console.log(path.resolve());//空代表当前的目录路径
 
/**
 * relative
 * 可以获取两个路径之间的相对关系
 * 
 */
console.log(path.relative(__dirname,'../a'));

//返回指定路径的所在目录
console.log(path.dirname(__dirname));

//basename 获取路径中的文件名
console.log(path.basename(__filename));
```