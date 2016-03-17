title: node-07npm
date: 2015-11-09 15:07:44
tags:
- node
- npm
- 学习笔记
categories: 笔记
---

# 模块管理
## 模块的安装
- 本地安装
npm install mime
- 全局安装
npm install mime -g

## 创建自己的模块并发布
```
mkdir gaoquan
cd gaoquan
npm init
创建index.js文件
npm publish 发布npm
```

## 搜索项目
npm search xxx

## 查看项目的信息
npm view xxx

## 查看全局安装目录
npm root -g

## 配置文件
### 用户的配置
C:\Users\Administrator\npmrc
### 全局配置
C:\Program File\nodejs\node_modules\npm\npmrc
prefix=${APPDATA}\npm
npm config set prefix "d:\"

### 查看本地安装的所有模块
npm list
npm list -g

### 取消本地安装的模块
npm uninstall xxx

### 更新本地安装的所有模块
npm update

### 下载指定版本的模块
npm install xxx@version

### 保存所有记录
doskey /history
