title: node-05bufer
date: 2015-10-30 11:02:22
tags:
- node
- buffer
- 学习笔记
categories: 笔记
---

> Buffer暂时存放输入输出数据的一段内存，是全局对象

## 创建buffer对象的三种方法
### new Buffer(size);
```
var buf1 = new Buffer(12);
buf1.fill(0);
```

### 数组的方式
```
var buf2 = new Buffer([1,2,3]);
```

### 字符串来创建
```
var buf3 = new Buffer('天天向上');
```
### 修改
> 字符串是只读的,不能被修改;buffer类似于数组,可以被修改

### slice
```
var substr = str.slice(1,2);
var subbuf = buf.slice(1,2);
```

### buffer <=> 字符串
```
var buf = new Buffer('天天向上');
console.log(buf.toString('utf8',4,10));
```

### buffer.write
```
/**
 * string
 * offset 写入buffer的偏移量
 * length 写入字节的长度
 * encoding
 */
buf.write('天天',0,6,'utf8');
console.log(buf.toString())
```

### 乱码处理
```
//StringDecoder
var StringDecoder = require('string_decoder').StringDecoder;
var decoder = new StringDecoder();
console.log(decoder.write(buf1));
console.log(decoder.write(buf2));
```

### Number 双精度的Double
```
//value offset
var buf = new Buffer(4);
buf.writeInt8(0,0);
buf.writeInt8(16,1);
buf.writeInt8(32,2);
buf.writeInt8(64,3);
console.log(buf);
console.log(buf.readInt8(0));
console.log(buf.readInt8(1));
console.log(buf.readInt8(2));
console.log(buf.readInt8(3));
```

```
/**
 * 高位字节 低位字节
 * big 高位字节
 * little 低位字节
 */
buf.writeInt16BE(1,0);
buf.writeInt16BE(9,2);
console.log(buf.readInt16BE(0));//1
console.log(buf.readInt16BE(2));//9
```

### 复制缓存
```
var srcBuf = new Buffer([4,5,6]);
var tarBuf = new Buffer(6);
tarBuf[0] = 1;
tarBuf[1] = 2;
tarBuf[2] = 3;
/**
 * targetBuffer 目标buffer
 * targetStart  目标的起始位置
 * sourceStart  源的起始位置
 * sourceEnd    源的结束位置
 */
srcBuf.copy(tarBuf,3,0,3);
console.log(tarBuf);
```

### buffer的其他属性
```
console.log(Buffer.isBuffer(srcBuf)); //判断是否是buffer
console.log(Buffer.byteLength("天天向上")); //返回字符串的字节长度
console.log(Buffer.isEncoding("utf8"));
```