# 事件冒泡（bubbling）和事件捕获（capturing）

让我们先看一个例子。

下面的事件处理器被绑定到 `<div>` 元素，但是如果你点击任何嵌套的标签，如 `<em>` 或者是 `<code>`，依然会执行事件：

```html autorun height=60
<div onclick="alert('The handler !')">
  <em>If you click on <code>EM</code>, the handler on <code>DIV</code> runs.</em>
</div>
```

这是不是有点奇怪？为什么实际上我们点击的是 `<em>`，然而绑定在 `<div>` 上事件被执行了呢？

## 事件冒泡（Bubbling）

事件冒泡的原理其实很简单。

**当在某个元素上发生事件时，首先执行绑定在该元素上的事件处理器，然后执行其父节点上的，然后以此类推，执行其他祖先节点上的事件处理器。**

假设我们有3个嵌套元素 `FORM > DIV > P`，每个元素都有一个处理函数：

```html run autorun
<style>
  body * {
    margin: 10px;
    border: 1px solid blue;
  }
</style>

<form onclick="alert('form')">FORM
  <div onclick="alert('div')">DIV
    <p onclick="alert('p')">P</p>
  </div>
</form>
```

点击内部的 `<p>` 标签，执行 `onclick` 事件的顺序如下：
1. 执行在 `<p>` 元素上的事件。
2. 然后在外面的 `<div>` 元素上的事件。
3. 然后在外层的 `<form>` 元素上的事件。
4. 以此类推，直到 `document` 对象。

![](event-order-bubbling.png)

所以如果我们点击 `<p>` 标签，那么我们会看到3个弹出框：`p` -> `div` -> `form`。

这个过程被称为“冒泡”，因为事件像“气泡”在水中浮起一样，从内部元素自下而上传递。

```warn header="*几乎* 所有的事件都是冒泡的"
注意这里用的是“几乎”。

例如，`focus` 事件不会冒泡。还有一些其他的例子，之后我们将会看到。但这些是例外情况，并不是通用的，大多数事件还是冒泡事件。
```

## event.target

父元素上的事件处理器，总是可以获取事件实际发生位置的详细信息。

**最深层的嵌套元素，即产生事件的元素被称为*target*元素，可以用 `event.target` 来访问。**

注意与 `this` 的区别（this=`event.currentTarget`）：

- `event.target` - 是最开始产生事件的“目标” 元素，它不会在冒泡过程而改变。
- `this` - 是“当前”元素，它取决于当前正在执行的事件处理器。

例如，如果我们有一个单独的事件处理器 `form.onclick`，那么它可以“捕获”表单中的所有点击事件。无论点击了哪里，当事件冒泡到 `<form>` 元素时，就会执行事件处理器。

在 `form.onclick` 事件处理器中：

- `this` (`=event.currentTarget`) 是 `<form>` 元素，事件处理器绑定在该元素上。
- `event.target` 是表单中实际被点击的的元素。

具体例子如下：

[codetabs height=220 src="bubble-target"]

`event.target` 有可能和 `this` 是一样的 -- 当点击直接发生在表单元素上的时候（译者注：比如点击了表单余白部分）.

##停止冒泡

冒泡事件会从目标元素开始，向上移动。 通常事件会向上移动到 `<html>`，然后到 `document` 对象，一些事件甚至可以到达 `window`，在事件向上移动时，会调用路径上的所有的事件处理器。

但注意，任何的事件处理器，都可以完成事件处理，并停止事件冒泡。

这个方法是 `event.stopPropagation()`。

比如下面的例子，如果点击 `<button>` 按钮，`body.onclick` 并不会执行：

```html run autorun height=60
<body onclick="alert(`the bubbling doesn't reach here`)">
  <button onclick="event.stopPropagation()">Click me</button>
</body>
```

```smart header="event.stopImmediatePropagation()"
如果一个元素对于某个事件，绑定了多个事件处理器，那么即使在其中一个处理器中停止冒泡，其他的事件处理器仍然会执行。

换句话说，`event.stopPropagation()` 停止事件向上移动，但在当前元素上，所有的事件处理器都会执行。

要停止冒泡并停止执行当前元素的其他事件处理器，有一个方法 `event.stopImmediatePropagation()`。调用这个函数后，其他的事件处理器就不会执行。
```

```warn header="如果没有充分的必要，不要停止事件冒泡"
事件冒泡是很自然的。如果没有真正的需求，不要停止它：事件冒泡是显而易见的，架构时请慎重考虑。

有时候 `event.stopPropagation()` 会产生隐患，之后可能会带来问题。

例如：

1. 我们来创建一个嵌套式菜单。每个子菜单处理点击事件时，调用 `stopPropagation`，这样外部元素的事件就不会被触发。
2. 之后我们决定捕捉整个窗口内的点击，并跟踪用户的行为（用户点击的地方）。一些计数程序会这样做。通常，计数器的代码会通过 `document.addEventListener('click'…)` 来实现。
3. 我们的计数器无法正常在 `stopPropagation` 之后执行，即停止点击的区域。这里就是所谓的“代码死区”。

通常来说，没有必要停止冒泡。其实可以使用自定义事件，我们稍后会介绍。此外，我们可以将数据写入处理器中的 `event` 对象，并在另外一个处理器中读取这个对象，这样我们就可以把子节点的信息传递给父节点的处理器了。
```


##事件捕获（Capturing）

还有另一个事件处理的阶段，该阶段被称为“事件捕获”。在实际代码中很少使用它，但有时它却很有用。

标准[DOM 事件](http://www.w3.org/TR/DOM-Level-3-Events/) 有3个阶段的事件传播：

1. 捕获阶段 - 事件从上至下传播到目标元素。
2. 目标阶段 - 事件抵达目标元素。
3. 冒泡阶段 - 事件从目标元素开始向上冒泡。

下面的示例图是当点击表格中的 `<td>` 元素时的不同阶段，该图片来自 W3 官方文档：

![](eventflow.png)

说明：对于 `<td>` 元素的点击，事件首先从根节点（window 对象）沿着树的路径抵达目标元素（事件捕获），然后开始向上回溯到根结点（事件冒泡），并在回溯的路径上调用事件处理器。

**由于捕获阶段的事件很少使用，之前我们所谈到的事件都是冒泡阶段的事件。通常我们不会注意捕获阶段的事件。**

使用 `on<event>`绑定事件，或是使用 HTML 属性，或使用 `addEventListener(event, handler)` 添加的处理程序并不知道事件捕获，它们只在第 2 和第 3 阶段运行。

要截获捕获阶段的事件，我们需要将 `addEventListener` 的第三个参数设置为 `true`。

实际上，`addEventListener` 函数的最后一个参数有两个可能的值：

- 如果最后一个参数是 `false`（默认值），那么在事件处理器会在冒泡阶段执行。
- 如果最后一个参数是 `true`，则事件处理器会在捕获阶段执行。

这里请注意，事件处理实际上是有 3 个阶段，第二阶段（“目标阶段”：当事件到达该元素的时候）不是单独处理的：事件捕获和事件冒泡都会触发目标阶段。

目标元素的事件，在捕获阶段中最后触发，然后首先在冒泡阶段最先触发。

让我们看一个例子：

```html run autorun height=140 edit
<style>
  body * {
    margin: 10px;
    border: 1px solid blue;
  }
</style>

<form>FORM
  <div>DIV
    <p>P</p>
  </div>
</form>

<script>
  for(let elem of document.querySelectorAll('*')) {
    elem.addEventListener("click", e => alert(`Capturing: ${elem.tagName}`), true);
    elem.addEventListener("click", e => alert(`Bubbling: ${elem.tagName}`));
  }
</script>
```

该代码在 HTML 文档中的*每个*元素上都设置了点击事件的处理器，以便于查看哪些处理器是正在执行的。

如果你点击 `<p>`，那么执行顺序是：

1. `HTML` -> `BODY` -> `FORM` -> `DIV` -> `P` (捕获阶段, for 循环中的第一个事件监听), 然后是:
2. `P` -> `DIV` -> `FORM` -> `BODY` -> `HTML` (冒泡阶段, for 循环中的第二个事件监听).

请注意，`P` 显示了两次：分别是在捕获结束时和冒泡开始时。

还有一个属性 `event.eventPhase`，这个属性表示当前的事件正处于那个阶段。但是这个参数很少使用，因为通常在绑定事件处理器时，我们会预先绑定到相应的阶段。

##总结

事件处理流程：

- 当一个事件发生时 - 产生事件的嵌套在最内层的元素被标记为“目标元素”（`event.target`）。
- 然后事件首先从HTML文档树的根元素开始，向下移动到 `event.target`，在路径上调用 `addEventListener(...., true)` 绑定的事件处理器。
- 然后事件从 `event.target` 移动到HTML文档树的根元素，调用使用 `on<event>` 和 `addEventListener` 绑定的事件处理器，使用 `addEventListener` 绑定时不传第3个参数或第3个参数要用 `false`。

每个事件处理器都可以访问 `event` 对象的属性：

- `event.target` - 触发事件的最内层的元素。
- `event.currentTarget` (=`this`) - 当前正在处理事件的元素（当前的元素已经绑定某事件处理器）
- `event.eventPhase` - 当前事件的阶段（capture = 1，bubbling = 3）。

任意的事件处理器都可以通过调用 `event.stopPropagation()` 来阻止事件继续，但是不推荐这样做，因为我们不能确定接下去的处理是否需要，可能完全是些不一样的处理。

捕获阶段的事件很少使用，通常我们处理冒泡阶段的事件。这里面的逻辑是这样的。

在现实世界中，当事故发生时，地方当局首先作出反应。因为他们最了解事故发生的地区。然后根据需要，再上报高层机关。

事件处理程序也是一样。绑定在特定元素上的处理器，这个处理器会处理关于该元素的各种细节。绑定在元素 `<td>` 上的处理器应该是最适用于 `<td>` 的，因为该处理器知道该元素的各种信息，所以它应该先执行。然后执行父元素的事件处理器，该处理器处于事件发生的上下文中，只是相对于目标元素少一些信息，以此类推，直到顶层元素，最后进行通用的处理。

事件冒泡和事件捕获是“事件代理”的基础，"事件代理"是非常强大的事件处理模式，在下一章中，我们会进一步来学习。
