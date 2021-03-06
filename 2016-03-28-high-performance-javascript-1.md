---
layout: post
title: 《高性能 JavaScript》笔记（一）
category: 笔记
tags: 
- JavaScript
- 高性能 
- 设计模式 
- JS
date: 2015-03-22 13:01:29
---

# 第1章 加载和执行

1. 多数浏览器使用单一进程来处理用户界面（UI）刷新和 JavaScript 脚本执行，所以同一时刻只能做一件事。JavaScript 执行过程耗时越久，浏览器等待响应的时间就越长。
2. 由于脚本会阻塞页面其他资源的下载，因此推荐将所有的 `<script>` 标签尽可能放到 `<body>` 标签的底部，以尽量减少对整个页面下载的影响。
3. 减少页面包含的 `<script>` 标签数量有助于改善阻塞页面渲染这一情况。
4. 把一段内嵌脚本放在引用外链样式表的 `<link>` 标签之后会导致页面阻塞去等待样式表的下载，这样做是为了确保内嵌脚本在执行时能获得最精确的样式信息。建议永远不要把内嵌脚本紧跟在 `<link>` 标签之后。
5. 无阻塞脚本

	1. 延迟的脚本
		在 window 对象的 load 事件触发后再下载脚本。 采用 `defer` 属性。`defer` 与 `async` 的相同点是采用并行下载，在下载过程中不会产生阻塞。区别在于执行时机，`async` 是加载完成后自动执行，而 `defer` 需要等待页面完成后执行。

	2. 动态脚本元素
	
		用标准的 DOM 方法可以很容易地创建一个新的 `<script>`，文件在该元素被添加到页面时开始下载，并可通过侦听事件来获得脚本加载完成时的状态。
		
	3. XMLHttpRequest 脚本注入
	
		如果收到了有效响应，就创建一个带有内联脚本的 `<script>` 标签。这种方法的优点是可以下载 JavaScript 代码但不立即执行，以及在所有主流浏览器中无一例外都能工作。主要局限是同源策略。
		
# 第2章 数据存取

1. 计算机科学中有一个经典问题是通过改变数据的存储位置来获得最佳的读写性能。
2. JavaScript 中有四种基本的数据存储位置

	* 字面量
	* 本地变量
	* 数组元素
	* 对象成员
	
3. 作用域链对 JavaScript 有许多影响，从确定哪些变量可以被函数访问，到确定 this 的赋值。
4. 改变作用域链

	1. with 语句
	
		一个新的变量对象被创建，并被推入作用域链的首位，这意味着函数的所有局部变量现在处于第二个作用域链对象中，因此访问代价更高了。
		
	2. try-catch 语句
	
		catch 子句把异常对象推入一个变量对象并置于作用域的首位，在 catch 代码块内部，函数所有局部变量将会放在第二个作用域链对象中。
		
5. 闭包、作用域和内存

	通常来说，函数的活动对象会随着执行环境一同销毁，但引入闭包时，由于引用仍然存在于闭包的 [[Scope]] 属性中，因此激活对象无法被销毁。
	
6. 原型

	对象可以有两种成员类型：实例成员和原型成员。实例成员直接存在于对象实例中，原型成员则从对象原型继承而来。
	
7. 嵌套成员

	嵌套成员会导致 JavaScript 引擎搜索所有对象成员。对象成员嵌套得越深，读取速度就会越慢。执行 `location.href` 总是比 `window.location.href` 要快。
	
8. 缓存对象成员值

	若在函数中要多次读取同一个对象属性，最佳做法是将属性值保存到局部变量。局部变量能用来替代属性以避免多次查找带来的性能开销。

	但这种优化技术并不推荐用于对象的成员方法，因为许多对象方法使用 **this** 来判断执行环境，把一个对象方法保存在局部变量会导致 **this** 绑定到 **window**，而 **this** 值的改变会使得 JavaScript 引擎无法正确解析它的对象成员，继而导致程序错误。

# 第三章 DOM 编程

1. 浏览器中的 DOM

	尽管 DOM 是个与语言无关的 API，它在浏览器中的接口却是用 JavaScript 实现的。浏览器中通常会把 DOM 和 JavaScript 独立实现。两个相互独立的功能只要通过接口彼此连接，就会产生消耗。
	
2. innerHTML 对比 DOM 方法

	二者性能相差无几。
	
3. 节点克隆

	使用 DOM 方法更新页面内容的另一个途径是克隆已有元素，而不是创建新元素，即使用 `element.cloneNode()` 替代 `document.createElement()`。

4. HTML 集合

	类似于数组的列表，但不是真正的数组，没有 `push()`、`slice()` 之类的方法，但提供了一个类似于数组的 `length` 属性。
	
5. 选择器 API

	`querySelectorAll()` 返回一个 NoeList——包含着匹配节点的类数组对象。这个方法不会返回 HTML 集合，不会对应实时的文档结构。
	
6. 重绘和重排

	重排一定导致重绘，重绘不一定重排。
	
7. 减少重绘和重排次数

	1. 使元素脱离文档流
	2. 对其应用多重改变
	3. 把元素带回文档中
	
	3种基本方法可以使 DOM 脱离文档
	
		1. 隐藏元素
		2. 使用文档片段
		3. 拷贝原始元素到一个脱离文档的节点中，修改副本，再替换原始元素
		
8. 使用事件委托来减少事件处理器的数量	

# 第4章 算法和流程控制

1. 循环

	1. 4种循环的类型
	
		* 标准的 for 循环
		* while 循环
		* do-while 循环
		* for-in 循环
		
	2. 循环性能
	
		for-in 循环比其他几种明显要慢，因为每次迭代操作都会同时搜索实例和原型属性，故不要使用 for-in 循环来遍历数组成员。
		
	3. 提升循环性能
	
		* 减少迭代工作量；减少对象成员及数组项的查找次数，把值存储到局部变量中。
		* 颠倒数组顺序：倒序循环
		* 减少迭代次数："达夫设备（Duff's Device）"
		
			循环体展开技术，使得一次迭代中实际上执行了多次迭代的操作。
			
			典型实现如下：
			
				var iterations = Math.floor(items.length / 8);
				var startAt = items.length % 8;
				var i = 0;

				do {
					switch(startAt) {
						case 0: process(items[i++]);
					   	case 7: process(items[i++]);
					   	case 6: process(items[i++]);
					   	case 5: process(items[i++]);
					   	case 4: process(items[i++]);
					   	case 3: process(items[i++]);
    				   	case 2: process(items[i++]);
						case 1: process(items[i++]);
					}
					startAt = 0;
				} while(iterations--);
				
			一个稍快的版本取消了 `switch` 语句，并将余数处理和主循环分开。
			
				var count = items.length - 1;
				var startAt = items.length % 8;
				var iterations = Math.floor(items.length / 8);

				while(startAt--) {
					process(items[count--]);
				}

				while(iterations--) {
					process(items[count--]);
					process(items[count--]);
					process(items[count--]);
					process(items[count--]);
					process(items[count--]);
					process(items[count--]);
					process(items[count--]);
					process(items[count--]);
				}
				
	4. 基于函数的迭代
	
		`forEach()`同时接受三个参数，分别是**当前数组项的值、索引以及数组本身**。速度慢的主要原因是对每个数组项调用外部方法所带来的开销。
		
2. 条件语句

	1. 大多数情况下 `switch` 比 `if-else` 运行得更快

	2. 优化 if-else
	
		* 确保最有可能出现的条件放在首位。
		* 把 if-else 组织成一系列嵌套的 if-else 语句。
		* 查找表：不用书写任何条件判断语句，即便候选值数量增加，也几乎不会产生额外的性能开销。
		
3. 递归

	1. 调用栈限制
	2. 递归模式
	
4. 迭代

	任何递归能实现的算法同样可以用迭代来实现。使用优化后的循环替代长时间运行的递归函数可以提升性能。因为运行一个循环比反复调用一个函数的开销要小得多。
	
5. Memoization

	缓存前一个计算结果供后续计算使用。
	
		// 利用 Memoization 重写 factorial() 函数
		function memfactorial(n) {
			if (!memfactorial.cache) {
				memfactorial.cache = {
					"0", 1,
					"1", 1
				};
			}
			
			if (!memfactorial.cache.hasOwnProperty(n)) {
				memfactorial.cache[n] = n * memfactorial(n - 1);
			}
			
			return memfactorial.cache(n);
		}

# 第5章 字符串和正则表达式（略）

# 第6章 快速响应的用户界面

---

（最后修改于 2016-04-03）