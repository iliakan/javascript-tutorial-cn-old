# JavaScript 简介

Let's see what's so special about JavaScript, what we can achieve with it and which other technologies play well with it.
我们一起来聊一下JavaScript，用它能做什么，它有哪些特性，已经跟它配合使用的一些技术。

## 什么是 JavaScript？

*JavaScript* 最初的目的是为了 *"让网页动起来"*。

这种编程语言我们称之为*脚本*。把它嵌入到HTML当中，在页面加载的时候会自动执行。

脚本作为纯文本存在和执行，并不需要编译执行。

这方面，JavaScript和[Java](http://en.wikipedia.org/wiki/Java)有很大的区别。

```smart header="Why <u>Java</u>Script?"
JavaScript在创建的时候，它的名字叫"LiveScript"。因为当时Java很流行，所以就取了个名字叫JavaScript。这样就可以让大家认为，JavaScript是Java的弟弟。

随时JavaScript的发展，它已经变成了一种独立的语言，同时也有了自己的语言规范[ECMAScript](http://en.wikipedia.org/wiki/ECMAScript)。现在，Java和JavaScript已经是两门不同的语言，彼此之前也没有任何关系。
```
现在，JavaScript不仅仅是在浏览器内执行，也可以在服务端执行。甚至在任意存在[JavaScript 引擎](https://en.wikipedia.org/wiki/JavaScript_engine)的环境中，都可以执行。

浏览器中嵌入了JavaScript引擎，有时也称作JavaScript虚拟机。
The browser has an embedded engine, sometimes it's also called a "JavaScript virtual machine".

不同的引擎有不同的名字，例如：

- [V8](https://en.wikipedia.org/wiki/V8_(JavaScript_engine)) --Chrome和Opera中的JavaScript引擎.
- [Gecko](https://en.wikipedia.org/wiki/Gecko_(software)) --Firefox中的JavaScript引擎。
- ...也有一些其他的JavaScript引擎，"Trident"和"Chakra"是不同版本IE的JavaScript引擎，"ChakraCore"是Microsoft Edge的JavaScript引擎, "Nitro"和"SquirrelFish"是Safari的JavaScript引擎，等等。

These terms above are good to remember, because they are used in developer articles in the internet. We'll use them too. For instance, if "a feature X is supported by V8", then it probably works in Chrome and Opera.
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

JavaScript的能力依赖于它执行的环境。比喻：[Node.JS](https://wikipedia.org/wiki/Node.js)就可以读写文件，可以发送响应网络请求。

浏览器中的JavaScript只处理和网页相关的操作，处理网页和用户的交互以及网页和服务端的网络请求。

浏览器中的JavaScript，可以干下面这些事：

- 在网页中插入新的HTML，修改现有的网页内容和网页的样式。
- 响应用户的行为，响应鼠标的点击或移动，键盘的敲击。
- 向远处服务器发送请求，下载或上传文件（[AJAX](https://en.wikipedia.org/wiki/Ajax_(programming))和[COMET](https://en.wikipedia.org/wiki/Comet_(programming))技术）。
- 获取或修改cookie，向用访问者发送消息，问问题。
- 存储浏览器端的一些本地数据（本地存储）。


## 浏览器中的JavaScript*不*能干什么?

JavaScript abilities in the browser are limited for the sake of the user's safety. The aim is to prevent an evil webpage from accessing private information or harming the user's data.

The examples of such restrictions are:

- JavaScript on the webpage may not read/write arbitrary files on the hard disk, copy them or execute programs. It has no direct access to OS system functions.

    Modern browsers allow it to work with files, but the access is limited and only provided if the user does certain actions, like "dropping" a file into a browser window or selecting it via an `<input>` tag.

    There are ways to interact with camera/microphone and other devices, but they require an explicit user's permission. So a JavaScript-enabled page may not sneakily enable a web-camera, observe the surroundings and send the information to the [NSA](https://en.wikipedia.org/wiki/National_Security_Agency).
- Different tabs/windows generally do not know about each other. Sometimes they do, for example when one window uses JavaScript to open the other one. But even in this case, JavaScript from one page may not access the other if they come from different sites (from a different domain, protocol or port).

    That is called a "Same Origin Policy". To workaround that, *both pages* must contain a special JavaScript code that handles data exchange.

    The limitation is again for user's safety. A page from `http://anysite.com` which a user has opened occasionaly must not be able to open or access another browser tab with the URL `http://gmail.com` and steal information from there.
- JavaScript can easily communicate over the net to the server where the current page came from. But its ability to receive data from other sites/domains is crippled. Though possible, it requires the explicit agreement (expressed in HTTP headers) from the remote side. Once again, that's safety limitations.

![](limitations.png)

Such limits do not exist if JavaScript is used outside of the browser, for example on a server. Modern browsers also allow installing plugin/extensions which may get extended permissions.

## What makes JavaScript unique?

There are at least *three* great things about JavaScript:

```compare
+ Full integration with HTML/CSS.
+ Simple things done simply.
+ Supported by all major browsers and enabled by default.
```

Combined, these 3 things only exist in JavaScript and no other browser technology.

That's what makes JavaScript unique. That's why it is the most widespread way of creating browser interfaces.

While planning to learn a new technology, it's beneficial to check its perspectives. So let's move on to the modern trends that include new languages and browser abilities.


## Languages "over" JavaScript

The syntax of JavaScript does not suit everyone's needs. Different people want different features.

That's normal, because projects and requirements are different for everyone.

So recently a plethora of new languages appeared, which are *transpiled* (converted) to JavaScript before they run in the browser.

The modern tools make the transpilation very fast and transparent, actually allowing developers to code in another language, autoconverting it "under the hood".

Examples of such languages:

- [CoffeeScript](http://coffeescript.org/) is a "syntax sugar" for JavaScript, it introduces shorter syntax, allowing to write more precise and clear code. Usually Ruby guys like it.
- [TypeScript](http://www.typescriptlang.org/) is concentrated on adding "strict data typing", to simplify development and support of complex systems. It is developed by Microsoft.
- [Dart](https://www.dartlang.org/) is a standalone language that has its own engine that runs in non-browser environments (like mobile apps). It was initially offered by Google as a replacement for JavaScript, but as of now, browsers require it to be transpiled to JavaScript just like the ones above.

There are more. Of course even if we use one of those languages, we should also know JavaScript, to really understand what we're doing.

## Summary

- JavaScript was initially created as a browser-only language, but now it is used in many other environments as well.
- At this moment, JavaScript has a unique position as a most widely adopted browser language with full integration with HTML/CSS.
- There are many languages that get "transpiled" to JavaScript and provide certain features. It is recommended to take a look at them, at least briefly, after mastering JavaScript.
