---
title: es6系列教程四
date: 2016-06-06 12:15:00
tags:
  - es6
  - javascript
categories: 原创
---
> Fading is true while flowering is past.  

# Map,Set 和 WeakMap,WeakSet
## Set
### 概述
集合(Set)对象允许你存储任意类型的唯一值(不能重复)，无论是原始数据还是对象引用。

### 语法
> new Set([iterable]);  

#### 参数
iterable -- 一个可迭代对象，其中的所有元素都会被加入到Set中。null被视作undefined。

### 简述
Set对象是值的集合，你可以按照插入的顺序迭代它的元素。Set中的元素只会出现一次，即Set中的元素是唯一的。

#### 值的相等
因为Set中的值总是唯一的，所以需要判断两个值是否相等。判断相等的算法与严格相等(===操作符)不同。具体来说，对于Set, +0(+0严格相等于-0)和-0是不同的值。尽管在最新的ECMAScript6规范中这点已被更改。另外NaN和undefined都可被存储在Set中，NaN之间被视为相同的值(尽管NaN !== NaN)。

### 属性
Set.length -- length属性的值为0

Set.prototype -- 表示Set构造器的原型，允许向所有Set对象添加新的属性。

### Set实例
所有的Set实例继承自Set.prototype。示例：使用Set对象。
```javascript
  var mySet = new Set();

  mySet.add(1);
  mySet.add(5);
  mySet.add("Some text");

  mySet.has(1); //true
  mySet.has(3); //false,3没有被添加到set中
  mySet.has(5); //true
  mySet.has(Math.sqrt(25));  // true
  mySet.has("Some Text".toLowerCase()); // true

  mySet.size; // 3

  mySet.delete(5); // 从set中移除5
  mySet.has(5);    // false, 5已经被移除

  mySet.size; // 2, 我们刚刚移除了一个值
```

示例：迭代Set
```javascript
  //迭代整个set
  //按顺序输出：1, "some text"
  for(let item of mySet) console.log(item);

  //按顺序输出：1,,"some text"
  for(let item of mySet.keys()) console.log(item);

  //按顺序输出：1,"some text"
  for(let item of mySet.values()) console.log(item);

  //按顺序输出：1,"some text"(键与值相等)
  for(let[key,value] of mySet.entries()) console.log(key);

  //转换Set为Array(with Array coprehensions)
  var myArr = [v for (v of mySet)]; //[1,"some text"]
  //替代方案(with Array.from)
  var myArr = Array.from(mySet); // [1,"some text"]

  //如果在HTML文档中工作，也可以：
  mySet.add(document.body);
  mySet.has(document.querySelector("body")); //true

  //Set和Array转换
  mySet2 = new Set([1,2,3,4]);
  mySet2.size; //4
  [...mySet2]; //[1,2,3,4]

  //截取
  var intersection = new Set([x for(x of set1) if (set2 has(x))]);

  //用forEach迭代
  mySet.forEach(function(value){
    console.log(value);
  });
```

示例：和Array对象的关系
```javascript
  var myArray = ["value1", "value2", "value3"];

  //用Set构造器将Set转换为Array
  var mySet = new Set(myArray);

  mySet.has("value1"); //returns true

  //用...(展开操作符)操作符将Set转换为Array
  alert(uneval([...mySet])); //与myArray完全一致
```

## Map
### 概述
Map对象就是简单的键/值映射。其中键和值可以是任意值(原始值或对象值).

在判断两个值是否为同一个键的时候，使用的并不是===运算符，而是使用了一种称之为"same-value"的内部算法，该算法很特殊，对于Map对象来说,+0(按照以往的经验与-0是严格相等的)和-0是两个不同的键。而NaN在作为Map对象的键时和另外一个NaN是一个相同的键(尽管NaN !== NaN)

### API
| **Constructor** | **描述** |
| ------ | :-------: |
| new Map([iterable])	| 返回一个新的Map对象。如果参数iterable是一个数组或者其他可迭代的对象 -- 它的元素是键值对，这样这些每一个键值对都可以添加到新的Map里面去 |
| **方法** | **描述** |
| myMap.get(key) | 返回键key关联的值，如果该键不存在则返回undefined |
| myMap.set(key,value) | 设置键key在myMap中的值为value。返回undefined |
| myMap.has(key) | 返回一个布尔值，表明键key是否存在于myMap中 |
| myMap.delete(key) | 删除键key及对应的值.在这之后,myMap.has(key)将会返回false |
| myMap.entries() | 返回一个迭代器，迭代器按照对象的插入顺序返回[key,value]; |
| myMap.forEach(callbackFn[,thisArg]) | 循环执行函数并把键/值对作为参数;thisArg为可选的，如果有thisArg的话将会作为执行函数的上下文this; |
| myMap.keys() | 返回一个迭代器，迭代器按照Map实例的插入顺序返回每一个key元素; |
| myMap.clear() | 清空myMap中的所有键值对 |
| **属性** | **描述** |
| myMap.size | 返回myMap中键值对的数量。 |

### 例子
```javascript
  var myMap = new Map();

  var keyObj = {},
      keyFunc = function(){},
      keyString = "a string";

  //添加键
  myMap.set(keyString, "和键'a string'关联的值");
  myMap.set(keyObj, "和键keyObj关联的值");
  myMap.set(keyFunc, "和键keyFunc关联的值");

  myMap.size; //3

  //读取值
  myMap.get(keyString); // "和键'a string'关联的值"
  myMap.get(keyObj); // "和键keyObj关联的值"
  myMap.get(keyFunc); //"和键keyFunc关联的值"

  myMap.get("a string"); //"和键'a string'关联的值"
                         //因为keyString == "a string"
  myMap.get({});         //undefined,因为keyObj !== {}
  myMap.get(function(){})//undefined,因为keyFunc !== function(){}
```

NaN也可以作为Map对象的键。虽然NaN和任何值甚至和自己都不相等(NaN !== NaN返回true)，但下面的例子表明，两个NaN作为Map的键来说是没有区别的：
```javascript
  var myMap = new Map();
  myMap.set(NaN,"not a number");

  myMap.get(NaN); //"not a number"

  var otherNaN = Number("foo");
  myMap.get(otherNaN); //"not a number"
```

javascript有两个0值，+0和-0.虽然+0 === -0,但是当这两个0作为Map的键时，被认为是两个不同的值。
```javascript
  var myMap = new Map();
  myMap.set(0,"正零");
  myMap.set(-0,"负零");

  0 === -0; // true

  myMap.get(-0); // "负零"
  myMap.get(0);  // "正零"
```

### Object和Map的比较
Object和Map类似的一点是，它们都允许你按键存取一个值，都可以删除键，还可以检测一个键是否绑定了值。因此，一直以来，我们都把对象当成Map来使用，现在有了Map,下面的区别解释了为什么使用Map更好点。
 - 一个对象通常都有自己的原型，所以一个对象总有一个"prototype"键，不过，现在可以使用map = Object.create(null)来创建一个没有原型的对象。
 - 一个对象的键只能是字符串，但一个Map的键可以是任意值。
 - 你可以很容易的得到一个Map的键值对个数，而只能跟踪一个对象的键值对个数。

## WeakSet
### 概述
一个WeakSet对象是一个无序的集合，可以用它来存储任意的对象值，并且对这些对象值保持弱引用。

### 语法
> new WeakSet([iterable]);  

#### 参数
iterable -- 如果传入一个可迭代对象作为参数，则该对象的所有迭代值都会被自动添加进生成的WeakSet对象中。

### 描述
WeakSet对象是一些对象值的集合，并且其中的每个对象值都只能出现一次。

它和Set对象的区别有两点：
 - WeakSet对象中只能存放对象值，不能存放原始值，而Set对象都可以。
 - WeakSet对象中存储的对象值都是被弱引用的，如果没有其他的变量或属性引用这个对象值，则这个对象值会被当成垃圾回收掉。正因为这样，WeakSet对象是无法被枚举的，没有办法拿到它包含的所有元素。

### 属性
WeakSet.length -- length属性的值为0

WeakSet.prototype -- WeakSet实例的所有继承属性和继承方法都在该对象上。

### WeakSet实例
所有WeakSet实例都继承自WeakSet.prototype

属性 -- WeakSet.prototype.constructor(返回构造函数即WeakSet本身)。

方法
 - WeakSet.prototype.add(value) -- 在该 WeakSet 对象中添加一个新元素 value.
 - WeakSet.prototype.clear() -- 清空该 WeakSet 对象中的所有元素.
 - WeakSet.prototype.delete(value) -- 从该 WeakSet 对象中删除 value 这个元素, 之后 WeakSet.prototype.has(value) 方法便会返回 false.
 - WeakSet.prototype.has(value) -- 返回一个布尔值,  表示给定的值 value 是否存在于这个 WeakSet 中.

### 示例
```javascript
  var ws = new WeakSet();
  var obj = {};
  var foo = {};

  ws.add(window);
  ws.add(obj);

  ws.has(window); // true
  ws.has(foo);    // false, 对象 foo 并没有被添加进 ws 中

  ws.delete(window); // 从集合中删除 window 对象
  ws.has(window);    // false, window 对象已经被删除了

  ws.clear(); // 清空整个 WeakSet 对象
```

## WeakMap
### 概述
WeakMap对象就是简单的键/值映射。但键只能是对象值，不可以是原始值。

### API
| **方法** | **描述** |
| ------ | :-------: |
| myWeakMap.get(key [,defaultValue]) | 返回键key关联的值，如果该键不存在则返回默认值defaultValue |
| myWeakMap.set(key,value) | 设置键key在myWeakMap中的值，返回undefined |
| myWeakMap.has(key) | 返回一个布尔值来表明键key是否在myWeakMap中 |
| myWeakMap.delete(key) | 删除键key及对应的值。在这之后，myWeakMap.has(key)将返回false |
| myWeakMap.clear() | 清空myWeakMap中的所有的键值对，返回undefined |

### 例子
```javascript
  var wm1 = new WeakMap(),
      wm2 = new WeakMap(),
      wm3 = new WeakMap();
  var o1 = {},
      o2 = function(){},
      o3 = window;

  wm1.set(o1, 37);
  wm1.set(o2, "azerty");
  wm2.set(o1, o2); // value可以是任意值,包括一个对象
  wm2.set(o3, undefined);
  wm2.set(wm1, wm2); // 键和值可以是任意对象,甚至另外一个WeakMap对象
  wm1.get(o2); // "azerty"
  wm2.get(o2); // undefined,wm2中没有o2这个键
  wm2.get(o3); // undefined,值就是undefined

  wm1.has(o2); // true
  wm2.has(o2); // false
  wm2.has(o3); // true (即使值是undefined)

  wm3.set(o1, 37);
  wm3.get(o1); // 37
  wm3.clear();
  wm3.get(o1); // undefined,wm3已被清空
  wm1.has(o1);   // true
  wm1.delete(o1);
  wm1.has(o1);   // false  
```

### 为什么要使用WeakMap?
经验丰富的JavaScript程序员会注意到，WeakMap完全通过两个数组(一个存放键，一个存放值)来实现。但这样的实现会有两个很大的缺点，首先是O(n)的时间复杂度(n是键值对的个数)。另外一个则可能或导致内存泄漏；在这种自己实现的WeakMap中，存放键的数组中的每个索引将会保持对所引用对象的引用，阻止他们被当做垃圾回收。而在原生的WeakMap中，每个键对自己所引用对象的引用是“弱引用”，这意味着，如果没有其他引用和该键引用同一个对象，这个对象将会被当做垃圾回收。

正由于这样的弱引用, WeakMap 的keys是无法遍历的 (无法列举出所有的keys). 如果允许被遍历的话, 遍历的结果将会受垃圾回收的影响, 从而得到不确定的结果. 因此,如果你想得到所有keys的值,你应该自己管理他们. 另外ECMAScript提案中还介绍了另外两种集合类型,Map和Set,他们没有使用弱引用,所以是可遍历的.

# Proxy 和 Reflect
## 简介
Proxy对象用来为基础操作(例如：属性查找、赋值、枚举、方法调用等)定义用户自定义行为。

## 术语
handler -- 包含traps的对象。
traps -- 提供访问属性的途径，与操作系统中的traps定义相似
target -- 被代理虚拟化的对象，这个对象常常用作代理的存储后端。关于对象不可拓展性和不可修改属性的不变量会被代理拦截。

## 语法
> var p = new Proxy(target, handler);  

### 参数
target -- 目标对象，可以是任意类型的对象，比如数组，函数，甚至是另外一个代理对象。
handler -- 处理器对象，包含了一组代理方法，分别控制所生成代理对象的各种行为。

## 方法
Proxy.revocable() -- 创建一个可撤销的代理对象。

## handler对象的方法
handler是占位符对象，它包含代理的traps

## 示例
### 基础示例
以下简单的例子中，当对象不存在属性名时，缺省返回数37.例子中使用了get处理器(get handler)
```javascript
  var handler = {
    get: function(target,name){
      return name in target? target[name] : 37;
    }
  };

  var p = new Proxy({}, handler);
  p.a = 1;
  p.b = undefined;

  console.log(p.a, p.b); // 1, undefined
  console.log('c' in p, p.c); // false, 37
```

### 无操作转发代理
在以下例子中，我们使用了一个原生JavaScript对象，代理会将所有应用到它的操作转发到这个对象上。
```javascript
  var target = {};
  var p = new Proxy(target,{});

  p.a = 37; //被转发到代理的操作
  console.log(target.a); //37 操作已经被正确地转发。
```

### 验证
通过代理，你可以轻松地验证向一个对象的传值。以下例子使用了set处理器(set handler)。
```javascript
  let validator = {
    set: function(obj,prop,value){
      if(pop === 'age'){
        if(!Number.isInteger(value)){
          throw new TypeError('The age is not an integer');
        }
        if(value > 200){
          throw new RangeError('The age seems invalid');
        }
      }

      //The default behavior to store the value
      obj[prop] = value;
    }
  };

  let person = new Proxy({}, validator);

  person.age = 100;
  console.log(person.age); //100
  person.age = 'young'; //抛出异常
  person.age = 300; //抛出异常
```

### 扩展构造函数
方法代理可以轻松地通过一个新构造函数来扩展一个已有的构造函数。以下的例子中使用了constructor处理器(construct handler)和apply处理器(apply handler).
```javascript

```
https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy
