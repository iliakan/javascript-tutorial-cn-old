# Async/await

使用 "async/await" 语法来书写 promises 代码会更简便舒适。而且它很容易理解和使用。

## Async functions

让我们从 `async` 这个关键词着手开始。它可以被放置在 function 前面，就像这样：

```js
async function f() {
  return 1;
}
```

将 "async" 放在 function 的前面的意思非常简单：这个 function 永远返回一个 promise。如果不是，那该返回值会被 `Promise.resolve` 包裹起来。

举个例子，下面的代码返回 `Promise.resolve(1)`:

```js
async function f() {
  return 1;
}

f().then(alert); // 1
```

...我们可以显式的返回一个 promise，它们效果一样：

```js
async function f() {
  return Promise.resolve(1);
}

f().then(alert); // 1
```

所以，`async` 保证 function 返回一个 promise，用 promise 包裹非 promise。非常简单，对吧？但是这不是全部。另外一个关键词 `await` 只在 `async` function 内部工作，这非常的 coooool。

## Await

语法：

```js
// 只在 async functions 内工作
let value = await promise;
```

关键词 `await` 使 JavaScript 一直等到 promise 执行完成并且返回它的结果。

下面是一个 promise 1秒后解决的例子：
```js
async function f() {

  let promise = new Promise((resolve, reject) => {
    setTimeout(() => resolve("done!"), 1000)
  });

*!*
  let result = await promise; // 等待直到这个 promise 解决（*）
*/!*

  alert(result); // "done!"
}

f();
```

function 的执行在 `(*)` 处被 "暂停"，然后 promise 完成以后恢复执行，`result` 变成了它的结果。所以上面的代码会一秒以后显示 "done!"。

再强调一下：`await` 使 JavaScript 等待直到 promise 完成，然后通过 promise 返回值继续执行。它不会花费任何 CPU 资源，因为引擎可以同时做其它的事情：执行其它的脚本，处理事件等。

它只是一个比 `promise.then` 更优雅的获取 promise 结果的语法，阅读和书写更简单。


如果我们尝试在 非-async function 内使用 `await`，会得到一个语法错误：

```js 
function f() {
  let promise = Promise.resolve(1);
*!*
  let result = await promise; // 语法错误
*/!*
}
```

如果我们忘记在 function 前放 `async`，我们会得到这样的错误。正如前面那样说的，`await` 只在 `async function` 内工作。

让我们再看看在 <promise-chaining> 那章的 `showAvatar()` 例子，我们将会用 `async/await` 重写它：

1. 首先我们需要用 `wait` 代替 `.then`。
2. 然后，我们应该让 function `async` 化,这样 `wait` 才能工作。

```js 
async function showAvatar() {

  // 读取我们的 JSON
  let response = await fetch('/article/promise-chaining/user.json');
  let user = await response.json();

  // 读取 github 用户
  let githubResponse = await fetch(`https://api.github.com/users/${user.name}`);
  let githubUser = await githubResponse.json();

  // 显示头像
  let img = document.createElement('img');
  img.src = githubUser.avatar_url;
  img.className = "promise-avatar-example";
  document.body.append(img);

  // 等待3秒
  await new Promise((resolve, reject) => setTimeout(resolve, 3000));

  img.remove();

  return githubUser;
}

showAvatar();
```

非常简介，而且阅读方便，是吧？比以前好多了。

我们不能把 `await` 放在代码的最顶层，但是那些刚开始使用 `wait` 的人们往往会忘记。那不会工作：

```js run
// 在最顶层的代码使用会出现语法错误
let response = await fetch('/article/promise-chaining/user.json');
let user = await response.json();
```

所以我们需要给那些 `awaits` 一个包裹 async function。就像上面的例子一样。

和 `promise.then` 一样, `await` 允许使用 thenable 对象（那些有一个可执行的 `then` 方法）。 同时，第三方对象可能不是一个 promise，但是兼容 promise：如果它支持 `.then`,那它就可以使用`await`.

举个例子，这里的 `await` 接受 `new Thenable(1)`：
```js run
class Thenable {
  constructor(num) {
    this.num = num;
  }
  then(resolve, reject) {
    alert(resolve); // function() { 原生代码 }
    // 1000ms 后解决，结果是 this.num*2
    setTimeout(() => resolve(this.num * 2), 1000); // (*)
  }
};

async function f() {
  // 等待1秒，然后结果变成2
  let result = await new Thenable(1);
  alert(result);
}

f();
```

如果 `await` 得到一个包含 `.then` 的 non-promise 对象，它执行那个方法，并提供原生的函数 `resolves`，`reject`作为参数。然后 `wait` 等待直到它们中的一个被执行（在上面的代码中，在 `(*)`处），再通过它的返回值继续执行。

一个 class 的方法也能是 async，只用在它前面放一个 `async`。

就像这样：

```js
class Waiter {
*!*
  async wait() {
*/!*
    return await Promise.resolve(1);
  }
}

new Waiter()
  .wait()
  .then(alert); // 1
```
它的含义是一样的：它保证方法的返回值是一个 promise，能 `await`。

## 错误处理

如果一个 promise 正常解决，那么 `await promise` 返回结果。但是如果 promise 被拒绝，抛出一个错误，就像有一个 `throw` 声明在那一行。

这个代码：

```js
async function f() {
*!*
  await Promise.reject(new Error("Whoops!"));
*/!*
}
```

...和上面的一样：

```js
async function f() {
*!*
  throw new Error("Whoops!");
*/!*
}
```

在实际的情况下，promises 在它拒绝之前可能花费一段时间。所以 `await` 会等待，然后再抛出错误。

我们可以使用 `try..catch` 捕捉到那个错误，和正常的 `throw` 一样：

```js
async function f() {

  try {
    let response = await fetch('http://no-such-url');
  } catch(err) {
*!*
    alert(err); // TypeError: 拉取失败
*/!*
  }
}

f();
```

如果发生了错误，程序的控制权跳到 `catch` 代码块。我们也能用多行包裹：

```js run
async function f() {

  try {
    let response = await fetch('/no-user-here');
    let user = await response.json();
  } catch(err) {
    // 同时捕获拉取和 response.json 的错误
    alert(err);
  }
}

f();
```

如果我们没有 `try..catch`，然后通过执行 async 函数 `f()` 生成的 promise 变成拒绝状态。我们可以添加 `.catch` 处理它：

```js run
async function f() {
  let response = await fetch('http://no-such-url');
}

// f() 变成一个 拒绝的 promise
*!*
f().catch(alert); // TypeError: 拉取失败 // (*)
*/!*
```

如果我们忘记添加 `.catch`，我们就会的到一个没有处理的 promise 错误（可以在 console 看见它）。我们像在 <promise-chaining> 所描述的一样可以用一个全局的事件捕获错误。

当我们使用 `async/await` 的时候，我们很少需要 `.then`，因为 `await` 为我们处理等待。我们可以使用常用的 `try..catch` 代替 `.catch`。这通常（并不是一直）会更方便。

但是在代码的最顶层，当我们在任何 `async` 函数外面时，我们语法上不能使用 `await`，所以添加 `.then/catch` 来处理最终结果或者错误是很有必要的。

就像上面例子中的 `(*)` 处。
```

当我们需要等待多个 promises 时，我们能用 `Promise.all` 包裹它们，然后 `await`：

```js
// 等待返回结果的数组
let results = await Promise.all([
  fetch(url1),
  fetch(url2),
  ...
]);
```

如果发生错误，它就像平常一样传递：从失败的 promise 到 `Promise.all`，然后变成一个我们可以用 `try..catch` 捕获的异常。

## 总结

function 前面的 `async` 关键词有两个效果：

1. 让函数永远返回一个 promise。 
2. 允许在函数里面使用 `await` 。

promise 前面的 `await` 关键词让 JavaScript 等待直到该 promise 完成，然后：

1. 如果它是一个错误，就会生成异常，就像 `throw error` 在那个地方被执行一样。
2. 否则，它返回结果，然后我们可以分配它给一个变量。

它们提供了一个非常棒的框架来编写异步代码，读和写都非常简单。

使用 `async/await` 的时候，我们基本不需要写 `promise.then/catch`，但是我们仍然不应该忘记它们是基于 promises，因为有时（e.g. 在最外层）我们不得不使用那些方法。同时， `Promise.all` 用来同时等待多个任务非常棒。