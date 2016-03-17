title: node-09stream
date: 2015-11-09 17:57:14
tags:
- node
- stream
- 学习笔记
categories: 笔记
---

> - 流是一组有序的,有起点和终点的数据传输手段
  在网络中传输数据的时候,总是先将对象转化为流数据,也就是字节数组.
  再通过流的传输,到达目的地后再转成(内容类型和编码)原始的数据
  - 流是一个接口,不同的业务有不同的实现.
  fs ReadStream
  http.IncomingMessage 客户端请求或客户端响应request response

## 直接读取
```
var fs = require('fs');
var rs = fs.createReadStream('index.txt',{start:2,end:3});
/**
 * 1.ReadStream 继承EventEmitter
 * 2.open 打开这个文件
 * 3.read 读到64k数据到buffer,发射data事件
 */
rs.setEncoding('utf8');
rs.on('data',function(data){
    console.log(data);
})
```

## 非流动模式读取
```
/**
 * 非流动模式 暂停模式
 */
var fs= require('fs');
var rs = fs.createReadStream('65.txt');
var arr = [];
rs.on('readable',function(){
    console.log('readable');
    var data;
    while(null != (data = rs.read())){ //从缓存区里读取1个字节
        arr.push(data);
        console.log(data.length);
    }
});
rs.on('end',function(){
    var b = Buffer.concat(arr);
    console.log(b.length);
})
```

## 写入文件
```
var fs = require('fs');
var out = fs.createWriteStream('write.txt');
for(var i=0;i<100;i++){
    var flag = out.write(i.toString());
    console.log(flag);
}
//操作系统缓存区抽干的时候触发
out.on('drain',function(){
    console.log('drain');
})
```

## copy文件
```
var fs = require('fs');
var rs = fs.createReadStream('index.txt');
var ws = fs.createWriteStream('write.txt');
rs.on('data',function(chunk){
    if(ws.write(chunk)==false){
        rs.pause();
    }
});
ws.on('drain',function(){
    rs.resume();
});
rs.on('end',function(){
    ws.end();
});
//pipe方法copy文件
var fs = require('fs');
var rs = fs.createReadStream('read.txt');
var ws = fs.createWriteStream('write.txt');
rs.pipe(ws);
```