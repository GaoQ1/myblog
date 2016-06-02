---
title: es6系列教程三
date: 2016-06-02 20:45:03
tags:
  - es6
  - javascript
categories: 原创
---
> Experience is the mother.

# Destructuring
解构赋值(destructuring assignment)语法是一个Javascript表达式，它使得从数组或者对象中提取数据赋值给不同的变量成为可能。

## 语法
```javascript
  [a,b] = [1,2];
  [a,b,...rest] = [1,2,3,4,5];
  {a,b} = {a:1,b:2}
  {a,b,...rest} = {a:1,b:2,c:3,d:4} //ES7
```

> {a,b} = {a:1,b:2}作为独立语法是非法的，左侧的{a,b}会被当成块结构而不是一个对象。
然而({a,b} = {a:1,b:2})的形式是允许的，其等价于var {a,b} = {a:1,b:2}.  

## 简述
对象字面量和数组字面量提供了一种简单的定义一个特定的数据组的方法。一旦你创建了这类数据组，你可以用任意的方法使用这些数据组，甚至从函数中返回它们。

解构赋值的一个特别有用的功能是：你可以用一个表达式读取整个结构。你可以从后面的例子中了解到很多它的其他的有趣应用。

解构赋值的作用类似于Perl和Python语言中的相似特性。

## 解构数组
### 简单例子
```javascript
  var foo = ["one","two","three"];

  //没有解构赋值
  var one = foo[0];
  var two = foo[1];
  var three = foo[2];

  //解构赋值
  var [one,two,three] = foo;
```

### 变换变量
```javascript
  var a = 1;
  var b = 2;
  [a,b] = [b,a];
```
执行这段代码后，b是1、a是3.没有解构赋值的情况下，交换两个变量需要一个临时变量.

### 返回多值
感谢解构赋值，函数现在可以返回多个值。尽管函数一直都可以返回一个数组，但现在这样做有更多的灵活性。
```javascript
  function f(){
    return [1,2];
  }
```

如你所见，两个返回值被括号括起来，用类似数组表示法的方式被返回。你可用这种方法返回任意量的变量。这个方法中，f()返回[1,2].
```javascript
  var a,b;
  [a,b] = f();
  console.log("A is " + a + " B is " + b);
```

[a,b] = f()把函数的返回值按顺序赋值给括号中的变量：a赋值为1、b赋值为2.

你也可以把返回值当作数组对待：
```javascript
  var a = f();
  console.log("A is " + a);
```
这样a就是一个包含1和2两个值的数组。

### 忽略某些返回值
你也可以忽略你不感兴趣的返回值：
```javascript
  function f(){
    return [1,2,3];
  }

  var [a,b] = f();
  console.log("A is " + a + " B is " + b);
```

运行这段代码后，a是1、b是3。2这个值被忽略，你可以忽略任意(或全部)返回值。例如：
```javascript
  [,,] = f();
```

### 用正则表达式匹配提取值
用正则表达式方法exec()匹配字符串返回一个数组，该数组第一个值是完全匹配正则表达式的字符串，然后的值是匹配正则表达式括号内内容部分。解构赋值允许你轻易地提取出需要的部分，忽略完全匹配的字符串--如果不需要的话。
```javascript
  var url = "https://developer.mozilla.org/en-US/Web/JavaScript";

  var parsedURL = /^(\w+)\:\/\/([]^\/]+)\/(.*)$/.exec(url);
  var [,protocol,fullhost,fullpath] = parseURL;

  console.log(protocol); //输出"https:"
```

## 解构对象
### 简单示例
```javascript
  var o = {p:42,q:true};
  var {p,q} = o;

  console.log(p);
  console.log(q);

  //用新变量名赋值
  var {p:foo,q:bar} = o;

  console.log(foo); //42
  console.log(bar); //true
```

### 函数参数默认值
ES5版本
```javascript
  function drawES5Chart(options) {
    options = options === undefined ? {} : options;
    var size = options.size === undefined ? 'big' : options.size;
    var cords = options.cords === undefined ? { x: 0, y: 0 } : options.cords;
    var radius = options.radius === undefined ? 25 : options.radius;
    console.log(size, cords, radius);
    // now finally do some chart drawing
  }

  drawES5Chart({
    cords: { x: 18, y: 30 },
    radius: 30
  });
```

ES6版本
```javascript
  function drawES6Chart({size = 'big',cords = {x:0,y:0},radius = 25} = {}){
    console.log(size,cords,radius);
  }

  drawES6Chart({
    cords: {x:18,y:30},
    radius: 30
  })
```

### 解构嵌套对象和数组
```javascript
  var metadata = {
    title: "Scratchpad",
    translations: [
      {
        locale: "de",
        localization_tags: [ ],
        last_edit: "2014-04-14T08:43:37",
        url: "/de/docs/Tools/Scratchpad",
        title: "JavaScript-Umgebung"
      }
    ],
    url: "/en-US/docs/Tools/Scratchpad"
  };

  var { title: englishTitle, translations: [{ title: localeTitle }] } = metadata;

  console.log(englishTitle); // "Scratchpad"
  console.log(localeTitle);  // "JavaScript-Umgebung"
```



















-----------------
