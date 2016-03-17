title: node-06file
date: 2015-10-30 20:52:37
tags:
- node
- buffer
- 学习笔记
categories: 笔记
---

## 读取文件
### 有同步和异步的方式,同步方式返回值为数据,异步方式回调函数返回数据
```
var fs = require('fs');
/**
 * 同步的方式
 */
var content = fs.readFileSync('./index.html','utf-8');
//console.log(content);

/**
 * 异步的方式
 * 尽量使用异步的方式，只有读取文件作为前置条件的时候使用同步
 */
fs.readFile('./index.html','utf8',function(err,data){
    console.log(data);
});
```

## 写文件
```
/**
 * 可以完整的写入一个文件
 * fs.writeFile
 */
var fs = require('fs');
/*fs.writeFile('./write.txt','第er行',{flag:'a',encoding:'utf8'},function(err){
    if(err){
        console.error(err);
    }
    console.log("写入成功");
})*/
//r+ 为写入
//a  为追加
fs.appendFile('./write.txt','第三行');

/**
 * base64
 * A-Za-z0-9+/
 * 把3个8位字节转化为4个6位字节，之后在6位前面补两个0，形成8位一个字节的形式
 **/
fs.readFile('./bg.png','base64',function(err,data){
    fs.writeFile('./b.png',data,'base64',function(){
        console.log('copy');
    })
})
```

## 从指定的位置开始读取文件open
```
/**
 * 如何从指定的位置开始读取文件
 * 0 stdin 标准输入
 * 1 stdout 标准输出
 * 2 stderr 错误输出
 */
var fs = require('fs');
//fd 是打开文件的索引
fs.open('./msg.txt','r',function(err,fd){
    console.log(fd);
})
/**
 * 读取文件,可以多次读取，每次读一小部分
 * fd 文件描述符
 * buffer 读到哪个buffer
 * offset buffer偏移量
 * length 写多少个字节
 * position 从文件的哪个位置开始读
 * callback
 **/

fs.open('./msg.txt','r',function(err,fd){
    var buffer = new Buffer(9);
    fs.read(fd,buffer,0,6,3,function(err,bytesRead,buf){
        console.log(bytesRead);
        console.log(buf.toString());
        fs.read(fd,buffer,6,3,9,function(err,bytesRead,buf){
            console.log(bytesRead);
            console.log(buf.toString());
        })
    })
})
```

## 写入文件
```
/**
 * Created by GaoQ on 2015/10/3.
 */
var fs = require('fs');
/***
 * fd
 * buffer buffer
 * offset 从buffer中读取的偏移量
 * length 写入的长度
 * position 写入文件的位置,position null 当前位置
 * callback
 */
/*
fs.open('./msg.txt','w',function(err,fd){
    fs.write(fd,new Buffer("天天向上"),0,6,0,function(err,byteWritten,buf){
        console.log('写入成功,写入了'+byteWritten);
        fs.write(fd,new Buffer("天天向上"),6,6,6,function(err,byteWritten,buf){
            console.log('写入成功,写入了'+byteWritten);
        })
    })
});
*/
/**
 * 写入文件之后要关闭文件
 **/
fs.open('./msg.txt','w',function(err,fd){
    console.log(fd);
    fs.fsync(fd);//把缓存区里的数据立刻同步到目标文件里去
    fs.close(fd);
    fs.open('./msg.txt','w',function(err,fd){
        console.log(fd);
    })
})
```

## copy文件
```
var BUFFER_SIZE = 8*1024;
var fs = require('fs');
function copy(src,dest){
    var buff = new Buffer(BUFFER_SIZE);
    //读取的文件索引位置,源文件描述符,目标文件描述符
    var readSoFar,fdSrc,fdDest,read;
    fdSrc = fs.openSync(src,'r');
    fdDest = fs.openSync(dest,'w');
    readSoFar = 0;
    do{
        //从源文件中读取内容,并返回实际读取到的字节数
        read = fs.readSync(fdSrc,buff,0,BUFFER_SIZE,readSoFar);
        fs.writeSync(fdDest,buff,0,read);
        readSoFar += read;
    }while(read == BUFFER_SIZE);
    fs.closeSync(fdDest);
    fs.closeSync(fdSrc);
}
copy('text.txt','text2.txt');
```