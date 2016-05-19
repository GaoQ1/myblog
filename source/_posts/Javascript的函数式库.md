---
title: Javascript的函数式库
date: 2016-05-19 11:44:41
tags:
- javascript
- 函数式编程
categories: 转载笔记
---
> A book that remains shut is but a block.

## Javascript的函数式库
据说所有的函数是程序员都会写自己的函数库，函数式Javascript程序员也不例外。随着如今开源代码分享平台如GitHab、Bower和NPM的涌现，对这些函数库进行分享、变得及补充变得越来越容易。 现在已经有很多Javascript的函数式变成库，从小巧的工具集到庞大的模块库都有。

每一个库都宣扬着自己的函数式编程风格。从一本正经的数学风格到灵活松散的非正式风格，每一个库都不尽相同， 然而他们他们有一个共同的特点：都是通过抽象的Javascript函数式能力来增进代码的重用行、可读性和健壮性。

### Underscore.js
Underscore在很多人眼里已经成为函数式Javascript库的标准。它成熟稳定， 其创建者Jeremy Ashkenas也是Backbone.js和Coffeescript的创建者。 Underscore实际上是对Ruby的Enumerable模块的重新实现， 这也解释了为什么Coffeescript也是受Ruby影响。

与jQuery相似，Underscore并不改变Javascript原生对象，而是用一个符号来定义自己的对象， 就是下划线（underscore）字符“_”。所以使用Underscore会是这个样子：
```javascript
   var x = _.map([1,2,3], Math.sqrt); // Underscore的map函数
   console.log(x.toString()); 
```

我们已经见过Javascript数组原生的map()方法，它是这样用的：
```javascript
   var x = [1,2,3].map(Math.sqrt); 
```

不同的是，用underscore时，数组对象和回调函数都是作为参数传入给underscore的map()方法（_.map）的， 而不是像数组原生的map()方法（Array.prototype.map）那样只需传递回调。

不过underscore除了map()还有很多内建函数，他们都是非常好用的函数， 比如find()、invoke()、pluck()、sortBy()、groupBy()等等。
```javascript
    var greetings = [{
      origin: 'spanish',
      value: 'hola'
    }, {
      origin: 'english',
      value: 'hello'
    }];
    console.log(_.pluck(greetings, 'value'));
    // 获取一个对象的属性.
    // 返回: ['hola', 'hello']
    console.log(_.find(greetings, function(s) {
      return s.origin ==
        'spanish';
    }));
    // 查找第一个回调函数返回真的元素
    // 返回: {origin: 'spanish', value: 'hola'}
    greetings = greetings.concat(_.object(['origin', 'value'], ['french', 'bonjour']));
    console.log(greetings);
    // _.object通过合并两个数组来建立一个对象
    // 返回: [{origin: 'spanish', value: 'hola'},
    //{origin: 'english', value: 'hello'},
    //{origin: 'french', value: 'bonjour'}]  
```

并且它还提供了链式调用方法
```javascript
    var g = _.chain(greetings)
      .sortBy(function(x) {
        return x.value.length
      })
      .pluck('origin')
      .map(function(x) {
        return x.charAt(0).toUpperCase() + x.slice(1)
      })
      .reduce(function(x, y) {
        return x + ' ' + y
      }, '')
      .value(); // 应用这些函数
    // 返回: 'Spanish English French'
    console.log(g);
```

> _.chain()方法的返回值被包在了一个拥有Underscore全部函数的对象里。_.value方法用于把被包裹的对象提取出来。 包裹的对象对于把Underscore混合到面向对象编程中非常有用。

Underscore也许并没有要追求函数式编程数学上的正确性，不过它也从来没有想要把Javascript扩展或者转变为一个纯函数语言。 它把自己定义为一个提供一大堆有用的函数式编程辅助函数的Javascript库。 也许它比那些伪造得看起来像函数式辅助函数的玩意儿要好些，不过它也不是一个严肃的函数式库。

那么有没有更好的库呢？一个建立在数学之上的库？

### Lazy.js
Lazy是一个实用的库，他更大程度上是沿着Underscore的路线，不过它有惰性求值策略。正因为如此，Lazy让即可解释的语言本不可能完成的函数式计算变成了可能。他还会显著提升性能。

Lazy的主意是，我们能够迭代的所有东西都是一个序列。由于这个库用方法执行的先后来控制顺序，很多很酷的事情就可以实现了：异步循环（并行编程）、无限序列、函数式响应式编程等等。

下面的例子展示了以下各种情形的代码：
```javascript
    //获得一首歌歌词的前三行
    var lyrics = "我徘徊在海之滨山之巅\n越此城镇越彼乡园\n ..."
    //如果没有惰性，整个歌词会先根据换行来分隔
    console.log(lyrics.split('\n').slice(0,3));
    //有了惰性，可以只文本分割出来前三行
    //歌词甚至可以无限长
    console.log(Lazy(lyrics).split('\n').take(3));
```

```javascript
   //前十个能被3整除的平方数
    var oneTo1000 = Lazy.range(1,1000).toArray();
    var sequence = Lazy(oneTo1000)
        .map(function(x){return x*x})
        .filter(function(x){return x % 3 === 0})
        .take(10)
        .each(function(x){console.log(x)});
```

```javascript
    //对无限序列的异步循环
    var asyncSequence = Lazy.generate(function(x){
        return x++
    }).async(100) //每两个元素间隔0.1秒
    .take(20) //只计算前20项
    .each(function(e){
        console.log(new Date().getMilliseconds() + ": " + e);
    });
```

不过Lazy库的这个主意并不能保证它完全的正确性。它还有一个前辈，Bacon.js，他们的工作方式差不多。

### 其他的一些库
Javascript函数式编程的库实在太多了，无法在本书中一一展示。我们再来简单看几个吧。

 - Functional
    - 这也许是Javascript的第一个函数式编程库，它包括了全面的高阶函数支持和string lambdas。
 - wu.js
    - 因其curryable()函数而饱受赞誉的wu.js库是一个很优秀的函数式编程库。它是第一个（据我所知） 实现了惰性求值的库，这影响了Bacon.js、Lazy.js等库。
    - 是的，它的名字来源于臭名昭著的摇滚组合“Wu-Tang Clan”
 - sloth.js
    - 和Lazy.js很像，但是更小
 - stream.js
    - 支持无限流，其它没什么
    - 特别小
 - Lo-Dash.js
    - 就像名字所暗示的那样，它是受underscore.js的启发
    - 高度优化
 - Sugar
    - Sugar是Javascript函数式编程技术的支持库，和Underscore相像，但是在实现上有一些关键的不同。
    - underscore中的 _.pluck(myObjs, 'value')在Suger中仅仅是myObjs.map('value')。 意思是他修改了Javascript原生的对象，所以它在跟其它库混用的时候会有些风险，比如Prototype。
 - from.js
    - 一个新的函数式库，Javascript的LINQ（语言集成查询）引擎，支持.net所提供的大多数LINQ函数。
    - 100%惰性求值，并支持lambda表达式
    - 很年轻，但是文档很出色
 - JSLINQ
    - 另一个Javascript的LINQ引擎
    - 比from.js更老也更成熟
 - Boiler.js
    - 另一个让Javascript扩展的函数式方法更加原生的工具库，包括：字符串、数字、对象、集合和数组
 - Folktale
    - 像Bilby.js那样，Folktable是一个对Fantasy Land实现的新库。并且像他的祖先那样， Folktable也是一个Javascript函数式编程库的集合。它还很年轻，但有光明的前景。
 - jQuery
    - 在这里看到jQuery很吃惊吗？尽管jQuery不是一个用于函数式编程的工具，但它自己是函数式的。 jQuery应该是根植于函数式编程的使用最广泛的库。
    - jQuery对象实际是一个monad。jQuery使用了monad的规则来实现方法链式调用： ```$('#mydiv').fadeIn().css('left': 50).alert('hi!');``` 
    - 它的一些函数是高阶的 ```$('li').css('left': function(index){return index*50});``` 
    - jQuery1.8以上的deferred.then实现了函数式概念Promise
    - jQuery是一个抽象层，主要是面向DOM。它不是一个框架或工具集， 只是一个使用抽象来提高代码复用和减少丑陋代码的方式。而函数式编程不全都是关于这些的吗？

## 开发和生产环境
### 环境
编程风格与应用所部属或者将要部署的环境没啥关系。但是库就有关系了。

### 浏览器
主要的Javascript应用还是跑在客户端的，也就是浏览器。基于浏览器的环境对于开发来说非常好， 因为浏览器无处不在，你可以在本地机器上写代码，解释器是浏览器的Javascript引擎， 所有的浏览器都有开发者终端。火狐的FireBug提供了非常有用的错误信息，并支持断点等等， 不过同样的代码运行在Chrome和Safari上的交叉引用的错误输出会很有用。甚至连IE都包含开发者工具。

浏览器的问题是他们对Javascript的解析不尽相同！尽管不是普遍现象，但是可能有些代码在不同的浏览器上会得到非常不同的结果。 不过主要的差别在于它们如何处理DOM，而不是在原型和函数如何工作上。举个明显的例子，Math.sqrt(4) 在任何浏览器和shell上都会返回2。但是scrollLeft方法依赖于浏览器的布局策略。

编写针对浏览器的特定代码是在浪费时间，这也是为何要使用库的另外一个原因。

### 服务器端Javascript
Node.js已经成为创建服务器端和基于网络应用的标准平台。函数式编程可以用于服务器端应用的编程吗？ 可以！OK，不过现在的这些函数式库是为这种注重性能的环境设计的吗？答案仍然是肯定的。

这章提到的所有的函数式库都可以工作在Node.js上，有些需要依赖browserify.js模块来处理浏览器元素。

### 服务器端环境的一个函数式用例
在我们的网络系统这个无畏的新世界里，服务器端应用开发人员总在担心并发问题，这是应该的。 经典的例子就是一个应用允许多个用户修改同一个文件，如果他们打算同时修改这个文件，你将会陷入令人作呕的混乱。 这就是困扰了程序员几十年的状态维持问题。

假设下面这个情景：
 1.一天早晨，亚当打开了一份报告开始编辑，但是出去吃午饭的时候没有保存。
 2.比利打开了同一份报告，添加了内容，并且保存了报告。
 3.亚当吃完午饭回来，又往这份报告里添加了内容，并且保存，不知情地覆盖了比利的内容。
 4.第二天，比利发现他的内容消失了。他的老板冲他咆哮，所有的人都一起冲着开发人员发飙，结果开发人员丢了工作。
 
长期以来，解决这个问题的办法就是给文件建立状态。当有人编辑这个文件的时候就把状态切换为加锁， 这就防止其他人编辑这个文件，并在保存这个文件后把状态切换为解锁。在我们的情景里，比利应该无法修改报告， 直到亚当吃完饭回来。并且只要文件没被保存就没有其他人可以编辑它。

这正是函数式编程关于不可变数据和状态的思想具有实际意义之处。函数式的实现并不是直接修改文件， 而是修改文件的一个拷贝，也就是一个新的版本。如果要保存这个版本而此时一个新的版本已经存在， 我们就知道已经有别人修改了原来的文件。危险规避了。

现在这个场景可以这样展开了：
 1.一天早晨，亚当打开了一份报告开始编辑，但是出去吃午饭的时候没有保存。
 2.比利打开了同一份报告，添加了内容，保存为了一个新的版本。
 3.亚当吃完饭回来继续添加内容，当他要保存时，系统告诉他现在已经存在一个新版本了。
 4.亚当打开了这个新版本，添加了自己的内容，并保存为另一个新版本。
 5.通过查看版本历史，老板看到了一切在平稳运行。所有人都很开心，应用的开发人员也得到了晋升和奖赏。
 
这个叫做事件源。不需要维护明确的状态，只需要事件。这个过程非常清晰，整个事件的历史都可以回顾。

这个思想以及其它一些优势是函数式编程在服务器端日益增长的原因。

## 第三章总结
你选择使用哪个数据库取决于你的需要是什么。需要函数响应式编程来处理事件和动态值？使用bacon.js。 需要无限流而不需要别的？用stream.js。想要一个函数式助手来补充jQuery？试试underscore.js。 需要严格特定多态的结构化环境？看看bilby.js。需要面面俱到的函数式编程工具？使用Lazy.js。 用这些都不爽？你自己写一个。

任何库都只擅长于它所使用的方式。尽管这章提到的库里面有几个缺点很少，大多数错误都会在不经意间就出现。 这取决于你选择的库是否正确，是否符合你的需求。