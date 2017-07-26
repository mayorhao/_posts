---
title: 【翻译】Angular组件通信
date: 2017-6-29 02:01:29
category: 翻译
tags: 
- infoQ
- 翻译
- Angular
- 组件
- 框架
---
# Angular组件通信
源文章:[Getting Components to Communicate in Angular](https://www.infoq.com/articles/angular-component-communication)
### 本文要点
* Angular应用是基于组件的，可以在组件之间传递数据。
* 可以通过`@Input()`向子组件传递数据。
* 可以通过`@Output()`捕获由子组件传来的数据。
* 可以使用像ngrx的外部数据存储来组织大型应用

组件是组成Angular应用的基石，在angular应用中，每个可见的元素都是由组件构成的。基于组件的架构的优势在于：很像JavaScript函数，如果一段代码变得过于复杂，或者承担过多的职责，那么你就可以把它拆分掉，这样每段代码都专注于一个功能。
也就是说，当我们着手于把组件拆分为更小的组件时，我们需要保证这些组件能够来回的传递数据。即在我们的应用中，合理的组件通信非常必要，通过它可以确保所有数据保持同步。幸运的是，Angular已经为我们提供了实现这个功能的工具。

![图片1](https://cdn.infoq.com/statics_s1_20170530-0600u1/resource/articles/angular-component-communication/en/resources/3Picture1.jpg)

<sup>我们可以在AppComponent下构建我们整个应用，但是该组件会承担过多的功能职责，在基于组件的架构中，公认更好的做法是：将组件拆分，这样每个组件只承担一个职责，<sub>

### 向组件传递数据
在Angular中，当父组件需要传递数据给子组件时，我们使用`@Input`，打个比方，我们正在开发一个应用程序来展示页面上的评论。 AppComponent会负责把评论数组加载进来，我们会把每条评论的数据分发给评论组件。
通过`@Input`,子组件会接收一个评论参数，以下是整个组件的代码：

```javascript
    @Component({
    selector: 'comment',
    templateUrl: './comment.component.html',
    styleUrls: ['./comment.component.css']
    })
    export class CommentComponent {
    @Input() comment;
    }
```
现在我们可以在别处调用该组件,并传递给它需要的评论数据，代码为：
```html
    <comment [comment]="comment"></comment>
```
<!--more-->
### 语法解析
首先，是组件选择器`<comment></comment>`,如果你之前使用过Angular，这个应该相当熟悉了。

第二，是属性绑定：`[comment]`，元素属性两边的方括号刚开始显得有点混乱，事实上，方括号并不是将数据从传递给组件所必需的，但是，如果没有它们，我们只能向组件的`@Input`传递普通的字符串。方括号告诉Angular这是一个属性绑定，并且传进来的值是动态的，它会将这个动态的数据插入和传入到组件。

最后，在属性绑定`comment`之后，我们就能够获取这个属性，这部分告诉Angular去查看`this.comment`属性，并将它传递给我们的评论组件。
### 概念梳理
要显示我们上图看到的评论列表，我们可以先结合一些Angular的概念，包括`*ngFor`.假设我们能够把评论数据作为评论组件类的一个属性，以`this.comment`的形式加载进来。
```html
    <comment
    *ngFor="let comment of comments"
    [comment]="comment">
    </comment>
```
最后的结果如下图所示：

![图片2](https://cdn.infoq.com/statics_s1_20170530-0600u1/resource/articles/angular-component-communication/en/resources/2Picture2.jpg)

<sub>使用评论组件展示的评论列表<sub>

### 捕获子组件事件
现在我们知道了如何向评论组件传递数据，但是要从列表中删掉一个组件，继而从列表中移除它时，我们该怎么做呢？这个做起来有些棘手，因为这个数据存在于评论组件的父组件，即AppComponent中。

一个解决问题的办法是，使用`@Output`,它可以让子组件触发事件，这个事件可以由父组件利用`EventEmitter`来捕获。

使得子组件和父组件通信的第一步，是通过`@OutPut`装饰器来为我们的组件新添一个类属性。
```javascript
    @Component({
    selector: 'comment',
    templateUrl: './comment.component.html'
    })
    export class CommentComponent {

    @Input() private comment;
    @Output() private onDelete = new EventEmitter();

    deleteComment() {
        this.onDelete.emit(this.comment);
    }
    }
```
在评论组件的模板里，我们可以调用`deleteComment()`方法。当点击删除按钮时，父组件便能够捕获这些事件。
```html
    <button (click)="deleteComment()">Delete</button>
```
### 捕获父组件事件
在AppComponent中，我们需要一个处理删除评论的问题的新方法。
```javascript
    @Component({
    selector: 'app-root',
    templateUrl: './app.component.html'
    })
    export class AppComponent {

    /**[code omitted]**/

    onCommentDelete(comment) {
        // logic to remove comment from comments array
    }

    }
```
在视图层面，我们只需要告诉Angular,当`onDelete`事件被触发时，调用`onCommentDelete()`方法。

```html
    <comment
      *ngFor="let comment of comments"
      [comment]="comment"
      (onDelete)="onCommentDelete($event)"></comment>
```
一旦我们的删除功能就绪，我们的应用就如下图所示:

![图3](https://cdn.infoq.com/statics_s1_20170530-0600u1/resource/articles/angular-component-communication/en/resources/Picture3.jpg)

<sub>现在，我们可以删除评论了，感谢Angular的`@Output()`装饰器<sub>

### 另一种获取数据的方式
到目前为止，我们只涉及到包括父/子层次结构的组件通信。虽然这可能满足你的大部分需求，但是随着应用程序规模的增长，继续通过父组件到子组件的模式来传递数据越来越难了。对于大型应用来说，使用“数据存储”(data store)来减少每个组件的工作量往往是有意义的。数据存储(data-stores)作为单个组件的中心数据仓库，当需要时可以被应用程序的某一部分调用。组件不再需要手动将数据传给子链(child-chain)，单个组件都可以订阅数据存储(data store)的一部分，以便只使用他们需要的数据，从而减少父组件和子组件来回传递数据的麻烦。

如果你熟悉React的话，这正是Redux在解决的问题。在Angular中，我们可以使用另一个库，名字叫[ngrx](https://github.com/ngrx)，它受到Redux的启发。两个库之间几乎没有什么关键性的区别：类型和观察对象。ngrx库很大程度依赖于TypeScript的生态系统，它相比于react，有着更多公式化的代码(boilerplate)，但同时也使得调试和跟踪bug更为简单。

当使用ngrx时，我们的状态(state)是一个不可变的数据结构。为了改变状态，我们可以调用名为actions的函数，反过来，这些actions告诉由纯净函数(pure function)组成的reducer，状态的哪一部分应该改变。最后，我们的应用程序将具有下一个版本的状态，所有订阅它的组件将立即收到新的数据。当组件接收到新数据时，它们也会自动重新渲染，使渲染的视图始终与数据库(store)保持同步。

这对开发意味着什么？

这是一个非常有意义的概念性问题：完全可能通过一个外部的数据源将数据传递给组件，而不是通过父-子组件的关系传递。这对开发到底意味着什么呢？

![图4](https://cdn.infoq.com/statics_s1_20170530-0600u1/resource/articles/angular-component-communication/en/resources/Picture4.jpg)

<sub>如果我们想在两个不同的地方显示评论数怎么办？依赖于组件层次实现起来会很难，这就是拥有一个使用ngrx的中心数据存储的好处<sub>

在大型应用中，向组件直接传递数据是非常有意义的，因为组件需要处理更多任务，并且增加额外的任务会更复杂，例如跟踪向子组件传递的数据。虽然没有必要把传递数据的重担转移到外部的数据存储(data-stores)，但是当你维护大型Angular应用时，保持清醒理智，是非常有意义的。

在上面的例子中，一个外部数据存储(store，未画出)正被使用，来传递有关多少个评论可用的信息，告诉侧边栏组件（SideBarComponent）“2个评论可用”，同时，将实际的评论数据传入评论列表组件(CommentList component)。这些数据完全绕过了父组件AppComponent。

### 快速回顾Angular组件通信

通过以上内容，我们基本了解了如何在Angular中进行组件通信，我们学习了如何通过`@Input()`装饰器来从父组件向子组件传递数据

我们同样也学习了如何使用`@Output()`装饰器和EventEmitter来将数据从子组件发回到父组件。

作为另一种选择，我们学习了在程序规模增长时，使用`ngrx`来减少基于父-子关系传递数据的麻烦，通过将数据保存在独立的数据存储(data-store)中，我们可以在组件中直接使用数据，而不再需要在多个子组件中传递数据，最终才到达目标组件。这使得维护大型应用更加简单。这些原则同样适用于像React这样的库，React的团队也通过相似的办法解决了同样的问题。

您可以在[这里](http://https//sergiocruz.github.io/ng-sample-comments/)看到我们的演示应用。而它的代码在[这里](https://github.com/sergiocruz/ng-sample-comments)可以看到。

### 如何继续学习

假如你对继续学习Angular非常有兴趣，请务必查看Code School的[通过Angular加速开发(Accelerating through Angular 2)](https://www.codeschool.com/courses/accelerating-through-angular-2)课程，以及我自己和Gregg Pollack的[通过组件交互和路由构建Angular2应用(build an Angular 2 app with component interaction & routing)](https://www.codeschool.com/courses/accelerating-through-angular-2)(Code School学员专享)。

另外，查阅[Angular组件交互文档(Angular's documentation on Component Interaction)](https://angular.io/docs/ts/latest/cookbook/component-communication.html)来进一步了解底层机制，以及最新的语法和最佳实践，毕竟Angular正在飞速的发展。

最后，你可以在官方ngrx github仓库通读[ngrx文档](https://angular.io/docs/ts/latest/cookbook/component-communication.html)，或者在[这里](https://github.com/sergiocruz/ng-sample-comments/tree/ngrx)查看我使用ngrx编写的示例应用。

### 关于作者
![图5](https://cdn.infoq.com/statics_s1_20170530-0600u1/resource/articles/angular-component-communication/en/resources/0F3A4898.jpg)

**Sergio Cruz**是Code School的程序开发人员和讲师，他专注于关于JavaScript的一切。最近，他教授了Code School的React课程，[(开启React之路(Powering up With React)](https://www.codeschool.com/courses/powering-up-with-react)”。当他不在Code School敲代码时，你会发现他会在ng-conf和OSCON这样的会议上作关于JavaScript的演讲。







