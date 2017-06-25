---
layout: post
title: jQuery学习笔记（1）
category: 笔记
tags: 
- jQuery
date: 2015-01-22 13:01:29


---

	当我在回顾这篇笔记的时候，我记不清楚我是从哪里归纳总结来的，没有提到教程地址，没有第二篇...

### 1.常见DOM事件

1. 鼠标事件：click、dblclick、mouseenter、mouseleave
2. 键盘事件：keypress、keydown、keyup
3. 表单事件：submit、change、focus、blur
4. 文档/窗口时间：load、resize、scoll、unload

### 2.获取内容和属性

1. 获得内容

	- text()——设置或返回所选元素的文本内容
	- html()——设置或返回所选元素的内容（包括HTML标记）
	- val()——设置或返回表单字段的值

2. 获取属性-attr()

### 3.设置内容和属性

1. 设置内容

	- text()——设置或返回所选元素的文本内容
	- html()——设置或返回所选元素的内容（包括HTML标记）
	- val()——设置或返回表单字段的值

2. text()、html()以及val()的回调函数  

	回调函数由两个参数：被选元素列表中当前元素的下标，以及原始（旧的）值。然后以函数新值返回您希望使用的字符串

3. 设置属性-attr()

4. attr()的回调函数  

	回调函数由两个参数：被选元素列表中当前元素的下标，以及原始（旧的）值。然后以函数新值返回您希望使用的字符串。

### 4. 添加元素

1. 添加新的HTML内容

	- append() - 在被选元素的结尾插入内容
	- prepend() - 在被选元素的开头插入内容
	- after() - 在被选元素之后插入内容
	- before() - 在被选元素之前插入内容

### 5. 删除元素

1. 删除元素/内容

	- remove()-删除被选元素（及其子元素）
	- empty()-从被选元素中删除子元素

2. 过滤被删除的元素  

	remove()方法也可接受一个参数，允许您对被删元素进行过滤。empty()则没有对应的方法
	
---

（完）

（最后修改于2015-09-23）