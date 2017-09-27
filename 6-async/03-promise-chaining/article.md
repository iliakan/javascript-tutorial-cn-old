
# Promises 链式操作

让我们回到在回调那一章提到的问题:

- 我们有一系列需要一个接一个完成的异步任务。例如，载入新的脚本文件。
- 怎样优雅的书写代码？

Promises 提供了优雅的解决方案。

[cut]

在这章中，我们将讨论 promises 链式操作。

它就像这样：

```js run
new Promise(function(resolve, reject) {

  setTimeout(() => resolve(1), 1000); // (*)

}).then(function(result) { // (**)

  alert(result); // 1
  return result * 2;

}).then(function(result) { // (***)

  alert(result); // 2
  return result * 2;

}).then(function(result) {

  alert(result); // 4
  return result * 2;

});
```

技巧是，我们将上一步的结果传递给 `.then` 链。 

它的流程是这样的：
1. 初始的 promise 在一秒以后成功 `(*)`，
2. 然后 `.then` 处理者被执行 `(**)`。
3. 上一步的返回值被传递到下一个 `.then` 处理者 `(***)`
4. ...等等。

当结果被一步一步传递给处理者链以后，`alert` 被按照顺序一步一步执行：`1` -> `2` -> `4`。

![](promise-then-chain.png)

整个流程没毛病，这是因为 `promise.then` 会返回 promise ，所以我们可以继续使用 `.then` 执行它。

每当处理者返回一个值，它就变成了那个 promise 的结果，所以下一个 `.then` 通过它被执行。

不跟你多BB，下面是一个简单粗暴的链式操作例子：

```js run
new Promise(function(resolve, reject) {

  setTimeout(() => resolve(1), 1000);

}).then(function(result) {

  alert(result);
  return result * 2; // <-- (1)

}) // <-- (2)
// .then…
```

`.then` 的返回值也是一个 promise ，这就是为什么我们能继续在 `(2)` 添加 `.then` 。当 `(1)` 的值返回以后，promise 变成 resolved 状态，所以下一个处理者使用那个结果继续执行。 

和链式操作不一样的是，技术上来说，我们可以给单个 promise 添加很多个 `.then` ，就像这样：

```js run
let promise = new Promise(function(resolve, reject) {
  setTimeout(() => resolve(1), 1000);
});

promise.then(function(result) {
  alert(result); // 1
  return result * 2;
});

promise.then(function(result) {
  alert(result); // 1
  return result * 2;
});

promise.then(function(result) {
  alert(result); // 1
  return result * 2;
});
```

...它们是完全不一样的事情。有图有真相：

![](promise-then-many.png)

同一个 promise 的所有 `.then` 都会返回相同的结果 -- 这个 promise 的结果。所以上图的代码的所有 `alert` 都会返回相同的结果： `1`。他们之间没有传递结果。

一般来说，一个 promise 很少需要多个处理者。链式操作使用的更多一点。

## 返回 promises

一般而言，`.then` 的返回值会立刻传递给下一个处理者。但是也有例外的时候。

如果返回值是一个 promise ，之后的代码会等待这个 promise 完成之后执行。promise 完成之后，它的返回值会被传递给下一个 `.then` 。

例如：

```js run
new Promise(function(resolve, reject) {

  setTimeout(() => resolve(1), 1000);

}).then(function(result) {

  alert(result); // 1

*!*
  return new Promise((resolve, reject) => { // (*)
    setTimeout(() => resolve(result * 2), 1000);
  });
*/!*

}).then(function(result) { // (**)

  alert(result); // 2

  return new Promise((resolve, reject) => {
    setTimeout(() => resolve(result * 2), 1000);
  });

}).then(function(result) {

  alert(result); // 4

});
```

第一个 `.then` 显示 `1` 然后在 `(*)` 处返回 `new Promise(…)` 。一秒钟之后执行成功，然后他的结果（ `resolve` 的参数，在这儿就是 `result*2`）被传递给在 `(**)` 的第二个 `.then`。他会显示 `2`，然后做了相同的事情。

返回的结果依然是 1 -> 2 > 4，但是每个 `alert` 之间多了1秒的延迟。

返回 promises 创建异步动作的链式操作。

## 例子: loadScript

让我们用上面的特性来让 `loadScript` 可以按照顺序一个一个的载入新脚本文件：

```js run
loadScript("/article/promise-chaining/one.js")
  .then(function(script) {
    return loadScript("/article/promise-chaining/two.js");
  })
  .then(function(script) {
    return loadScript("/article/promise-chaining/three.js");
  })
  .then(function(script) {
    // 使用载入的脚本文件里面定义的函数
    // 显示它们确实已经载入成功了
    one();
    two();
    three();
  });
```

在上面的代码中，每一个 `loadScript` 执行完成都会返回一个 promise，当 它执行完成以后，下一个 `.then` 就会被执行。然后开始加载一个脚本文件。所有的脚本文件都会按照顺序一个接一个加载。

我们可以给链式操作添加更多的异步动作。请注意，代码依然是扁平的，它会向下发展而不是向右。妈妈再也不用担心我会写出 "恶魔金字塔" 那样的代码啦。

请注意，技术上来说也可以直接在每个 promise 后面写 `.then` 而不是返回他们，就像这样：

```js run
});
loadScript("/article/promise-chaining/one.js").then(function(script1) {
  loadScript("/article/promise-chaining/two.js").then(function(script2) {
    loadScript("/article/promise-chaining/three.js").then(function(script3) {
      // 这个函数可以使用变量 script1, script2 and script3
      one();
      two();
      three();
    });
  });
```

上面的代码结果是一样的：按照顺序载入三个脚本文件。但是代码是 "向右发展". 这就和我们在回调那一章遇到的问题一样了。使用链式操作（在 `.then` 内返回 promise）可以有效的解决这种问题。

有些时候直接写 `.then` 也是可以的，因为内层的函数可以访问外层的变量（上面的例子中，最内层的函数可以访问所有的 `scriptX`），所以编写代码的时候还是得具体情况具体分析。

值得庆幸的是，`.then` 可以返回任何的 "thenable" 对象，它会被像 promise 一样对待。

任何使用 `.then` 的对象都是 "thenable" 对象。

第三方库可以设计它自己的 "可兼容 promise " 对象。他们可以扩展设置方法，同时可以兼容原生的 promises ，因为他们使用了 `.then`。

下面是一个 thenable 对象的例子：

```js run
class Thenable {
  constructor(num) {
    this.num = num;
  }
  then(resolve, reject) {
    alert(resolve); // function() { native code }
    // 1秒以后成功，并返回 this.num*2 
    setTimeout(() => resolve(this.num * 2), 1000); // (**)
  }
}

new Promise(resolve => resolve(1))
  .then(result => {
    return new Thenable(result); // (*)
  })
  .then(alert); // 1000ms 以后显示2
```

JavaScript 会检查 `.then` 在 `(*)` 处返回的对象：如果它有一个被叫做 `then` 的可执行方法，然后它就会执行这个方法，并且它会提供原生的 `resolve`, `reject` 函数作为参数。在上面的例子中，在  `(**)`处的 `resolve(2)` 在1秒以后被执行。然后结果被传递给更深的链。

这个特性允许我们不需要继承  `Promise` 就可以直接整合定制的对象到 promise 链。



## 更大的例子：fetch

在前端代码中，promises 通常被用来做网络请求。我们现在就来看一个这样的例子。

我们会使用 [fetch](mdn:api/WindowOrWorkerGlobalScope/fetch) 方法从远程服务器载入用户的信息。这个方法很复杂，它有很多可选的参数，但是基本使用非常简单：

```js
let promise = fetch(url);
```

上面的代码会向 `url` 发起一个网络请求，然后返回一个 promise 。当远程服务器返回头部响应之后，*在整个响应被下载之前*，这个 promise 会被置为解决并返回一个  `response` 对象。

我们可以执行 `response.text()` 方法来获取整个响应：当从远程服务器下载完完整的文本后，它会返回一个解决的 promise。

下面的代码向 `user.json` 发出一个请求，然后载入它的文本：

```js run
fetch('/article/promise-chaining/user.json')
  // 下面的 .then 会在远程服务器响应之后运行
  .then(function(response) {
    // response.text() 会返回一个新的，会返回完整文本的 promise
    // 当我们完成以后就下载它
    return response.text();
  })
  .then(function(text) {
    // ...这就是远程文件的内容
    alert(text); // {"name": "iliakan", isAdmin: true}
  });
```

`response.json()` 方法会读取远程数据然后把它转换成 JSON 格式。这对我们来说更方便，所以现在让我们换成它试试。

为了更简洁，我们将使用箭头函数：

```js run
// 基本和上面一样，但是 response.json() 会将远程内容转换成 JSON 格式
fetch('/article/promise-chaining/user.json')
  .then(response => response.json())
  .then(user => alert(user.name)); // iliakan
```

现在让我们用已经载入的用户信息搞点事情。

例如，我们可以再向 github 发起一个请求，加载用户的简介然后展示其头像：

```js run
// 向 user.json 发起一个请求
fetch('/article/promise-chaining/user.json')
  // 载入 json 格式的数据
  .then(response => response.json())
  // 向 github 发起一个请求
  .then(user => fetch(`https://api.github.com/users/${user.name}`))
  // 载入 json 格式的数据
  .then(response => response.json())
  // 展示头像图片（githubUser.avatar_url）3秒钟（也许可以搞个动画）
  .then(githubUser => {
    let img = document.createElement('img');
    img.src = githubUser.avatar_url;
    img.className = "promise-avatar-example";
    document.body.append(img);

    setTimeout(() => img.remove(), 3000); // (*)
  });
```

上面的代码运行起来没毛病，看看备注就很容易看懂。但是它存在一个 promises 初学者非常喜欢犯的问题。

注意观察 `(*)` 处：如果我们想在头像展示和移除 *以后* 干点啥，该怎样实现？例如，我们想展示一个可以编辑用户信息的表单。就目前来说，臣妾做不到啊。

为了将这个链变成可扩展的，我们需要返回一个 promise，它会在在头像展示完成之后置为解决。

就像这样：

```js run
fetch('/article/promise-chaining/user.json')
  .then(response => response.json())
  .then(user => fetch(`https://api.github.com/users/${user.name}`))
  .then(response => response.json())
*!*
  .then(githubUser => new Promise(function(resolve, reject) {
*/!*
    let img = document.createElement('img');
    img.src = githubUser.avatar_url;
    img.className = "promise-avatar-example";
    document.body.append(img);

    setTimeout(() => {
      img.remove();
*!*
      resolve(githubUser);
*/!*
    }, 3000);
  }))
  // 3秒之后触发
  .then(githubUser => alert(`Finished showing ${githubUser.name}`));
```

现在，当 `setTimeout` 运行 `img.remove()` 后，它会执行 `resolve(githubUser)`，因此将控制权传递给下一个链中的 `.then`，同时也会传递用户的数据。

一般来说，异步动作应该都返回一个 promise。

这让扩展计划任务变为可能。即时我们现在没有扩展链的计划，我们之后可能会需要它。

最后，我们将代码分成可以重复使用的函数：

```js run
function loadJson(url) {
  return fetch(url)
    .then(response => response.json());
}

function loadGithubUser(name) {
  return fetch(`https://api.github.com/users/${name}`)
    .then(response => response.json());
}

function showAvatar(githubUser) {
  return new Promise(function(resolve, reject) {
    let img = document.createElement('img');
    img.src = githubUser.avatar_url;
    img.className = "promise-avatar-example";
    document.body.append(img);

    setTimeout(() => {
      img.remove();
      resolve(githubUser);
    }, 3000);
  });
}

// 使用它们
loadJson('/article/promise-chaining/user.json')
  .then(user => loadGithubUser(user.name))
  .then(showAvatar)
  .then(githubUser => alert(`Finished showing ${githubUser.name}`));
  // ...
```

## Error handling

Asynchronous actions may sometimes fail: in case of an error the corresponding promises becomes rejected. For instance, `fetch` fails if the remote server is not available. We can use `.catch` to handle errors (rejections).

Promise chaining is great at that aspect. When a promise rejects, the control jumps to the closest rejection handler down the chain. That's very convenient in practice.

For instance, in the code below the URL is wrong (no such server) and `.catch` handles the error:

```js run
*!*
fetch('https://no-such-server.blabla') // rejects
*/!*
  .then(response => response.json())
  .catch(err => alert(err)) // TypeError: failed to fetch (the text may vary)
```

Or, maybe, everything is all right with the server, but the response is not a valid JSON:

```js run
fetch('/') // fetch works fine now, the server responds successfully
*!*
  .then(response => response.json()) // rejects: the page is HTML, not a valid json
*/!*
  .catch(err => alert(err)) // SyntaxError: Unexpected token < in JSON at position 0
```


In the example below we append `.catch` to handle all errors in the avatar-loading-and-showing chain:

```js run
fetch('/article/promise-chaining/user.json')
  .then(response => response.json())
  .then(user => fetch(`https://api.github.com/users/${user.name}`))
  .then(response => response.json())
  .then(githubUser => new Promise(function(resolve, reject) {
    let img = document.createElement('img');
    img.src = githubUser.avatar_url;
    img.className = "promise-avatar-example";
    document.body.append(img);

    setTimeout(() => {
      img.remove();
      resolve(githubUser);
    }, 3000);
  }))
  .catch(error => alert(error.message));
```

Here `.catch` doesn't trigger at all, because there are no errors. But if any of the promises above rejects, then it would execute.

## Implicit try..catch

The code of the executor and promise handlers has an "invisible `try..catch`" around it. If an error happens, it gets caught and treated as a rejection.

For instance, this code:

```js run
new Promise(function(resolve, reject) {
*!*
  throw new Error("Whoops!");
*/!*
}).catch(alert); // Error: Whoops!
```

...Works the same way as this:

```js run
new Promise(function(resolve, reject) {
*!*
  reject(new Error("Whoops!"));
*/!*  
}).catch(alert); // Error: Whoops!
```

The "invisible `try..catch`" around the executor automatically catches the error and treats it as a rejection.

That's so not only in the executor, but in handlers as well. If we `throw` inside `.then` handler, that means a rejected promise, so the control jumps to the nearest error handler.

Here's an example:

```js run
new Promise(function(resolve, reject) {
  resolve("ok");
}).then(function(result) {
*!*
  throw new Error("Whoops!"); // rejects the promise
*/!*
}).catch(alert); // Error: Whoops!
```

That's so not only for `throw`, but for any errors, including programming errors as well:

```js run
new Promise(function(resolve, reject) {
  resolve("ok");
}).then(function(result) {
*!*
  blabla(); // no such function
*/!*
}).catch(alert); // ReferenceError: blabla is not defined
```

As a side effect, the final `.catch` not only catches explicit rejections, but also occasional errors in the handlers above.

## Rethrowing

As we already noticed, `.catch` behaves like `try..catch`. We may have as many `.then` as we want, and then use a single `.catch` at the end to handle errors in all of them.

In a regular `try..catch` we can analyze the error and maybe rethrow it if can't handle. The same thing is possible for promises. If we `throw` inside `.catch`, then the control goes to the next closest error handler. And if we handle the error and finish normally, then it continues to the closest successful `.then` handler.

In the example below the `.catch` successfully handles the error:
```js run
// the execution: catch -> then
new Promise(function(resolve, reject) {

  throw new Error("Whoops!");

}).catch(function(error) {

  alert("The error is handled, continue normally");

}).then(() => alert("Next successful handler runs"));
```

Here the `.catch` block finishes normally. So the next successful handler is called. Or it could return something, that would be the same.

...And here the `.catch` block analyzes the error and throws it again:

```js run
// the execution: catch -> catch -> then
new Promise(function(resolve, reject) {

  throw new Error("Whoops!");

}).catch(function(error) { // (*)

  if (error instanceof URIError) {
    // handle it
  } else {
    alert("Can't handle such error");

*!*
    throw error; // throwing this or another error jumps to the next catch
*/!*
  }

}).then(function() {
  /* never runs here */
}).catch(error => { // (**)

  alert(`The unknown error has occured: ${error}`);
  // don't return anything => execution goes the normal way

});
```

The handler `(*)` catches the error and just can't handle it, because it's not `URIError`, so it throws it again. Then the execution jumps to the next `.catch` down the chain `(**)`.

In the section below we'll see a practical example of rethrowing.

## Fetch error handling example

Let's improve error handling for the user-loading example.

The promise returned by [fetch](mdn:api/WindowOrWorkerGlobalScope/fetch) rejects when it's impossible to make a request. For instance, a remote server is not available, or the URL is malformed. But if the remote server responds with error 404, or even error 500, then it's considered a valid response.

What if the server returns a non-JSON page with error 500 in the line `(*)`? What if there's no such user, and github returns a page with error 404 at `(**)`?

```js run
fetch('no-such-user.json') // (*)
  .then(response => response.json())
  .then(user => fetch(`https://api.github.com/users/${user.name}`)) // (**)
  .then(response => response.json())
  .catch(alert); // SyntaxError: Unexpected token < in JSON at position 0
  // ...
```


As of now, the code tries to load the response as JSON no matter what and dies with a syntax error. You can see that by running the example above, as the file `no-such-user.json` doesn't exist.

That's not good, because the error just falls through the chain, without details: what failed and where.

So let's add one more step: we should check the `response.status` property that has HTTP status, and if it's not 200, then throw an error.

```js run
class HttpError extends Error { // (1)
  constructor(response) {
    super(`${response.status} for ${response.url}`);
    this.name = 'HttpError';
    this.response = response;
  }
}

function loadJson(url) { // (2)
  return fetch(url)
    .then(response => {
      if (response.status == 200) {
        return response.json();
      } else {
        throw new HttpError(response);
      }
    })
}

loadJson('no-such-user.json') // (3)
  .catch(alert); // HttpError: 404 for .../no-such-user.json
```

1. We make a custom class for HTTP Errors to distinguish them from other types of errors. Besides, the new class has a constructor that accepts the `response` object and saves it in the error. So error-handling code will be able to access it.
2. Then we put together the requesting and error-handling code into a function that fetches the `url` *and* treats any non-200 status as an error. That's convenient, because we often need such logic.
3. Now `alert` shows better message.

The great thing about having our own class for errors is that we can easily check for it in error-handling code.

For instance, we can make a request, and then if we get 404 -- ask the user to modify the information.

The code below loads a user with the given name from github. If there's no such user, then it asks for the correct name:

```js run
function demoGithubUser() {
  let name = prompt("Enter a name?", "iliakan");

  return loadJson(`https://api.github.com/users/${name}`)
    .then(user => {
      alert(`Full name: ${user.name}.`); // (1)
      return user;
    })
    .catch(err => {
*!*
      if (err instanceof HttpError && err.response.status == 404) { // (2)
*/!*
        alert("No such user, please reenter.");
        return demoGithubUser();
      } else {
        throw err;
      }
    });
}

demoGithubUser();
```

Here:

1. If `loadJson` returns a valid user object, then the name is shown `(1)`, and the user is returned, so that we can add more user-related actions to the chain. In that case the `.catch` below is ignored, everything's very simple and fine.
2. Otherwise, in case of an error, we check it in the line `(2)`. Only if it's indeed the HTTP error, and the status is 404 (Not found), we ask the user to reenter. For other errors -- we don't know how to handle, so we just rethrow them.

## Unhandled rejections

What happens when an error is not handled? For instance, after the rethrow as in the example above. Or if we forget to append an error handler to the end of the chain, like here:

```js untrusted run refresh
new Promise(function() {
  noSuchFunction(); // Error here (no such function)
});
```

Or here:

```js untrusted run refresh
new Promise(function() {
  throw new Error("Whoops!");
}).then(function() {
  // ...something...
}).then(function() {
  // ...something else...
}).then(function() {
  // ...but no catch after it!
});
```

In theory, nothing should happen. In case of an error happens, the promise state becomes "rejected", and the execution should jump to the closest rejection handler. But there is no such handler in the examples above. So the error gets "stuck".

In practice, that means that the code is bad. Indeed, how come that there's no error handling?

Most JavaScript engines track such situations and generate a global error in that case. We can see it in the console.

In the browser we can catch it using the event `unhandledrejection`:

```js run
*!*
window.addEventListener('unhandledrejection', function(event) {
  // the event object has two special properties:
  alert(event.promise); // [object Promise] - the promise that generated the error
  alert(event.reason); // Error: Whoops! - the unhandled error object
});
*/!*

new Promise(function() {
  throw new Error("Whoops!");
}); // no catch to handle the error
```

The event is the part of the [HTML standard](https://html.spec.whatwg.org/multipage/webappapis.html#unhandled-promise-rejections). Now if an error occurs, and there's no `.catch`, the `unhandledrejection` handler triggers: the `event` object has the information about the error, so we can do something with it.

Usually such errors are unrecoverable, so our best way out is to inform the user about the problem and probably report about the incident to the server.

In non-browser environments like Node.JS there are other similar ways to track unhandled errors.

## Summary

To summarize, `.then/catch(handler)` returns a new promise that changes depending on what handler does:

1. If it returns a value or finishes without a `return` (same as `return undefined`), then the new promise becomes resolved, and the closest resolve handler (the first argument of `.then`) is called with that value.
2. If it throws an error, then the new promise becomes rejected, and the closest rejection handler (second argument of `.then` or `.catch`) is called with it.
3. If it returns a promise, then JavaScript waits until it settles and then acts on its outcome the same way.

The picture of how the promise returned by `.then/catch` changes:

![](promise-handler-variants.png)

The smaller picture of how handlers are called:

![](promise-handler-variants-2.png)

In the examples of error handling above the `.catch` was always the last in the chain. In practice though, not every promise chain has a `.catch`. Just like regular code is not always wrapped in `try..catch`.

We should place `.catch` exactly in the places where we want to handle errors and know how to handle them. Using custom error classes can help to analyze errors and rethrow those that we can't handle.

For errors that fall outside of our scope we should have the `unhandledrejection` event handler (for browsers, and analogs for other environments). Such unknown errors are usually unrecoverable, so all we should do is to inform the user and probably report to our server about the incident.
