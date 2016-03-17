title: zepto学习笔记
date: 2015-10-27 14:21:46
tags:
- zepto
- 学习笔记
categories: 笔记
---

> Zepto的设计目的是有一个5-10K的通用库、下载并快速执行、有一个熟悉通用的API。

| module | default | description |
| ------ | :-------: | :----------- |
| zepto	| ✔	| 核心模块；包含许多方法 |
| event	| ✔|	通过on()& off()处理事件 |
| ajax	| ✔|	XMLHttpRequest 和 JSONP 实用功能 |
| form	| ✔|	序列化 & 提交web表单 |
| ie	|✔| 	增加支持桌面的Internet Explorer 10+和Windows Phone 8。 |
| detect| |		提供 $.os和 $.browser消息 |
| fx		| | The animate()方法 |
| fx_methods|	|	以动画形式的 show, hide, toggle, 和 fade*()方法. |
| assets|	|	实验性支持从DOM中移除image元素后清理iOS的内存。 |
| data	|	|一个全面的 data()方法, 能够在内存中存储任意对象。 |
|deferred |	|	提供 $.Deferredpromises API. 依赖"callbacks" 模块. 当包含这个模块时候, $.ajax() 支持promise接口链式的回调。 |
| callbacks	| |	为"deferred"模块提供 $.Callbacks。 |
| touch		| | 在触摸设备上触发tap– 和 swipe– 相关事件。这适用于所有的`touch`(iOS, Android)和`pointer`事件(Windows Phone)。 |
| gesture	| |	在触摸设备上触发 pinch 手势事件。 |
| stack		| | 提供 andSelf& end()链式调用方法 |
| ios3		| | String.prototype.trim 和 Array.prototype.reduce 方法 (如果他们不存在) ，以兼容 iOS 3.x. |

## 如何自制zepto
```
    1.git clone https://github.com/madrobby/zepto.git
    2.npm install 下载package.json里dependence
    3.MODULES="zepto event data ..." npm run-script dist
    the resulting files are:
    1.dist/zepto.js
    2.dist/zepto.min.js
```

--- 
   
### 整体上zeptojs的用法和jquery一样