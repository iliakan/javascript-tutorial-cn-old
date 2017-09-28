# Promise

想象一下, 你现在是周杰伦, 你的粉丝们一直在问你的下一首单曲 "谁都听不懂" 什么时候出来.

为了摆脱这无穷无尽的为什么, 你承诺只要歌曲一发售马上告诉他们. 你让粉丝们在一份表格上填写自己的名字和地址, 等到歌曲创作完成, 他们就马上可以知道啦. 如果出了什么意外, 歌曲永远不会发售的话, 粉丝也会得到这个消息.

现在每个人都开心了: 你再也不会被粉丝纠缠了, 而你的粉丝们也不会错误你的下一首单曲.

我们在写代码的时候也经常会遇到这种问题:

1. "生产代码" 会完成一些任务但是需要一段时间去完成. 例如载入新的文件. 在上面的例子中, 周杰伦就是 "生产代码".
2. "消费代码" 希望在 "生成代码" 完成任务以后马上知道. 粉丝们就是 "消费代码".
3. *promise* 是一个将 "生产代码" 和 "消费代码" 联系起来的JavaScript对象. 周杰伦让粉丝填的表格就是 *promise*. "生产代码" 完成任务以后通过 *promise* 告诉 "消费代码".

这个比喻不是很准确, 因为 JavaScript promises 要比一个表格复杂很多: promises有很多其它的特征和限制. 不过他们大体上还是相似的.

Promise对象的构造语法是这样的:

```js
let promise = new Promise(function(resolve, reject) {
  // executor (the producing code, "singer")
});
```

传递给 `new Promise` 的函数被称为 *executor*. 当promises被创建以后, *executor* 会立即执行. 它包含了生产代码, 生产代码最终会执行完成并返回结果. 就我们最开始的比喻来说, "周杰伦"就是 executor.

`promise` 对象有几个内在的属性:

- `state` -- 初始值是 "pending", 然后变成 "fulfilled" 或者 "rejected",
- `result` -- 可以是任何值, 初始值是 `undefined`.

当executor执行完成以后, 它会执行下列中的一个:

- `resolve(value)` -- 标识任务执行成功:
    - 设置 `state` 为 `"fulfilled"`,
    - 设置 `result` 为 `value`.
- `reject(error)` -- 标识任务执行失败:
    - 设置 `state` 为 `"rejected"`,
    - 设置 `result` 为 `error`.

![](promise-resolve-reject.png)

下面是一个简单的完整executor例子:

```js run
let promise = new Promise(function(resolve, reject) {
  // 这个函数将会在Promise创建以后自动执行

  alert(resolve); // function () { [native code] }
  alert(reject);  // function () { [native code] }

  // 1秒以后,任务执行成功
  setTimeout(() => *!*resolve("done!")*/!*, 1000);
});
```

执行上面的代码后, 我们可以得到两件事情:

1. executor是自动和立即执行的 (通过 `new Promise`).
2. executor接收两个参数: `resolve` 和 `reject` -- 它们都是JavaScript引擎自带的. 我们不需要专门创建它们. 在准备好以后,executor会执行他们.

executor在思考1秒以后,通过执行 `resolve("done")` 来产生结果:

![](promise-resolve-1.png)

上面是一个 "成功完成任务" 的例子.

下面是一个失败并返回错误的例子:

```js
let promise = new Promise(function(resolve, reject) {
  // 1秒以后,任务执行失败
  setTimeout(() => *!*reject(new Error("Whoops!"))*/!*, 1000);
});
```

![](promise-reject-1.png)

简单来说, executor会执行一个任务 (通常来说需要点时间), 然后它会通过执行 `resolve` 或者 `reject` 来改变promises对象的状态.

promise的resolved或者rejected状态被称为 "settled", 区别于初始的 "pending" 状态.

executor只会执行一次 `resolve` 或者 `reject`, 然后promises的状态被改变.

之后的所有的 `resolve` 和 `reject` 都会被忽略:

```js
let promise = new Promise(function(resolve, reject) {
  resolve("done");//执行

  reject(new Error("…")); // 忽略
  setTimeout(() => resolve("…")); // 忽略
});
```

这是因为executor执行完成以后只会返回一个结果, 要么成功要么失败. 在程序语言中, 存在一些其他的数据结构允许多个返回结果, 例如streams和queues. 相对于promise来说, 他们各有千秋. JavaScript核心不支持streams和queues, 同时promises更能解决我们的问题, 所以我们会专注于讨论promises.

同时, 如果我们给 `resolve/reject` 传递了多个参数 -- 只有第一个会被使用, 其他的都会被无视.

技术上来说, 我们可以给 `reject` (就像 `resolve` 一样) 传递任何类型的参数. 但是一般推荐使用 `Error` 对象作为 `reject` 的参数 (或者继承于Error对象). 这样做的原因你马上就会明白.
```

一般而言, executor通常会执行一些异步的动作, 然后一段时间以后执行 `resolve/reject` , 但是也有例外. 我们可以马上执行 `resolve` 或者 `reject` , 就像这样:

```js
let promise = new Promise(function(resolve, reject) {
  resolve(123); // 马上返回结果: 123
});
```

这就像我们开始了一项任务, 然后顺利的完成了它: 我们兑现了之前的承诺.

`state` 和 `result` 是promises的两个内部属性. 我们并不能直接通过代码访问, 但是我们可以使用 `.then/catch` 来满足我们想要做的, 现在就让我们来讨论这两个方法.

## 消费者: ".then" 和 ".catch"

promises对象连接着生成代码(executor)和消费函数 -- 接收返回的结果/错误. 我们使用 `promise.then` 和 `promise.catch` 来定义消费函数.


`.then` 的语法是这样的:

```js
promise.then(
  function(result) { /* 处理成功的结果 */ },
  function(error) { /* 处理错误的结果 */ }
);
```

当promise执行成功后就会运行第一个函数参数, 如果是返回错误的话, 就会执行第二个函数.

举个例子:

```js run
let promise = new Promise(function(resolve, reject) {
  setTimeout(() => resolve("done!"), 1000);
});

// .then的第一个函数被执行
promise.then(
*!*
  result => alert(result), // 1秒以后显示 "done!"
*/!*
  error => alert(error) // 不会被执行
);
```

如果是失败的话:

```js run
let promise = new Promise(function(resolve, reject) {
  setTimeout(() => reject(new Error("Whoops!")), 1000);
});

// .then的第二个函数会被执行
promise.then(
  result => alert(result), // 不会被执行
*!*
  error => alert(error) // 1秒以后显示 "Error: Whoops!" 
*/!*
);
```

如果我们只关心执行成功的返回值的话, 我们可以值提供一个参数给 `.then`:

```js run
let promise = new Promise(resolve => {
  setTimeout(() => resolve("done!"), 1000);
});

*!*
promise.then(alert); // 1秒以后显示 "done!"
*/!*
```

如果我们只关心错误返回, 那么我们可以使用 `.then(null, function)` 或者 `.catch(function)`


```js run
let promise = new Promise((resolve, reject) => {
  setTimeout(() => reject(new Error("Whoops!")), 1000);
});

*!*
// .catch(f) 和 promise.then(null, f) 效果一样
promise.catch(alert); // 1秒以后显示 "Error: Whoops!" 
*/!*
```

`.catch(f)` 和 `.then(null, f)` 的效果是完全一样的, 只是简写.

如果promises是pending状态, `.then/catch` 会一直等待结果. 相反, 如果promise已经执行, `.then/catch` 会立刻执行:

```js run
// 立刻执行的resolved promise
let promise = new Promise(resolve => resolve("done!"));

promise.then(alert); // done! (马上显示)
```

执行的任务可能会马上成功也可能会需要一段时间. 所以promise.then需要保证两种情况都能正常处理.

更值得庆幸的是, 当 `.then/catch` 处理者执行的时候, 它会被放进一个内部的队列. JavaScript引擎在当前代码执行完成以后,会从队列里面取出对应的处理者, 类似于 `setTimeout(..., 0)`.

简单来说, 当 `.then(handler)` 将要被触发的时候, 它就像 `setTimeout(handler, 0)` 一样处理.

在下面的例子中, promise马上返回成功, 所以 `.then(alert)` 立即被触发: `alert` 会被立即执行.

```js run
// 一个立刻成功的promise
let promise = new Promise(resolve => resolve("done!"));

promise.then(alert); // done! (代码完成马上触发)

alert("code finished"); // 这个alert会先显示
```

一般来说 `.then` 之后的代码会在处理者之前执行 (即时是立刻返回的promise). 一般来说不用在意这个问题, 有些时候也需要考虑到.

现在让我们尝试更多练习代码, 看看promises是怎样帮助我们编写异步代码的.

## 例子: loadScript

在上一章中我们设计了一个载入新文件的 `loadScript`函数.

基于回调函数的解决方案是这样的:

```js
function loadScript(src, callback) {
  let script = document.createElement('script');
  script.src = src;

  script.onload = () => callback(null, script);
  script.onerror = () => callback(new Error(`Script load error ` + src));

  document.head.append(script);
}
```

现在让我们用promise改写它.

新的 `loadScript` 函数将不再需要回调函数. 取而代之的是, 它创建然后返回一个在加载文件成功以后会修改状态的promise对象. 外部代码可以通过 `.then`来使用它:

```js run
function loadScript(src) {  
  return new Promise(function(resolve, reject) {
    let script = document.createElement('script');
    script.src = src;

    script.onload = () => resolve(script);
    script.onerror = () => reject(new Error("Script load error: " + src));

    document.head.append(script);
  });
}
```

使用方法:

```js run
let promise = loadScript("https://cdnjs.cloudflare.com/ajax/libs/lodash.js/3.2.0/lodash.js");

promise.then(
  script => alert(`${script.src} is loaded!`),
  error => alert(`Error: ${error.message}`)
);

promise.then(script => alert('One more handler to do something else!'));
```

对比回调函数的解决方案:

``` 
-="回调" +="Promises"
- 我们需要在执行 `loadScript` 前就写好 `回调`. 也就是说我们必须在 `loadScript` 返回结果之前就要知道做什么.
- 只有一个回调.
+ promise允许我们以正常的顺序书写异步编程. 我们先执行 `loadScript`, 然后在 `.then` 使用结果.
+ 我们可以在之后的任何时候执行 `.then` 多次.
```

综上所述, promise让我们在异步编程的时候更方便和灵活. 但是只有这些. 我们将在下一章讨论它.