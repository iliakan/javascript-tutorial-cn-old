# JavaScript 简介

我们一起来聊一下JavaScript，用它能做什么，它有哪些特性，以及一些跟它配合使用的技术。

## 什么是 JavaScript？

*JavaScript* 最初的目的是为了 *"让网页动起来"*。

这种编程语言我们称之为*脚本*。把它嵌入到HTML当中，在页面加载的时候会自动执行。

脚本作为纯文本存在和执行，并不需要编译执行。

这方面，JavaScript和[Java](http://en.wikipedia.org/wiki/Java)有很大的区别。

```smart header="Why <u>Java</u>Script?"
JavaScript在创建的时候，它的名字叫"LiveScript"。因为当时Java很流行，所以就取了个名字叫JavaScript。这样就可以让大家认为，JavaScript是Java的弟弟。

随着JavaScript的发展，它已经变成了一门独立的语言，同时也有了自己的语言规范[ECMAScript](http://en.wikipedia.org/wiki/ECMAScript)。现在，Java和JavaScript已经是两门不同的语言，彼此之前也没有任何关系。
```
现在，JavaScript不仅仅是在浏览器内执行，也可以在服务端执行。甚至在任意存在[JavaScript 引擎](https://en.wikipedia.org/wiki/JavaScript_engine)的环境中，都可以执行。

浏览器中嵌入了JavaScript引擎，有时也称作JavaScript虚拟机。

不同的引擎有不同的名字，例如：

- [V8](https://en.wikipedia.org/wiki/V8_(JavaScript_engine)) --Chrome和Opera中的JavaScript引擎.
- [Gecko](https://en.wikipedia.org/wiki/Gecko_(software)) --Firefox中的JavaScript引擎。
- ...也有一些其他的JavaScript引擎，"Trident"和"Chakra"是不同版本IE的JavaScript引擎，"ChakraCore"是Microsoft Edge的JavaScript引擎, "Nitro"和"SquirrelFish"是Safari的JavaScript引擎，等等。

上面这些经常在一些关于开发的文章中提到；也很方便记忆。比喻：某个新的功能，JavaScript引擎V8是支持的；那么我们可以认为这个功能在Chrome和Opera中可以正常运行。

```smart header="How the engines work?"
引擎很复杂，但是基本原理很简单。

1. 脚本是纯文本（可以被压缩）。
2. 引擎（通常嵌入在浏览器中）读取（理解）这些文本并转化（编译）成机器语言。
3. 然后就可以在机器上飞速的运行。

在每一个阶段，引擎都会做一些优化。引擎甚至会监视脚本的执行，分析数据流，从而采取相应的优化措施。
```

## 浏览器中的JavaScript能干什么?
现在的JavaScript是一种安全语言。它不会去操作计算机的内存和CPU。因为JavaScript最开始就是为浏览器准备的，浏览器也不需要操作这些。

JavaScript的能力依赖于它执行的环境。例如：[Node.JS](https://wikipedia.org/wiki/Node.js)就可以读写文件，可以发送响应网络请求。

浏览器中的JavaScript只处理和网页相关的操作，处理网页和用户的交互以及网页和服务端的网络请求。

浏览器中的JavaScript，可以干下面这些事：

- 在网页中插入新的HTML，修改现有的网页内容和网页的样式。
- 响应用户的行为，响应鼠标的点击或移动，键盘的敲击。
- 向远处服务器发送请求，下载或上传文件（[AJAX](https://en.wikipedia.org/wiki/Ajax_(programming))和[COMET](https://en.wikipedia.org/wiki/Comet_(programming))技术）。
- 获取或修改cookie，向用访问者发送消息，提问题。
- 存储浏览器端的一些本地数据（本地存储）。


## 浏览器中的JavaScript*不*能干什么?

为了用户的（信息）安全，在浏览器中的JavaScript的能力是有限的。这样主要是为了阻止邪恶的网站获得或修改用户的私人数据。

例如：

- 网页中的JavaScript不能读、写、复制及执行用户磁盘上的文件或程序。也不能直接控制操作系统。

    现代浏览器允许JavaScript做一些文件相关的操作，但是这个操作是受到限制的。仅当用户使用某个特定的动作，JavaScript才能操作这个文件。例如，把文件“拖”到浏览器中，或者通过`<input>` 标签选择文件。

    JavaScript有很多方式和设备的照相机/麦克风交互，这些都需要提前获得用户的允许。所以，JavaScript并不会偷偷的通过你的摄像头观察你，更不会把你的信息发送到[NSA](https://en.wikipedia.org/wiki/National_Security_Agency)。


- 不同的浏览器标签页基本彼此不相关。有时候，也会有一些关系。例如，通过JavaScript打开另外一个新的标签页。如果两个标签页打开的不是同一个网站，他们不能够相互通信（域名，协议或者端口任一不相同的网站，都认为是不同的网站）。

    这就是“同源策略”。为了解决不同标签页交互的问题，两个同源的网站必须*都*包含特殊的JavaScript代码，才能够实现数据交换。

    这个限制也是为了用户的信息安全。例如，来自`http://anysite.com`的网页的JavaScript不能够获取任何`http://gmail.com`（另外一个标签页打开的网页）页面的数据。
- JavaScript通过互联网可以很容易的和服务器通讯（当前网页域名的服务器）通讯。但是从其他的服务器中获取数据的功能是受限的，需要（在HTTP头中）添加某些参数。这也是为了用户的数据安全。

![](limitations.png)

Such limits do not exist if JavaScript is used outside of the browser, for example on a server. Modern browsers also allow installing plugin/extensions which may get extended permissions.
非浏览器中的JavaScript，一般没有这些限制。例如服务端的JavaScript就没有这些限制。现代浏览器还允许通过JavaScript来安装浏览器插件/扩展，当然这也是在用户授权的前提下。

## JavaScript为什么与众不同？

至少有*3*件事值得一提：

```compare
+ 和HTML/CSS完全的集成。
+ 使用简单的工具（语言）完成简单的任务。
+ 被所有的主流浏览器支持，并且默认开启。
```

满足这三条的浏览器技术也只有JavaScript了。

这就是为什么JavaScript与众不同！这也是为什么大家都通过JavaScript来跟浏览器交互。

当然，学习一项新技术的时候，最好先看一下他的前景。所有，接下来，我们来看看新的趋势（包含一些新的语言）。


## 比JavaScript“好”的语言

不同的人喜欢不同的功能，JavaScript的语法也不能够满足所有人。

这是正常的，因为每个人的项目和需求都不一样。

所以，最近出现了很多不同的语言，这些语言在浏览器中执行之前，都会被*编译*（转化）成JavaScript。

现代的工具编译得很快，并且让用户不可感知。这就允许开发中使用一种新的语言，就是使用JavaScript一样。

例如：

- [CoffeeScript](http://coffeescript.org/) 是JavaScript的语法糖, 他语法简短，精确简捷。通常使用Ruby的人喜欢用。
- [TypeScript](http://www.typescriptlang.org/) 主要是是添加了严格类型系统。这样就能简化开发，也能用于开发一些负责的系统。TypeScript是微软开发的。
- [Dart](https://www.dartlang.org/)是一门独立的语言。他拥有自己的引擎，在非浏览器环境中运行（如：在手机应用中运行）。最开始是Google提供的，用于替代JavaScript的，但是现在，浏览器页需要他和上面的语言一样需要被编译成JavaScript。

当然，还有更多其他的语言。即使我们在使用这些语言，我们也需要知道JavaScript。因为学习JavaScript可以让我们真正明白我们自己在做什么。

## 总结

- JavaScript最开始是为浏览器设计的一门语言，但是现在其他的环境中运行。
- 现在，JavaScript是在浏览中使用最广，并且能够很好集成HTML/CSS的一门语言。
- 有很多其他的语言可以编译成JavaScript，这些语言还提供更多的功能。最好要了解一下这些语言，只是在掌握JavaScript之后，需要了解一下。
