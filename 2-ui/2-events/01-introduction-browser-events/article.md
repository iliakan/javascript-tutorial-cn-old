# 浏览器事件入门

*事件*是当某个交互产生时的信号。所有的 DOM 节点都会产生这样的信号（但事件的产生并不限于 DOM 节点）。

[cut]

以下是最常用的 DOM 事件的列表，让我们先来看看：

**鼠标事件：**
- `click` - 当鼠标点击某个元素时（在触屏设备上，tap 事件会触发 click 事件）。
- `contextmenu` - 当鼠标右键单击某个元素时。
- `mouseover` - 当鼠标光标到悬停在某个元素上时。
- `mousedown` 和 `mouseup` - 当鼠标按键在某个元素上按下和松开时。
- `mousemove` - 鼠标移动时。

**表单事件：**
- `submit` - 当网页用户提交表单`<form>`时。
- `focus` - 当用户选中某个元素，使其获得焦点时，例如选中输入框`<input>`。

**键盘事件：**
- `keydown` 和 `keyup` - 当用户按下然后松开键盘按键时。

**文档事件**
- `DOMContentLoaded` - 当 HTML 页面被加载和处理时，DOM 树被完全创建时。

** CSS事件：**
- `transitionend` - 当 CSS 动画结束时。

此外，还有很多其他种类的事件。

## 事件处理器(Event handler)

为了对事件做出反应，我们可以给事件绑定一个*处理器* - 这是一个函数，会在事件触发的时候被执行。

当用户和页面进行交互时，事件处理器会根据交互行为执行 JavaScript 代码。

绑定事件处理器有多种方法。让我们先从一个简单的例子开始。

### HTML属性(HTML-attribute)

事件处理函数可以设置在 HTML 元素的属性中，属性名称为`on <event>`。

例如，给`input`元素绑定一个`click`的事件处理器，我们可以使用`onclick`，如下所示：

```html run
<input value="Click me" *!*onclick="alert('Click!')"*/!* type="button">
```

在鼠标点击按钮后，会执行`onclick`中的代码。

注意，在`onclick`的函数参数中，我们用的是单引号，因为属性绑定时用的是双引号。如果我们忘记了代码是在属性内部，然后使用了双引号，就像这样：`onclick="alert("Click!")"`，程序就会报错。

不推荐在 HTML 属性中编写大量代码，我们最好创建一个 JavaScript 函数，然后调用这个函数。

例如，点击时执行函数`countRabbits()`:

```html autorun height = 50
<script>
  function countRabbits() {
    for(let i=1; i<=3; i++) {
      alert("Rabbit number " + i);
    }
  }
</script>

<input type="button" *!*onclick="countRabbits()"*/!* value="Count rabbits!">
```

正如我们所知道的，HTML 属性名称不区分大小写，因此`ONCLICK`, `onClick`和`onCLICK`都是一样的。但通常属性名都使用小写：`onclick`。

### DOM 属性

我们可以使用 DOM 属性`on <event>`来绑定一个事件处理器。

例如`elem.onclick`：

```html autorun
<input id="elem" type="button" value="Click me">
<script>
*!*
  elem.onclick = function() {
    alert('Thank you');
  };
*/!*
</script>
```

在使用 HTML 属性绑定事件处理器时，浏览器会读取 HTML 属性，然后根据属性内容创建一个新函数，并将这个属性绑定至 DOM 属性。

所以这个绑定方法其实和前一个方法是一样的。

**事件处理器始终被绑定在 DOM 属性中：HTML 属性只是初始化它的一种方法。**

以下的两个代码片段的工作原理是相同的：

1. 只用 HTML：

    ```html autorun height=50
    <input type="button" *!*onclick="alert('Click!')"*/!* value="Button">
    ```
2. HTML + JS：

    ```html autorun height=50
    <input type="button" id="button" value="Button">
    <script>
    *!*
      button.onclick = function() {
        alert('Click!');
      };
    */!*
    </script>
    ```

**由于一个 DOM 元素中只有一个`onclick`属性，对于一个 DOM 元素，不能绑定多个事件处理程序。**

例如下面的代码，用 JavaScript 绑定事件处理器时，会重写已经绑定了的处理器：

```html run height=50 autorun
<input type="button" id="elem" onclick="alert('Before')" value="Click me">
<script>
*!*
  elem.onclick = function() { // overwrites the existing handler
    alert('After'); // only this will be shown
  };
*/!*
</script>
```

顺便说一下，我们可以直接将一个现有的函数绑定到一个事件处理器：

```js
function sayThanks() {
  alert('Thanks!');
}

elem.onclick = sayThanks;
```

要删除一个事件处理器 - 将处理器赋值为空`elem.onclick = null`。

## 通过 this 访问元素

当事件处理器绑定在某个元素上，在处理函数中`this`的值就表示被绑定的元素。

在下面的代码中，点击`button`按钮时，通过调用`this.innerHTML`，显示按钮内的文本内容：

```html height=50 autorun
<button onclick="alert(this.innerHTML)">Click me</button>
```

## 容易犯的一些错误

如果你开始使用事件处理器 - 请注意一些细节。

**下面所示的例子中，应该使用`sayThanks`对函数赋值，而不是`sayThanks()`。**

```js
// 正确
button.onclick = sayThanks;

// 错误
button.onclick = sayThanks();
```

如果我们添加了括号，那么`sayThanks()`将是函数执行后的*返回值*，于是错误代码中的`onclick`会被赋值为`undefined`（因为该函数不返回任何东西）。按钮点击之后也不会有任何反应。

...但是在HTML元素标记中，我们需要使用括号：

```html
<input type="button" id="button" onclick="sayThanks()">
```

两种方式的区别可以这样理解。当浏览器读取元素中的属性时，它会用属性值创建一个新的处理函数。

所以上一个例子就可以等同于：
```js
button.onclick = function() {
*!*
  sayThanks(); // the attribute content
*/!*
};
```

**使用函数，而不是用字符串。**

这样的赋值`elem.onclick = "alert(1)"`也可以被执行。但由于浏览器兼容性的原因，但不推荐这样使用。

**不要对事件处理器使用`setAttribute`。**

例如下面的例子就不能绑定点击事件：

```js run no-beautify
// 点击 <body> 会产生错误，
// 因为属性总应该是字符串，如果强行将函数赋值为字符串，就会报错
document.body.setAttribute('onclick', function() { alert(1) });
```

** DOM 属性事件。**

为`elem.onclick`绑定一个事件处理器，注意不是`elem.ONCLICK`，因为 DOM 属性是区分大小写的。

## addEventListener

用这种方式绑定事件处理器时，有一个根本问题 - 我们不能对一个事件绑定多个处理器。

例如，点击某个按钮时，我们想要高亮显示这个按钮，同时又想要显示一条消息。

我们对这个按钮绑定两个事件处理器。但是一个对 DOM 属性赋值时，新的赋值将覆盖现有的 DOM 属性：

```js no-beautify
input.onclick = function() { alert(1); }
// ...
input.onclick = function() { alert(2); } // 重写上个处理函数
```

网络标准的开发人员们很久以前就意识到这一点，并提出了使用另一种替代方法`addEventListener`和`removeEventListener`来管理事件处理器，来避免上述的问题。

添加事件处理的语法：

```js
element.addEventListener(event, handler[, phase]);
```

`event`
: 事件名称, 例如`"click"`

`handler`
: 事件处理函数

`phase`
：可选参数，表示在某个“阶段”触发事件处理器。这个会在后文提到。通常我们不用这个参数。

用`removeEventListener`来删除某个事件处理器：


```js
// 参数与addEventListener完全相同
element.removeEventListener(event, handler[, phase]);
```

````warn header="Removal requires the same function"
要删除某个事件处理时，我们应该传递在和添加时传递的完全一样的函数。

下面的例子就不起效：

```js no-beautify
elem.addEventListener( "click" , () => alert('Thanks!'));
// ....
elem.removeEventListener( "click", () => alert('Thanks!'));
```

事件处理器并不会被删除，因为`removeEventListener`得到的是另一个函数 - 只不过函数使用了相同的代码。

以下是正确的方法：

```js
function handler() {
  alert( 'Thanks!' );
}

input.addEventListener("click", handler);
// ....
input.removeEventListener("click", handler);
```

请注意 - 如果我们不把函数存储在某个变量中，那么之后我们无法将其删除。因为没有办法“找回” 传递给“addEventListener”的处理函数。
````

对`addEventListener`多次调用，便可以添加多个事件处理，如下所示：

```html run no-beautify
<input id="elem" type="button" value="Click me"/>

<script>
  function handler1() {
    alert('Thanks!');
  };

  function handler2() {
    alert('Thanks again!');
  }

*!*
  elem.onclick = () => alert("Hello");
  elem.addEventListener("click", handler1); // Thanks!
  elem.addEventListener("click", handler2); // Thanks again!
*/!*
</script>
```

通过上面的例子，我们可以看到，DOM属性和`addEventListener`都可以来设置事件处理器*。但一般来说，我们只使用其中一种。

````warn header="For some events handlers only work with `addEventListener`"
还有一些无法通过DOM属性来绑定的事件。这些事件必须通过`addEventListener`来绑定。

例如，事件`transitionend`（CSS动画完成时调用）就是这样的。

试着执行下面的代码。在大多数浏览器中，只有第二段代码会正常执行，而第一端不能。

```html运行
<style>
  input {
    transition: width 1s;
    width: 100px;
  }

  .wide {
    width: 300px;
  }
</style>

<input type="button" id="elem" onclick="this.classList.toggle('wide')" value="Click me">

<script>
  elem.ontransitionend = function() {
    alert("DOM property"); // doesn't work
  };

*!*
  elem.addEventListener("transitionend", function() {
    alert("addEventListener"); // shows up when the animation finishes
  });
*/!*
</script>
```
````


## 事件对象

要正确的处理事件，我们需要更多地了解事件中产生的细节。不仅仅只是“点击”或“按键”，而是鼠标指针点击的坐标在哪里？键盘按下了哪个按键？等等

当事件发生时，浏览器创建一个*事件对象*，将细节放在其中，并将其作为参数传递给事件处理器。

以下是从事件对象中获取鼠标坐标的示例：

```html run
<input type="button" value="Click me" id="elem">

<script>
  elem.onclick = function(*!*event*/!*) {
    // show event type, element and coordinates of the click
    alert(event.type + " at " + event.currentTarget);
    alert(event.clientX + ":" + event.clientY);
  };
</script>
```

`event`对象的一些属性：

`event.type`
：事件类型，这里是`"click"`.。

`event.currentTarget`
：处理事件的元素。这里与`this`完全相同，除非你将这个处理函数`bind'绑定到别的对象，这时`event.currentTarget`可以和 this 区分开用。

`event.clientX / event.clientY`
：光标的窗口相对坐标，用于鼠标事件。

还有很多的属性。它们依赖于事件类型，所以我们稍后在详细介绍不同的事件时，再来学习。

````smart header="The event object is also accessible from HTML"
如果我们在HTML中绑定一个事件处理器，我们也可以使用`event`对象，像这样：

```html autorun height=60
<input type="button" onclick="*!*alert(event.type)*/!*" value="Event type">
```

这是可行的，因为当浏览器读取属性时，它会创建一个这样的处理函数：`function(event) { alert(event.type) }`。这个处理函数的第一个参数是`"event"`，而函数体则是 onclick 的属性值。
````

## 总结

有3种方式来绑定事件处理器：

1. HTML 属性：`onclick="..."`
2. DOM 属性：`elem.onclick = function`
3. 通过调用方法添加`elem.addEventListener(event, handler[, phase])`，或是删除：`removeEventListener`。

谨慎使用 HTML 属性绑定，因为HTML标签中间掺杂 JavaScript 会看起来有点奇怪。也不建议在属性中写大量的代码。

可以使用 DOM 属性，但是我们不能为特定事件绑定多个处理函数。大多情况下，这点限制并不是什么问题。

最后的方式是最灵活的，但代码写起来最长。有些事件绑定必须用这种方式，例如`transtionend`和`DOMContentLoaded`（还有其他的一些）。

当处理函数被调用时，它将获取一个事件对象，并将该事件对象作为第一个参数。该对象包含发生的事件的详细信息。之后我们会学习更多。

现在，我们只是学习了事件处理的基础。在接下来的章节中，会讨论更多细节。
