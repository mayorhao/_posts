---
title: JavaScript的函数柯里化
date: 2016-9-7 02:20:29
categories: JavaScript
toc: false
tags: 
- 函数式编程
- javaScript
- ES6
---
### 引子
对我来说，在一年前，柯里化还是一个陌生的名词。如今，函数式编程的概念深入人心，函数的柯里化这个名词更是我们避不开的热词。
但不可否认，它是一个很实用的技巧。甚至有国外大神在撰写关于柯里化的文章时，采用的标题借用了英国乐队 Badfinger 歌曲 Without You 中歌词：can't live if livin's without you。
好了，言归正传。

### 来源及概念
curry化的命名是为了纪念数学家 Haskell Curry（编程语言 Haskell也是以他的名字命名)。
它的书面概念是：
>柯里化通常也称部分求值，其含义是给函数分步传递参数，每次传递参数后部分应用参数，并返回一个更具体的函数接受剩下的参数，这中间可嵌套多层这样的接受部分参数函数，直至返回最后结果。 因此柯里化的过程是逐步传参，逐步缩小函数的适用范>围，逐步求解的过程。

说白了就是，柯里化是一种技巧，可以让本来接收多个参数的函数，分步接收参数。它的做法是：每次只接收部分参数，然后返回一个闭包接收剩下的参数。
####举个例子
```javascript
var concat3Words = function (a, b, c) {
    return a+b+c;
};

var concat3WordsCurrying = function(a) {
    return function (b) {
        return function (c) {
            return a+b+c;
        };
    };
};
console.log(concat3Words("foo ","bar ","baza"));            // foo bar baza
console.log(concat3WordsCurrying("foo "));                  // [Function]
console.log(concat3WordsCurrying("foo ")("bar ")("baza"));  // foo bar baza
```
这个例子展示了curry化如何通过返回闭包的形式分步接收参数。
好了。那，curry到底有什么作用

### 函数curry化的作用

#### 参数复用，设置默认参数。
打个比方吧，每天上班我一定需要做地铁，但是下了地铁之后是步行还是骑共享单车就不一定了，而共享单车也有三种选择，ofo，摩拜，优步。
我们可以通过代码来实现打印出我每天到底是怎么去上班的。
```javascript
    let curried=function(defaultway){
        return function(way){
            if(way!="walk"){
                way="骑"+way
            }
            console.log("我今天是先坐"+defaultway+"，然后"+way+"到公司的");
        }
    }
```
这样我们就可以默认先做默认工具(地铁),剩下的可以随意组合了.
```javascript
    let fn=curried("地铁");
    fn("ofo"); //我今天是先坐地铁，然后骑ofo到公司的
    fn("优步")；//我今天是先坐地铁，然后骑优步到公司的
```
通用一点的应用是，bind函数我们应该都很熟悉，一个简单的实现是：
```javascript
    function bind(fn,contex){
        return function(){
            return fn.apply(contex.arguments);
        }
    }
```
这样，我们可以传入默认参数
```javascript
    function bind(fn,contenx){
        var args = Array.prototype.slice.call(arguments,2);
        return function(){
            var innerArgs=Array.prototype.slice.call(arguments);
            var finnalArgs=args.concat(innerArgs);
            return fn.apply(context,finalArgs);
        }
    }
``` 

#### 提前返回，或者称之为惰性加载函数
很常见的一个例子，兼容现代浏览器以及IE浏览器的事件添加方法。我们正常情况可能会这样写：

```javascript
var addEvent = function(el, type, fn, capture) {
    if (window.addEventListener) {
        el.addEventListener(type, function(e) {
            fn.call(el, e);
        }, capture);
    } else if (window.attachEvent) {
        el.attachEvent("on" + type, function(e) {
            fn.call(el, e);
        });
    } 
};
```
上面的方法有什么问题呢？很显然，我们每次使用addEvent为元素添加事件的时候，(eg. IE6/IE7)都会走一遍```if...else if ...```,其实只要一次判定就可以了，怎么做？–柯里化。改为下面这样子的代码：
```javascript
var addEvent = (function(){
    if (window.addEventListener) {
        return function(el, sType, fn, capture) {
            el.addEventListener(sType, function(e) {
                fn.call(el, e);
            }, (capture));
        };
    } else if (window.attachEvent) {
        return function(el, sType, fn, capture) {
            el.attachEvent("on" + sType, function(e) {
                fn.call(el, e);
            });
        };
    }
})();
```
初始```addEvent```的执行其实值实现了部分的应用（只有一次的if...else if...判定），而剩余的参数应用都是其返回函数实现的，典型的柯里化。
 
#### 延迟计算，延迟执行

其实上文中的```bind（）```函数就是典型的延迟执行。那延迟计算的例子：我每周末都要去钓鱼，我想知道我12月份4个周末总共钓了几斤鱼，把一些所谓的模式、概念抛开，我们可能就会下面这样实现：
```javascript
var fishWeight = 0;
var addWeight = function(weight) {
    fishWeight += weight;
};

addWeight(2.3);
addWeight(6.5);
addWeight(1.2);
addWeight(2.5);

console.log(fishWeight);
```
柯里化的实现：

```javascript
    var curryWeight = function(fn) {
    var _fishWeight = [];
    return function() {
        if (arguments.length === 0) {
            return fn.apply(null, _fishWeight);
        } else {
            _fishWeight = _fishWeight.concat([].slice.call(arguments));
        }
    }
};
var fishWeight = 0;
var addWeight = curryWeight(function() {
    var i=0; len = arguments.length;
    for (i; i<len; i+=1) {
        fishWeight += arguments[i];
    }
});

addWeight(2.3);
addWeight(6.5);
addWeight(1.2);
addWeight(2.5);
addWeight();    //  这里才计算

console.log(fishWeight);    // 12.5
```
到最后才计算一次。

以上就是curry化的一些个人见解。还会继续补充。

