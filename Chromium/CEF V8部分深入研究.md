#### 前言

由于要实现我司原有的JS扩展层（ModelJS）来支撑UI上的大量接口，而这些接口又没有定义，直接在浏览器里面声明全部报错，需要Native code来进行支持。所以，参照原来opera给出的方案，组里开始寻找替代品，最终找到了CEF来进行支持。

#### 总体结构

CEF类似于chromium content shell，但是它又大量封装了自己的一套方法，所以不是完全相同的。个人粗略认为content shell是比CEF更加纯净的一个APP。由于近来任务需要，对CEF的V8实现部分进行了较深入的研究。直接先上图：

![CEF V8 classes](../images/CEF-V8-class-diagram.png)

这张类图清晰地展示了一个general view，部分类方法实在太多了就没完整写进去。其实CEF做的东西并不复杂，大致的框架搞清楚后就可以剖析内部的具体内容了。CEF V8实现部分是对原有V8的东西进行了定制。接下来，要对一些用到的基本概念进行总结。

-----------------------------------------------------------------------------------------------------------------------------
#### 基本概念

##### Isolate

一个Isolate可以理解为是一个V8 instance。在blink中，isolate和thread是1：1的关系。一个isolate有一个main thread和一个worker thread。具体的内部代码实现并没有深究，有一张图例可以展示isolate的大致概念：

![V8_Isolate](../images/worlds.png)

##### Context

在V8中，Context是一个全局变量作用域(global variable scope)的概念。简要的说，一个window object对应一个context。比如在浏览器中，<iframe>标签可以代表着一个window object，而该window object的作用域与父frame并不一致。
这是因为他们具有不同的contexts，各个contexts又具有不同的global variable scope和prototype chain。从而parent frame和current frame相互隔离。举例官网的例子说明：

```
// main.html
<html><body>
<iframe src="iframe.html"></iframe>
<script>
var foo = 1234;
String.prototype.substr =
    function (position, length) { // Hijacks String.prototype.substr
        console.log(length);
        return "hijacked";
    };
</script>
</body></html>

// iframe.html
<script>
console.log(foo);  // undefined
var bar = "aaaa".substr(0, 2);  // Nothing is logged.
console.log(bar);  // "aa"
</script>
```
这段例子清晰展示了不同context之间的隔离关系。

##### Entered context and current context

由于在一个isolate中，可能会具有多个frames，而每个frame具有自己的context，简要的说，isolate和context的关系是一对多。所以不同的context就会有相互进入(Enter)的情况。
要理解entered context和current context的区别，就要了解两个runtime stacks。
第一个stack是JS functions stack。这个栈是由V8来统一管理，当一个function调用另一个function时候，被调用（callee）的function会被压入栈中。当函数返回时候，函数从栈中弹出，并返回调用函数，而此时的调用函数是在栈顶。
这里function都有各自的context，我们称当前运行的函数的context为current context。例如官网给出的例子：
```
// main.html
<html><body>
<iframe src="iframe.html"></iframe>
<script>
var iframe = document.querySelector("iframe");
iframe.onload = function () {
    iframe.contentWindow.func();
}
</script>
</body></html>

// iframe.html
<script>
function func() {
  ...;
}
</script>
```
上述例子中，func()在运行时候，current context就是<iframe> context。

第二个stack由V8 binding来调度。我们可以成为context stack。当V8 binding调用JS时候，V8 binding进入一个context并将当前context压入栈顶，然后JS就在当前context下运行了。如果当前JS执行完后，控制权交还给V8 binding，并将栈顶弹出。
栈的push和pop操作是由以context为传入参数的V8 API完成，也可显示调用8::Context::Enter()和v8::Context::Exit()。我们将之前进入的context为entered context。以上述程序为例，当func()运行时候，entered context是main frame的context（而不是<iframe>）。
这就好比一个链表，entered context是指向当前current context的前一个链表节点。

还有个特殊的context叫debugger context，在这里就不多介绍了。

##### World

一个world，个人认为相当于一个容器，好像tomcat那种概念。在上面的图例中可以看到world的概念。对于world一共有三种形式：main world, isolated world, worker world。
Main world是从web获取的JS的执行容器，isolated world是chrome扩展部分的content script的执行容器。main world和isolated world是一对多的关系。而worker world是在worker thread中的。
所有的worlds可以共享C++ DOM objects，但是每个world需要有自己的DOM wrappers。需要注意的是，每个world也具有自己的context。这意味着每个world也有自己的global variable scope和prototype chain。
在这种架构下，每个world内的JS变量是不可以相互共享的。这是出于安全考虑。最简单的一个例子，比如chrome extension可以在共享的标准DOM结构下在沙箱里运行独自的untrusted JS code，就是基于这种设计。

##### isolate, context, world, frame之间的关系

1. 从DOM角度来讲，一个HTML网页有N个frames，每个frame具有自己的context。
2. 从JS的角度讲，一个isolate有M个worlds，每个world有自己的context。

那么总结下，当main thread执行时候，就有N个frames和M个worlds参与进来，总共有N*M个contexts。见下图

![contexts](../images/contexts.png)

也就是说main thread只允许一次存在一个current context，在其生命周期中有N*M个contexts被创建。

##### DOM wrapper和context联系

当一个DOM wrapper被创建，就需要为其指定一个适合的context。如果DOM wrapper在一个错误的context中创建，那么会导致JS object leaking，从而产生安全问题。

---

#### CEF app创建一个extension的时序

![extension sequence](../images/SequenceDiagram.PNG)