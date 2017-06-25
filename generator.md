---
title: Generator从陌生到熟悉
categories: ES6
toc: false
tags: 
- ES6
- generator
- 读书
---
### 关于Generator
1. 声明
```javascript
    function* Generator(){
        //code here
    }
    const Generator=function*(){
        //code here
    }
```
注意到那个`*`号

2. 生成器对象
generator不能像普通函数一样直接调用，调用会生成generator对象
```javascript
    const gen=Generator();
```
gen本身也是可以迭代的，可以用for-of 来迭代，在代码中我们已经试了，迭代出每次yield切出的值

3. 方法
* generator.next(value)
返回一个状态对象，大概像这样：
```json
    {
        value:any,
        done:Boolean
    }
```
value 是yeild切出的值，done是指generator完成的状态，一旦触发最后一个yield，状态done变为true，或者reurn语句或者抛出错误。
next方法还可以接受参数，作用是，有大神的[Generator基礎](http://huli.logdown.com/posts/292331-javascript-es6-generator-foundation)总结的：
>1. yield其實是兩個動作的合體：丟東西出去->等東西進來
>2. 每次next()都會跑到yield丟東西出來的那個步驟
<!--more-->
还是拿代码说明一下
```javascript
function *get_adder(){
  let total = 0;
  while(true){
    console.log("before yield");
    total+=yield total;
    console.log("after yield, total:"+total);
  }
}

var adder = get_adder();
console.log(adder.next().value);
console.log(adder.next(100).value);
//输出：
// before yield
// 0  在这里终端，切出value，yield开始等待值
// after yield ,total:100 第二个next ，传入100
// before yield
// 0 切出第二个value值
```
从上面代码可以看到
* generator.throw(error)

    抛出错误
* generator[@@literator] 

    暂时看不懂。

### 拓展：
 如何用for-of 生成迭代器