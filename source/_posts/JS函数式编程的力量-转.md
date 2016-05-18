---
title: JS函数式编程的力量(转)
date: 2016-05-18 09:29:02
tags:
- javascript
- 函数式编程
categories: 转载笔记
---
> 关于javascript函数式编程这一部分皆为转载外加自己的一些理解和实践。

## Javascript函数式编程的力量
几十年来，函数式编程一直是计算机科学狂热者的至爱，由于数学的纯洁性和谜一般的本质， 它被埋藏在计算机实验室，只有数据学家和有希望获得博士学位的人士使用。但是现在，它正经历一场复兴， 这要感谢一些现代语言比如Python，Julia，Ruby，Clojure以及——但不是最后一个——Javascript。

你是说Javascript？这个WEB脚本语言？没错！

Javascript已经被证明是一项长期以来都没有消失的重要的技术。这主要是由于它扩展的一些框架和库而使其具有重生的能力， 比如backbone.js，jQuery，Dojo，underscore.js等等。这与Javascript函数式编程语言的真实身份直接相关。 对Javascript函数式编程的理解很重要，并且在相当长的一段时间会对各种水平的程序员很有用。

为什么呢？函数式编程非常强大、健壮并且优雅。它对于大型数据结构非常有用并且高效。 Javascript作为一个客户端脚本语言，在应对日益复杂的网站时，函数式地操作DOM、 组织API响应以及完成一些其它任务会非常有好处。

在这本书里，你将会学习用Javascript进行函数式编程所需要知道的一切：如何用函数式编程构建你的Javascript web应用， 如何解锁Javascript隐藏的力量，如何编写更强大的代码，并且由于程序更小，使得代码更容易维护，能够更快被下载， 并且花费更少的开支。你还会学到函数式编程的核心概念，以及如何将它们应用到Javascript， 还有将Javascript作为函数式语言时如何回避一些问题，如何在Javascript中混合使用函数式编程和面向对象编程。

不过在我们开始前，先来做个实验。
    
### 例子
也许快速举个例子是介绍Javascript函数式编程最好的方式。我们将用Javascript完成一些任务—— 一个使用传统、原生的方法，另一个使用函数式编程。然后我们将会比较这两种方法。

### 应用--一个电子商务网站
为了追求真实感，我们来做一个电子商务网站，一个邮购咖啡豆的公司。这个网站会销售好几种类型的咖啡，有不同的品质，当然也有不同的价格。
    
### 命令式方法
首先，我们开始写程序。为了让这个例子更接地气，我们需要创建一些对象来保存数据。如果需要的话我们可以从数据库里取值。但是现在我们假设他们是静态定义的。
```javascript
    //create some objects to store the data
    var columbian = {
        name: 'columbian',
        basePrice: 5
    };
    var frenchRoast = {
        name: 'french Roast',
        basePrice: 8
    };
    var decaf = {
        name: 'decaf',
        basePrice: 6
    };
    //我们将使用辅助函数计算价格
    //根据size打印到一个HTML的列表中
    function printPrice(coffee,size){
        if(size == 'small'){
            var price = coffee.basePrice + 2;
        }else if(size == 'medium'){
            var price = coffee.basePrice + 4;
        }else{
            var price = coffee.basePrice + 6;
        }
        //create the new html list item
        var node = document.createElement("li");
        var label = coffee.name + ' ' + size;
        var textnode = document.createTextNode(label+ 'price: $'+ price);
        node.appendChild(textnode);
        document.getElementById('products').appendChild(node);
    }
    //现在我们只需要根据咖啡的各种价格和size的组合调用printPrice函数
    printPrice(columbian, 'small');
    printPrice(columbian, 'medium');
    printPrice(columbian, 'large');
    printPrice(frenchRoast, 'small');
    printPrice(frenchRoast, 'medium');
    printPrice(frenchRoast, 'large');
    printPrice(decaf, 'small');
    printPrice(decaf, 'medium');
    printPrice(decaf, 'large');
```
如你所见，这个代码非常基础。如果现在有更多的咖啡种类而不只是这三个改怎么办？如果有20个，甚至50个？ 如果有更多的size呢？如果有有机和无机之分呢？这将会很快将代码量变得巨大无比！

采用这种方法，我们让机器去打印每一种咖啡类型和每一个size。这就是采用这种命令式方法的基本问题。

### 函数式编程
命令式的代码一步一步地告诉电脑需要做什么，相反，函数式编程追求用数学方式来描述问题，其余的交给电脑来做。
通过更函数式一些的方法，同样的应用可以这样来写：
```javascript
    //从接口中分解数据和逻辑
    var printPrice = function(price,label){
        var node = document.createElement("li");
        var textnode = document.createTextNode(label+ ' price: $'+ price);
        node.appendChild(textnode);
        document.getElementById('products2').appendChild(node);
    }
    //为每种咖啡创建函数对象
    var columbian = function(){
        this.name = 'colimbian';
        this.basePrice = 5;
    };
    var frenchRoast = function(){
     this.name = 'french roast';
     this.basePrice = 8;
    };
    var decaf = function(){
     this.name = 'decaf';
     this.basePrice = 6;
    };
    //为每种size通过字面量创建对象
    var small = {
        getPrice: function(){return this.basePrice + 2},
        getLabel: function(){return this.name + ' small'}
    };
    var medium = {
      getPrice: function(){return this.basePrice + 4},
      getLabel: function(){return this.name + ' medium'}
    };
    var large = {
      getPrice: function(){return this.basePrice + 6},
      getLabel: function(){return this.name + ' large'}
    };
    //将所有咖啡的种类和size放到数组里
    var coffeeTypes = [columbian, frenchRoast, decaf];
    var coffeeSizes = [small, medium, large];
    //创建由上面内容组成的新对象，并把它们放到一个新数组里
    var coffee = coffeeTypes.reduce(function(previous,current){
        var newCoffee = coffeeSizes.map(function(mixin){
            //`plusmix`是函数时的mixin
            var newCoffeeObj = plusMixin(current,mixin);
            return new newCoffeeObj();
        });
        return previous.concat(newCoffee);
    },[]);
    //现在我们已经定义了如何获得所有咖啡种类和size组合方式的价格，现在可以直接打印他们了
    coffee.forEach(function(coffee){
        printPrice(coffee.getPrice(),coffee.getLabel());
    })
```

首先需要明确的是这个代码更加模块化了。现在新增一种size或者新增一个咖啡种类就像下面的代码这样简单：
```javascript
    var peruvian = function(){
        this.name = 'peruvian';
        this.basePrice = 11;
    }
    var extraLarge = {
        getPrice: function(){return this.basePrice + 10},
        getLabel: function(){return this.name + ' extra large'}
    };
    coffeeTypes.push(peruvian);
    coffeeSizes.push(extraLarge);
```

咖啡对象的数组和size对象的数组混合(mix)到了一起，也就是他们的方法和成员变量被组合到了一块--通过一个叫"plusMixin"的自定义函数。这些咖啡类型的类包含了成员变量，而这些size对象(small,medium,large)包含了获取名称和计算价格的方法。混合(mixing)这个动作通过一个map操作来起作用，也就是对数组中的每一个成员执行一个纯函数并返回一个新的函数， 然后这些返回的函数被放到了一个reduce函数中被操作，reduce也是一个高阶函数，和map有些像， 只是reduce把数组里的所有元素处理后组合到了一个东西里面。最终，新的数组包含了所有可能的种类和size的组合， 这个数组通过forEach方法遍历，forEach也是一个高阶函数，它会让数组里面每一个对象作为参数执行一遍回调函数。 在这个例子里，这个回调函数是一个匿名函数，它获取这些对象后，以对象的getPrice()和getLabel() 两个方法的返回值作为参数调用printPrice函数。

实际上，我们可以让这个例子更加函数式：去掉coffees变量，并将函数串到一起链式调用，这也是函数式编程的一个小技巧。
```javascript
    coffeeTypes.reduce(function(previous,current){
        var newCoffee = coffeeSizes.map(function(mixin){
            //`plusMixin`
            var newCoffeeObj = plusMixin(current,mixin);
            return new newCoffeeObj();
        });
        return previous.concat(newCoffee);
    }.[]).forEach(function(coffee){
        printPrice(coffee.getPrice(),coffee.getLabel());
    });
```
    
这样，控制流没有像命令是代码那样从头到尾的顺序进行。在函数式编程里，map函数和其他高阶函数代替了for和while循环，只有少量关键的代码是在顺序执行。这使得新接触的人在阅读这样范式的代码有些困难，但是一旦你能够欣赏它，你就会发现这根本没啥难的，而且这样写起来更好。
    
## 总结
首先，采用函数式风格的优点已经明确了。 其次，不要害怕函数式编程。的确，它往往被认为是编程语言的纯逻辑形式，但是我们不需要理解lambda演算也能够在日常任务中应用它。 实际上，通过把我们的程序拆分成小的片段，它们变得更容易被理解、维护，也更加可靠。 map和reduce函数是Javascript中不太被知道的内建函数，然而我们将要关注它们。