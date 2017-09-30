# Promise API

`Promise` 库包含四个静态的方法。下面我们马上就会讲解他们。

## Promise.resolve

语法：

```js
let promise = Promise.resolve(value);
```

返回一个为已解决状态的值是 `value` promise。

和上面一样：

```js
let promise = new Promise(resolve => resolve(value));
```

当我们已经有了一个返回值，但是我们希望把它用 promise 包裹的时候，就会用这个方法。

例如，下面的 `loadCached` 函数会拉取 `url` 的数据，并且会记住返回值，所以之后的再次拉取同一个 URL 会马上返回：

```js
function loadCached(url) {
  let cache = loadCached.cache || (loadCached.cache = new Map());

  if (cache.has(url)) {
*!*
    return Promise.resolve(cache.get(url)); // (*)
*/!*
  }

  return fetch(url)
    .then(response => response.text())
    .then(text => {
      cache[url] = text;
      return text;
    });
}
```

因为该方法保证返回一个 promise ，所以我们可以使用 `loadCached(url).then(…)`。这就是在 `(*)` 出的 `Promise.resolve` 目的： 它确保了界面的统一。我们通常都可以在 `loadCached` `.then`。

## Promise.reject

语法：

```js
let promise = Promise.reject(error);
```

创建一个包含 `error` 的已拒绝 promise。

同样的效果：

```js
let promise = new Promise((resolve, reject) => reject(error));
```

在实际代码中我们基本不会这样使用，这里只是为了展示所有的语法。

## Promise.all

这个方法可以同时运行很多个 promises，并且等待它们全部完成。

语法是这样的：

```js
let promise = Promise.all(iterable);
```

它接受一组 `可迭代` 的 promises，技术上来说，可以是任何可迭代的东西。但是通常来说都是数组，它会返回一个新的 promise。在所有的 promises 都完成以后，返回的新 promise 会返回一个数组并被置为已解决。

例如，下面的 `Promise.all` 会在三秒之后完成，它会返回数组 `[1, 2, 3]`：

```js run
Promise.all([
  new Promise((resolve, reject) => setTimeout(() => resolve(1), 3000)), // 1
  new Promise((resolve, reject) => setTimeout(() => resolve(2), 2000)), // 2
  new Promise((resolve, reject) => setTimeout(() => resolve(3), 1000))  // 3
]).then(alert); // 1,2,3 当 promises 准备好了：每个 promise 分别返回数组里的一个值
```

请注意，它们的相对顺序是一样的。尽管第一个 promise 需要最长的时间去完成，它的返回值依然是返回数组的第一个值。

一个常用的技巧是将一组任务放进一个 promises 数组，然后用放进 `Promise.all`。

例如，我们有一组 URLS，我们能同时拉取它们的数据，就像下面这样：

```js run
let urls = [
  'https://api.github.com/users/iliakan',
  'https://api.github.com/users/remy',
  'https://api.github.com/users/jeresig'
];

// 使用 map 方法拉取所有的 url
let requests = urls.map(url => fetch(url));

// Promise.all 会等到所有的任务都被完成
Promise.all(requests)
  .then(responses => responses.forEach(
    response => alert(`${response.url}: ${response.status}`)
  ));
```

一个更符合实际生活的例子是通过用户名拉取一组 github 用户的数据（或者我们可以用过商品 id 拉取一组商品的数据，逻辑是一样的）：

```js run
let names = ['iliakan', 'remy', 'jeresig'];

let requests = names.map(name => fetch(`https://api.github.com/users/${name}`));

Promise.all(requests)
  .then(responses => {
    // 所有的响应都准备好了，我们可以展示 HTTP 状态码
    for(let response of responses) {
      alert(`${response.url}: ${response.status}`); // 每个 url 都会显示200
    }

    return responses;
  })
  // 使用 map 方法将响应数组放进 response.json() 的数组去读取他们的内容
  .then(responses => Promise.all(responses.map(r => r.json())))
  // 所有的 JSON 答案都被解析："users" 被解析后结果的数组
  .then(users => users.forEach(user => alert(user.name)));
```

如果其中的任何一个 promise 返回，则 `Promise.all` 会马上返回这个错误并置为拒绝状态。

举个例子：


```js run
Promise.all([
  new Promise((resolve, reject) => setTimeout(() => resolve(1), 1000)),
*!*
  new Promise((resolve, reject) => setTimeout(() => reject(new Error("Whoops!")), 2000)),
*/!*
  new Promise((resolve, reject) => setTimeout(() => resolve(3), 3000))
]).catch(alert); // 错误： Whoops!
```

上面的代码中，第二个 promise 会在两秒以后置为拒绝状态。这会导致 `Promise.all` 马上被置为拒绝状态，所以 `.catch` 被执行：返回的错误信息成为整个 `Promise.all` 的返回值。

重要的是 promises 没有提供办法去 "取消" 或者 "退出" 它们的执行。所以其他的 promises 继续执行，最终被完成，但是他们的返回解决会被忽略。

有办法避免这样的问题：我们可以写额外的代码去 `clearTimeout` （或者取消）报错的 promises，也可以将错误作为返回数组的成员展示出来（详情请看本章的 task）。

一般而言， `Promise.all(iterable)` 接受一个可迭代的（通常是数组）promises。但是如果那些对象里有非 promise，它会被包裹成 `Promise.resolve`.

例如，下面代码的结果是 `[1, 2, 3]`：

```js run
Promise.all([
  new Promise((resolve, reject) => {
    setTimeout(() => resolve(1), 1000)
  }),
  2, // 被当成 Promise.resolve(2) 对待
  3  // 被当成 Promise.resolve(3) 对待
]).then(alert); // 1, 2, 3
```

所以，我们也可以传递非 promise 值给 `Promise.all`，实在是很方便。


## Promise.race

和 `Promise.all` 一样，它接受可迭代的多个 promises，但是它不会等待所有的 promises 都完成 -- 而是等待最先完成的结果（或者错误），然后通过它继续。

它的语法是这样的：

```js
let promise = Promise.race(iterable);
```

下面的例子中的结果会是 `1`：

```js run
Promise.race([
  new Promise((resolve, reject) => setTimeout(() => resolve(1), 1000)),
  new Promise((resolve, reject) => setTimeout(() => reject(new Error("Whoops!")), 2000)),
  new Promise((resolve, reject) => setTimeout(() => resolve(3), 3000))
]).then(alert); // 1
```

所以，第一个结果/错误成了整个 `Promise.race` 的结果。等到第一个完成的 promise "赢得了赛跑"，所有其他的结果/错误都会被无视。

## Summary

`Promise` 库包含四个静态的方法：

1. `Promise.resolve(value)` -- 返回一个值，并将 promise 置为已处理，
2. `Promise.reject(error)` -- 返回一个值，并将 promise 置为已拒绝，
3. `Promise.all(promises)` -- 等待所有的 promises 完成，然后返回他们的结果的数组。如果其中任何一个 promise 被拒绝，则它会变成 `Promise.all` 的错误，其他的所有结果都会被无视。
4. `Promise.race(promises)` -- 等待最先完成的 promise，然后它的结果/错误变成结果。

`Promise.all` 是四个中是实际中使用最常见的一个。