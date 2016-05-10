---
title: reduce的用法
date: 2016-05-09 23:05:47
tags:
- javascript
- reduce
categories: 教程
---
> Real dream is the other shore of reality.

## 概述
reduce() 方法接收一个函数作为累加器(accumulator)，数组中的每个值（从左到右）开始合并，最终为一个值。

## 语法
```javascript
    arr.reduce(callback,[initialValue])
```

### 参数
执行数组中的每个值得函数，包含四个参数

previousValue -- 上一次回调函数返回的值，或者是提供的初始值(initialValue)

currentValue -- 数组中当前被处理的元素

index -- 当前元素在数组中的索引

array -- 调用reduce的数组

initialValue -- 作为第一次调用callback的第一个参数。

## 描述
reduce为数组中的每一个元素依次执行回调函数，不包括数组中被删除或从未被赋值的元素，接受四个参数：初始值（或者上一次回调函数的返回值），当前元素，当前索引，调用reduce的数组。

回调函数第一次执行时，preciousValue和currentValue可以是一个值，如果initialValue在调用reduce时被提供，那么第一个preciousValue等于initialValue，并且currentValue等于数组中的第一个值；如果initialValue未被提供，那么previousValue等于数组中的第一个值，currentValue等于数组中的第二个值。

如果数组为空并且没有提供initialValue，会抛出TypeError。如果数组仅有一个元素（无论位置如何）并且没有提供initialValue，或者有提供initialValue但是数组为空，那么此唯一值将被返回并callback不会被执行。

例如下面的代码

```javascript
    [0,1,2,3,4].reduce(function(previousValue,currentValue,index,array){
        return previousValue + currentValue;
    })
```

回调函数被执行了四次，每次的参数和返回值如下表

|   | previousValue | currentValue | index | array | return value |
| ----- | :-----: | :-----: | :-----: | :-----: | -----: |
| first call | 0 | 1 | 1 | [0,1,2,3,4] | 1 |
| second call | 1 | 2 | 2 | [0,1,2,3,4] | 3 |
| third call | 3 | 3 | 3 | [0,1,2,3,4] | 6 |
| fourth call | 6 | 4 | 4 | [0,1,2,3,4] | 10 |

reduce 的返回值是回调函数的最后一次被调用的返回值10。

如果把初始值作为第二个参数传入reduce，最终返回值变为20，结果如下：

```javascript
    [0,1,2,3,4].reduce(function(previousValue,currentValue,index,array){
        return previousValue + currentValue;
    },10);
```

## 栗子
1. 将数组所有项相加

```javascript
    var total = [0,1,2,3].reduce(function(a,b){
        return a + b;
    });
    //total 6
```

2. 将数组扁平化

```javascript
    var flattened = [[0,1],[2,3],[4,5],[6,7]].reduce(function(a,b){
        return a.concat(b);
    });
    // flattened is [0,1,2,3,4,5]
```

## 兼容旧环境(Polyfill)
Array.prototype.reduce 被添加到ECMA-262标准第5版；因此可能在某些环境中不被支持。可以将下面的代码插入到脚本开头来允许在那些未被原生支持reduce的实现环境中使用它。

```javascript
    if('function' !== typeof Array.prototype.reduce){
        Array.prototype.reduce = function(callback,opt_initialValue){
            'use strict'
            if(null == this || 'undefined' === typeof this){
                throw new TypeError('Array.prototype.reduce called on null or undefined');
            }
            if('function' !== typeof(callback)){
                throw new TypeError(callback + 'is not a function');
            }
            var index, value, length = this.length >>> 0, isValueSet = false;
            if(1<argument.length){
                value = opt_initialValue;
                isValueSet = true;
            }
            for(index = 0;length > index; ++index){
                if(this.hasOwnProperty(index)){
                    if(isValueSet){
                        value = callback(value,this[index],index,this);
                    }else{
                        value = this[index];
                        isValueSet = true;
                    }
                }
            }
            if(!isValueSet){
                throw new TypeError('Reduce of empty with no initial value');
                return value;
            }
        }
    }
```








