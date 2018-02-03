# JavaScript 简介

下面我们将一起来看看 `JavaScript` 能做什么，有哪些特性，以及跟它配合使用的一些技术。

## JavaScript 是什么

`JavaScript` 最初的目的是为了 `让网页动起来`。

这种编程语言我们称之为 `脚本`，把它嵌入 `HTML` 中，在页面加载时就会自动执行。

`脚本` 作为纯文本存在和执行，并不需要编译。

因此，`JavaScript` 和 [Java](https://zh.wikipedia.org/wiki/Java) 有很大的区别。

```md
最开始的时候，JavaScript 叫 “LiveScript”。当时 Java 很流行，所以就改名叫 JavaScript，这样，大家可能就误认为， JavaScript 是 Java 的弟弟。

随着 JavaScript 的发展，它成为了一门独立的语言，同时也有了自己的语言规范 ECMAScript。现在，Java 和 JavaScript 是两门不同的语言，彼此之间并没有任何关系。
```

现在，`JavaScript` 不仅可以在 `浏览器` 中执行，也可以在 `服务端` 执行，甚至在存在任意 [JavaScript 引擎](https://zh.wikipedia.org/wiki/V8_(JavaScript%E5%BC%95%E6%93%8E))的环境中执行。

浏览器d都嵌入了 `JavaScript 引擎`，有时也叫 `JavaScript 虚拟机`。

不同的引擎，名字也不同，例如：

* [V8](<https://en.wikipedia.org/wiki/V8>) -- Chrome 和 Opera 使用。
* [Gecko](<https://zh.wikipedia.org/wiki/Gecko>) -- Firefox 使用。
* ... 还有一些其他的 `JavaScript 引擎`，比如 `IE` 使用的是 `Trident` 和 `Chakra`，`Edge` 使用的是 `ChakraCore`，`Safari` 使用的是 `Nitro` 和 `SquirrelFish` 等。

上面这些也容易记忆，因为在开发文档中经常出现，例如：某个功能，支持 `V8`，即在 `Chrome` 和 `Opera` 中可以正常运行。

```
引擎比较复杂，但其基本原理很简单。

1. 引擎编译和执行脚本。
2. 将脚本转换为机器可以理解的语言。
3. 然后在机器上运行。

引擎还会监听整个脚本的执行，分析数据流等，尝试优化以上每一个阶段，最终达到整个过程十分流畅的目的
```

## 浏览器中的 JavaScript 能做什么

现在的 `JavaScript` 是一种安全的语言。它不会去操作计算机的内存和 CPU，因为 `JavaScript` 最开始是为浏览器而生的，浏览器并不需要。

`JavaScript` 的能力依赖它执行的环境。例如，[Node.JS](https://zh.wikipedia.org/wiki/Node.js) 支持读写文件，执行网
络请求。

浏览器中的 `JavaScript` 可以处理来自用户或服务器的任何操作或交互。

比如：

* 在网页中插入新的 `HTML`，修改网页的内容和样式。
* 响应用户的行为，比如鼠标的点击,移动和用户的按键。
* 向远程服务器发送请求，下载或上传文件（[AJAX](<https://zh.wikipedia.org/wiki/AJAX>)，[COMET](<https://en.wikipedia.org/wiki/Comet_(programming)>)）。
* 获取或修改 `cookie`。
* 在用户浏览器对数据j进行本地的存储（本地存储）。

## 浏览器中的 JavaScript 不能做什么

为了用户的安全，浏览器对 `JavaScript` 做了一定的限制，主要是为了阻止一些网站不法获得或修改用户的私人数据。

比如：

* 浏览器中的 `JavaScript` 不能读、写、复制及执行用户磁盘上的文件或程序，也不能直接控制操作系统。

	现代浏览器允许 `JavaScript` 做一些文件相关的操作，但这个操作是有限制的，比如仅当用户把文件 “拖” 到浏览器中，或者通过 `<input>` 标签来选择文件等。

	用户有很多场景需要使用照相机或麦克风，`JavaScript` 可以实现，但都需要提前获得用户的允许，所以，`JavaScript` 不会偷偷地通过你的摄像头观察你，更不会把你的信息发送到 [NSA](https://zh.wikipedia.org/wiki/%E7%BE%8E%E5%9B%BD%E5%9B%BD%E5%AE%B6%E5%AE%89%E5%85%A8%E5%B1%80)。

- 浏览器不同的标签页打开的网站彼此是不相关的，有时，也可能有关，简单讲，就是如果两个标签页打开的不是同一个网站，它们之间将不能相互通信（域名、协议或端口有一个不同，都认为是不同的网站）。

	这就是所谓的 `同源策略`，它限制了从一个源加载的文档或脚本如何与来自另一个源的资源进行交互。

	这也是为用户的安全考虑。比如，来自 `http://anysite.com` 网页的 `JavaScript` 不能获取任何来自 `http://gmail.com` 网页的数据。

- `JavaScript` 通过互联网可以很容易和来自当前页面的服务器进行通信，但要从其他服务器获取数据是需要条件的，比如需要（在 `HTTP` 头中）添加某些参数，这也是为用户的安全考虑。

![limitations](limitations.png)

非浏览器中的 `JavaScript` 并没有这些限制，比如服务端的 `JavaScript`，另外，现代浏览器还允许通过 `JavaScript` 来安装浏览器插件或扩展，当然这也是在用户授权的前提下。

## 为什么 JavaScript 与众不同

这里，至少有 3 点值得一提：

```
- 与 HTML/CSS 完美集成。
- 本身简单，做的事也简单。
- 主流浏览器都支持，并且默认开启。
```

满足这三点的浏览器技术也只有 `JavaScript` 了。

这就是为什么 `JavaScript` 与众不同，也是为什么大家都通过 `JavaScript` 来跟浏览器打交道。

当然，学习一门新技术，最好还是先看一下它的前景，所以，接下来，我们来看看它们的趋势。

## 比 JavaScript 更好的语言

每个人的需求不一样，即 `JavaScript` 并不能满足所有人

所以，目前出现了很多不同的语言，但这些语言在浏览器中执行之前，都会被编译成 `JavaScript`。

现代工具编译的速度是惊人的，可以让用户毫无察觉，这就允许开发者选择一门新的语言，并且做到最终的效果和 JavaScript 一样。

比如：

* [CoffeeScript](http://coffeescript.org/) `JavaScript` 的语法糖，增强了 `JavaScript` 的简洁性与可读性，通常，使用 `Ruby` 的人最为喜爱。
* [TypeScript](http://www.typescriptlang.org/) 微软开发的一种自由和开源的编程语言，设计目的是开发大型应用。
* [Dart](https://www.dartlang.org/) 一门独立的语言，拥有自己的引擎，在非浏览器环境中运行（如手机应用），最开始由 Google 提供用于替代 `JavaScript`，但现在，也需要和上面的语言一样在浏览器中编译成 `JavaScript` 。

当然，还存在其他的语言，但即使我们使用这些语言，也需要了解 `JavaScript`，因为它可以让我们明白最终我们到底都做了什么。

## 总结

* `JavaScript` 最初是为浏览器设计的一门语言，但现在也可以在其它的环境中运行。
* `JavaScript` 是在浏览中使用最广、并且能与 `HTML/CSS` 完美集成的一门语言。
* 存在很多其他的语言可以编译成 `JavaScript`，这些语言可能还提供更多的功能，我们可以适当了解下。
