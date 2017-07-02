---
title: 什么是thunk函数？
categories: ES6
toc: false
tags: 
- ES6
- 异步
- javascript
---
### thunk函数是什么？
thunk函数起源于一个问题：函数的参数到底应该怎么求值（求值策略）？
比方说这样一段代码：
```javascript
let x=1;
function a(num){
    return num
}
//那么问题来了，*x+1*的值是什么时候计算的呢？
a(x+1)
```
一种流派是*传值调用*，即直接结算完x+5，再把值传给函数a,C语言就是这么干的
```
//相当于
f(2)
```
而另一种流派是*传名调用*,即直接把表达式x+5传入函数体。直到用到它的时候才求值。Hskell语言采用这种策略。
```
//相当于
f(x+1)
return (1+1)
```
这两种流派哪种比较好？
我比较倾向于传名调用。因为传值调用虽简单，但是很可能会造成性能浪费。想象：
```javascript
let x=1;
function f(a,b){
    return a
}
//调用
f(3*3*x-2*x-1,2)
```
好不容易计算完了参数a的数值，结果调用时根本就没有用到。这是对性能的极大浪费。
> 查阅资料后这里需要强调下。求值策略。在计算机科学里，并不仅限于函数的传参方式。它决定了变量之间、函数调用时实参和形参之间值是如何传递的。也分为按值传递与按引用传递。但在js中，有大神将其对象的传递方式归纳为按共享传递。该求值策略被用于JS，ruby，python，java等多种语言。该策略与引用传递最大的区别是，对函数形参的赋值，不会影响到实参的值。详细可查阅[共享传值](http://bosn.me/js/js-call-by-sharing/)
### thunk函数是什么？
而具体传名调用的实现，一般是通过一个临时的函数来把值计算出来，再传入到函数体中。而这个临时的函数，就叫做Thunk函数。
```javascript
function f(m){
    return m*2
}
f(x+5);
//加入thunk函数的版本
var thunk=function(){
    return x+5
}
//调用
function f(thunk){
    return thunk()*2
}
```
上面代码中，函数f的参数x+5被thunk函数给替换掉了，以后用到x+5，都直接调用thunk就好。
> 所以Thunk函数就是 “传名调用”的一种实现方式。用来替换传入函数的表达式。
### Javascipt Thunk函数。
我们知道javascript的参数是传值调用的。在JS中，Thunk函数替换的不是表达式，而是多参函数，将其替换成单参数版本，只接受回调参数作为参数。
比如：
```javascript
// 正常版本的readFile（多参数版本）
fs.readFile(fileName, callback);

// Thunk版本的readFile（单参数版本）
var readFileThunk = Thunk(fileName);
readFileThunk(callback);

var Thunk = function (fileName){
  return function (callback){
    return fs.readFile(fileName, callback); 
  };
};
```
老实说，跟函数curring有点像。多参数的函数变为单参数，这个单参数函数就是Thunk函数。
<font color=#0099ff  face="黑体">任何函数，只要接受回调函数作为参数，就可以写成一个Thunk函数的形式</font>
我们可以简单的写一Thunk函数转换器。
```javascript
let Thunk=function(fn){
    var args=Array.prototype.slice.call(arguments);
    return function(callback){
        args.push(callback);
        return fn.apply(this,args)
    }
}
```
老实说，除了Thunk函数最后是传入callback作为参数外，其它地方真的很想函数currying。
这样，我们就可以用它来生成Thunk函数了。
```javascript
let readFileThunk=Thunk(fs.readFile);
readFileThunk(fileA)(callback);
```
### Thunkify 模块
有一个可以转化函数为Thunk函数的npm包，叫[Thunkify模块](https://github.com/tj/node-thunkify)
使用npm 安装后，可以这样使用它
```javascript
let Thunkify=require("Thunkify");
let fs=require("fs");
let read =thunkify(fs.readFile);
read('package.json')((err,str)=>{
//do something
})
```
它的源码是这样的。
```javascript
function thunkify(fn){
  return function(){
    var args = new Array(arguments.length);
    var ctx = this;

    for(var i = 0; i < args.length; ++i) {
      args[i] = arguments[i];
    }

    return function(done){
      var called;

      args.push(function(){
    //保证callback只执行一次。
        if (called) return;
        called = true;
        done.apply(null, arguments);
      });

      try {
        fn.apply(ctx, args);
      } catch (err) {
        done(err);
      }
    }
  }
}
```
注意：thunkify只允许回调函数执行一次。

### Generator函数的流程管理。
为什么要用Thunk函数，有什么用？以前确实没什么用，但是有了generator之后，就有用了。Thunk现在可以用于Generator函数的自动流程管理。
比如说，还是读取文件，下面用Generator函数封装了两个异步操作：
```javascript
var 



