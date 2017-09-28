
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

## 错误处理

异步操作有时候可能失败：当发生错误的时候，对应的 promises 就会变成拒绝状态。例如，`fetch` 在远程服务器宕机的时候就会失败。这种时候我们可以用 `.catch` 来处理错误。

Promise 链在处理这种情况也是棒棒哒。当一个 promise 返回拒绝，控制权会传递给之后最近的拒绝处理者。这在实际应用中实在是太方便啦。

例如，下面代码中的 URL 是错的（不存在这样的服务器），那么 `.catch` 就会处理这个错误：

```js run
*!*
fetch('https://no-such-server.blabla') // 拒绝
*/!*
  .then(response => response.json())
  .catch(err => alert(err)) // TypeError: fetch 失败（文字可能不同）
```

或者，就算服务器没有一毛钱问题，但是返回值不是合法的 JSON 数据格式：

```js run
fetch('/') // fetch 现在可以正常工作，服务器有响应
*!*
  .then(response => response.json()) // 拒绝：这个页面是 HTML，不是一个合法的 JSON
*/!*
  .catch(err => alert(err)) // SyntaxError: Unexpected token < in JSON at position 0 （这个都看不懂，就太尴尬了）
```


在下面的例子中，我们在「头像加载并且展示」链的最后追加了 `.catch` 处理所有的错误：

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

上面的 `.catch` 并不会被触发，因为没有错误。但是。如果任意一个 promise 返回拒绝，`.catch` 就会被执行了。

## 隐形的 try..catch

在 promise 被处理或者执行的时候，会有一个 "隐形的 `try..catch`"。如果发生了错误，它就会被当成拒绝触发。

例如下面的代码：

```js run
new Promise(function(resolve, reject) {
*!*
  throw new Error("Whoops!");
*/!*
}).catch(alert); // Error: Whoops!
```

...下面也一样：

```js run
new Promise(function(resolve, reject) {
*!*
  reject(new Error("Whoops!"));
*/!*  
}).catch(alert); // Error: Whoops!
```

执行者的 "隐形的 `try..catch`" 会自动捕获错误然后当成拒绝触发。

和执行者一样，处理者也存在 "隐形的 `try..catch`"。如果我们在  `.then` 内 `throw` 错误，这表示一个拒绝的 promise，所以控制权会给到最近的错误处理者。

下面是一个例子：

```js run
new Promise(function(resolve, reject) {
  resolve("ok");
}).then(function(result) {
*!*
  throw new Error("Whoops!"); // 拒绝 promise
*/!*
}).catch(alert); // Error: Whoops!
```

不仅仅是 `throw` 的错误，包括编程错误在内的错误都会被捕获到：

```js run
new Promise(function(resolve, reject) {
  resolve("ok");
}).then(function(result) {
*!*
  blabla(); // 不存在的函数
*/!*
}).catch(alert); // ReferenceError: blabla is not defined
```

作为副作用，最终的 `.catch` 不仅会捕获到明确的拒绝，同时也偶尔捕获之前的处理者的错误。

## 重新抛出

我们已经知道了，`.catch` 和 `try..catch` 一样。我可以写多个 `.then` ，然后只在最后写一个 `.catch` 来处理所有的错误。

在正常的 `try..catch` 中，我们可以分析错误，如果无法处理的话我们会把它重新抛出。在 promises 我们当然也可以这样做。如果我们在  `.then` 内 `throw` 错误，控制权会给到最近的错误处理者。如果我们正常处理错误，它会继续执行最近的成功的 `.then` 处理者。

在下面的例子中，`.catch` 成功测处理了错误：
```js run
// 执行过程： catch -> then
new Promise(function(resolve, reject) {

  throw new Error("Whoops!");

}).catch(function(error) {

  alert("错误被正常处理，继续正常执行");

}).then(() => alert("下一个成功的处理者被执行"));
```

这里的 `.catch` 正常完成。所以下一个成功的处理者被执行。或者它会返回一些东西，没什么却别。

...下面的 `.catch` 分析错误，然后再次抛出它：

```js run
// 执行过程： catch -> catch -> then
new Promise(function(resolve, reject) {

  throw new Error("Whoops!");

}).catch(function(error) { // (*)

  if (error instanceof URIError) {
    // 处理它
  } else {
    alert("不能处理这样的错误");

*!*
    throw error; // 抛出这个或者其他错误，跳到下一个 catch
*/!*
  }

}).then(function() {
  /* 永远不会执行到这儿 */
}).catch(error => { // (**)

  alert(`发生未知错误： ${error}`);
  // 不会返回任何事情 => 正常流程执行

});
```

在 `(*)` 出的处理者捕获到错误，但是不能处理它，因为错误不是 `URIError`，所以它再次爆出它。然后程序会跳到在 `(**)` 处的链的下一个 `.catch`。

In the section below we'll see a practical example of rethrowing.

## Fetch 错误处理例子

现在让我们给上面的用户加载的例子添加错误处理。

[fetch](mdn:api/WindowOrWorkerGlobalScope/fetch) 返回的 promise 会在请求无法完成的时候拒绝。例如，远程服务器挂了，或者不正确的 URL。但是如果服务器响应了404甚至505错误，然后它会被认为是有效的响应。

如果服务器在 `(*)` 处返回的是一个500错误的非 JSON 页面会怎样? 如果在 `(**)` 处，根本没有这样的用户， 然后 github 返回了一个404错误的页面又会怎样？

```js run
fetch('no-such-user.json') // (*)
  .then(response => response.json())
  .then(user => fetch(`https://api.github.com/users/${user.name}`)) // (**)
  .then(response => response.json())
  .catch(alert); // SyntaxError: Unexpected token < in JSON at position 0
  // ...
```


目前来说，不管服务器是挂了还是返回任何东西，我们的代码都会尝试将反应当做 JSON 对待。运行上面的代码你就会发现问题，因为 `no-such-user.json` 根本不存在。

这就有问题了，因为错误通过链流走了，没有任何详细信息：发送了什么错误，哪儿发生的错误？

所以，让我们再多加一步来加以完善：我们应该要检查拥有 HTTP 状态的 `response.status` 属性，如果不是200，那就抛出一个错误。

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

1. 我们写了一个自定义 class 来区别 HTTP 错误和其他类型的错误。另外，这个 class 的构造函数接受 `response` 对象作为参数，然后把它保存进错误。所以错误处理代码可以进入它。
2. 然后，我们把请求和错误代码放进一个函数，这个函数会请求 `url` *并且* 将任何非200状态当成错误。这就很有灵性了：因为我们经常会需要这样的逻辑处理。
3. 现在 `alert` 会展示更好的信息。

写一个自己的错误处理 class 的好处是，我们很容易在错误处理代码中检查它。

例如，我们发起一个请求，如果我们接到404返回 -- 就让用户修改信息。

下面的代码中，会先让用户输入一个用户名，然后加载他的 github 页面。如果输入的用户不存在，则代码会要求用户再次输入：

```js run
function demoGithubUser() {
  let name = prompt("请输入用户名", "iliakan");

  return loadJson(`https://api.github.com/users/${name}`)
    .then(user => {
      alert(`Full name: ${user.name}.`); // (1)
      return user;
    })
    .catch(err => {
*!*
      if (err instanceof HttpError && err.response.status == 404) { // (2)
*/!*
        alert("用户不存在，请再次输入。");
        return demoGithubUser();
      } else {
        throw err;
      }
    });
}

demoGithubUser();
```

解释一下：

1. 如果 `loadJson` 返回一个合法的用户对象，在 `(1)` 处会显示出用户的名字，同时返回该用户，然后我们可以在链中添加更多的跟用户有关的操作。在这样的情况下，之后的 `.catch` 就会被忽略，一切都那么简单美好。
2. 否则，如果发生了一个错误，我们会在检查它 `(2)` 。只有是 HTTP 错误并且状态是404的时候，我们会让用户再次输入。至于其他的错误 -- 我们不知道怎样处理他， 所以我们就重新抛出它。

## 没有被处理的拒绝

如果一个错误没有被处理会发生什么呢？例如，在上面例子中重新抛出之后。或者，如果我们忘记在链的最后加上错误处理，就像这样：

```js untrusted run refresh
new Promise(function() {
  noSuchFunction(); // 错误（没有这个函数）
});
```

或者这样：

```js untrusted run refresh
new Promise(function() {
  throw new Error("Whoops!");
}).then(function() {
  // ...something...
}).then(function() {
  // ...something else...
}).then(function() {
  // ...之后没有 catch
});
```

理论上来讲，什么都不会发生。如果发生了一个错误，promise 的状态就会变成 "已拒绝"，然后程序就会跳到最近的拒绝处理者那里。但是如果跟上面的代码一样，没有这样的错误处理者的话，错误就被卡住。

在实际中，这样的代码就很差了。的确，好的代码怎么可能没有错误处理呢？

大部门的 JavaScript 引擎会追踪这种情况，然后生成一个全局的错误。我们可以在 console 中打印出来。

在浏览器中，我们可以通过事件 `unhandledrejection` 捕获它：

```js run
*!*
window.addEventListener('unhandledrejection', function(event) {
  // 这个事件对象有两个特殊的属性：
  alert(event.promise); // [object Promise] - 生成错误的 promise 
  alert(event.reason); // Error: Whoops! - 没有被处理的错误对象object
});
*/!*

new Promise(function() {
  throw new Error("Whoops!");
}); // 没有 catch 来处理错误
```

该事件是[HTML standard](https://html.spec.whatwg.org/multipage/webappapis.html#unhandled-promise-rejections)的一部分。现在，如果发生了一个错误，并且没有 `.catch` 来处理它，那么 `unhandledrejection` 就会被触发：这个 `event` 对象包含关于错误的信息，那我们就可以利用它搞事情了。

一般来说，这样的错误是无法回收的，所以我们最好的处理方式是，将错误告诉给用户，同时也尽可能报告给服务器。

在像 Node.JS 这样的非浏览器环境下会有一些其他的类似的方法来追中没有被处理的错误。

## 总结

总而言之，`.then/catch(handler)` 会根据错误处理者的不用返回不用的新 promise：

1. 如果它返回一个值或者没有 `return` 就完成了（和 `return undefined` 一样），那么新的 promise 就变成已解决状态，最近的解决处理者（`.then` 的第一个参数）被执行。
2. 如果它抛出一个错误，那么新的 promise 就变成已拒绝状态，最近的拒绝处理者（`.then` 的第二个参数）被执行。
3. 如果它返回一个 promise，那么 JavaScript 会等到它再次执行完成然后根据它的返回值做类似上面的处理。

下面的图片展示了 `.then/catch` 返回的 promise 是如何改变的：

![](promise-handler-variants.png)

简洁一点的图：

![](promise-handler-variants-2.png)

在上面的错误处理的例子中，`.catch` 通常都会被放到链的最后。在实际应用中，并不是每一个 promise 链都有 `.catch`。就像平常代码也不全都会被 `try..catch` 包裹一样。

我应该把 `.catch` 放在我们正好需要处理错误的地方。使用定制的错误处理 class 能帮助我们分析错误，然后将我们不能处理的错误再次抛出。

我们应该用 `unhandledrejection` 事件去处理那些在我们处理范围之外的错误（就浏览器环境而言，其他环境有类似的方法）。这样的错误通常是无法回收的，所以我们最好的处理方式是，将错误告诉给用户，同时也尽可能报告给服务器。
