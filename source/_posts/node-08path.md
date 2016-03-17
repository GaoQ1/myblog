title: node-08path
date: 2015-11-09 16:32:21
tags:
- node
- path
- 学习笔记
categories: 笔记
---

# 级联创建和删除目录
```
var fs = require('fs');
var path = require('path');
fs.mkdirP = function(dir){
    var paths = dir.split('/'); //本操作系统文件目录分隔符
    for(var i=0;i<paths.length;i++){
        var currentPath = paths.slice(0,i+1).join('/');
        if(!fs.existsSync(currentPath)){
            fs.mkdirSync(currentPath);
        }
    }
}
fs.rmdirP = function(dir){
    var paths = dir.split('/');
    for(var i=paths.length;i>0;i--){
        var currentPath = paths.slice(0,i).join('/');
        console.log(currentPath);
        if(fs.existsSync(currentPath)){
            fs.rmdir(currentPath);
        }
    }
}
fs.mkdirP('1/1/1',function(err){
    if(err){
        console.log(err);
    }else{
        console.log('创建目录成功');
    }
});
fs.rmdirP('1/1/1');
```

# fs的方法
- fs.stat
```
fs.stat(path,function(err,stat){
    console.log(stat.size);
})
```
- fs.exists 判断文件是否存在
- fs.realpath 取得文件的绝对路径
- fs.rename 对文件进行重命名或者移动
- fs.truncate 对文件进行截断
```
fs.truncate(filename,5,function(err){ ... })
```

# fs的watchfile
```
var fs = require('fs');
// curr 当前的状态
// prev 修改之前的状态
var func1 = function(curr,prev){
    if(Date.parse(prev.ctime) == 0){
        console.log('文件被创建');
    }else if(Date.parse(curr.ctime) == 0){
        conosle.log('文件被删除');
    }else{
        console.log('文件被修改了');
    }
}
fs.watchFile('write.txt',{interval:1000},func1);
```

# path的一些方法
## path.dirname(...) //取得参数的父目录
## path.basename(...) //取得文件名
## path.basename('write.txt','.txt') //只取得文件名,去除后缀
## path.extname(...) //取得扩展名
## path.sep //路径分隔符
## path.delimiter //环境变量路径分隔符
