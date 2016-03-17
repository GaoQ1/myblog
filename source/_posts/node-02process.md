title: node-02process
date: 2015-10-27 22:42:54
tags:
- node
- process
- 学习笔记
categories: 笔记
---

```
console.log('hello')==>process.stdout.write('hello')

//当控制台接收到回调函数之后会调用回调函数进行处理
process.stdin.on('data',function(data){
    process.stdout.write(data);
})

```

```
process.argv.forEach(function(item){
    console.log(item);
})

//当程序退出的时候进行清理工作
process.on('exit',function(){
    console.log('clear');
})

//未捕获到的异常
process.on('uncaughtExpection',function(err){
    console.log(err);
})

//内存的使用量
console.log(process.memoryUsage());

//查看工作目录
console.log(process.cwd());

//下一个事件环的时候执行,异步执行,优先级最高
process.nextTick(...);

```