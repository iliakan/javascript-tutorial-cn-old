

# 回调简介

Javascript 中的很多操作都是 **异步**。

例如下面的这个 `loadScript(src)` 函数：

```js
function loadScript(src) {
  let script = document.createElement('script');
  script.src = src;
  document.head.append(script);
}
```

这个函数的功能是载入新的 Javascript 文件。 当它成功将 `<script src="…">` 添加到文档以后，浏览器就会加载然后执行新的 Javascript 文件。

我们可以这样使用它:

```js
// 加载然后执行新的 Javascript 文件
loadScript('/my/script.js');
```

我们之所以叫这个函数为异步函数，是因为这个函数无法马上执行完成而是需要等待文件载入成功。

这个函数首先是加载文件，然后执行文件。当文件在加载的时候，函数后面的代码可能会已经执行完成，在加载文件的同时，其它的 Javascript 文件也会执行。

```js
loadScript('/my/script.js');
// 之后的代码不会等待文件的载入完成
```

现在假设我们希望使用上面的函数，在新的 Javascript 文件载入完成以后使用它所定义的新函数。

...如果我们在 `loadScript(…)` 函数之后想马上使用新载入的函数，就会出现问题：

```js
loadScript('/my/script.js'); // 这个文件中定义了一个函数 "function newFunction() {…}"

*!*
newFunction(); // 不存在滴！
*/!*
```

一般而言，浏览器还没有时间去载入 Javascript 文件。就目前而言，`loadScript` 函数没有提供任何手段让我们去追踪载入是否完成。它所能做的只是载入文件，然后执行。但是，我们需要知道载入文件什么时候完成，我们什么时候能使用载入文件里面的新函数和变量。

现在让我们给 `loadScript` 添加一个 `回调` 函数参数， `回调` 函数会在载入完成以后执行：

```js
function loadScript(src, *!*callback*/!*) {
  let script = document.createElement('script');
  script.src = src;

*!*
  script.onload = () => callback(script);
*/!*

  document.head.append(script);
}
```

现在如果我们想使用新文件里面的函数，把它写在回调函数里面就可以啦：

```js
loadScript('/my/script.js', function() {
  // 回调函数会在载入完成以后执行
  newFunction(); // 成功啦!
  ...
});
```

这就是我们的解决方案：第二个参数是一个函数 (通常是匿名函数)，会在动作执行完成之后执行。

下面是一段可以执行的例子：

```js
function loadScript(src, callback) {
  let script = document.createElement('script');
  script.src = src;
  script.onload = () => callback(script);
  document.head.append(script);
}

*!*
loadScript('https://cdnjs.cloudflare.com/ajax/libs/lodash.js/3.2.0/lodash.js', script => {
  alert(`Cool, the ${script.src} is loaded`);
  alert( _ ); // _ 函数在刚才载入的文件中定义
});
*/!*
```

这种方式通常被称为基于回调函数的异步编程。异步函数应该提供一个回调函数参数，在异步动作执行完成以后我们会执行回调函数。

这就是我们在 `loadScript` 函数中所做的事情，很明显，这是非常一般的解决方案。

## 回调中再回调

怎么样按照顺序载入两个文件：先载入第一个然后载入第二个？

最容易想到的解决方案是在 `loadScript` 的回调函数里面再次执行 `loadScript`：

```js
loadScript('/my/script.js', function(script) {

  alert(`Cool, the ${script.src} is loaded, let's load one more`);

*!*
  loadScript('/my/script2.js', function(script) {
    alert(`Cool, the second script is loaded`);
  });
*/!*

});
```

等待外层的 `loadScript` 执行完成以后，回调函数执行内层的 `loadScript`，看起来还不错，没毛病。

...但是如果有人就是要刁难我胖虎，一定要载入很多个文件呢？

```js
loadScript('/my/script.js', function(script) {

  loadScript('/my/script2.js', function(script) {

*!*
    loadScript('/my/script3.js', function(script) {
      // ...胖虎已经感觉很难受了
    });
*/!*

  })

});
```

我们必须把代码写进一个又一个的回调函数里面。当层级多了以后，代码就会显得臃肿难看了，我们很快会讨论新的解决方案解决这个问题。

## 错误处理

在上面的代码中，我们没有考虑过报错的问题。如果载入报错会怎样？我们的回调函数应该足够健壮，有能力去处理错误。

下面是一个改进版的，可以处理错误的 `loadScript` 函数：

```js
function loadScript(src, callback) {
  let script = document.createElement('script');
  script.src = src;

*!*
  script.onload = () => callback(null, script);
  script.onerror = () => callback(new Error(`Script load error ` + src));
*/!*

  document.head.append(script);
}
```

载入成功后会执行 `callback(null, script)`，否则会执行 `callback(error)`。

使用方法：
```js
loadScript('/my/script.js', function(error, script) {
  if (error) {
    // 错误处理
  } else {
    // 载入成功
  }
});
```

上面的代码是非常常见的开发模式。它被称为 "error-first callback"。

按照约定俗成的规则：
1. `回调` 的第一个参数预订为错误处理，如果出现错误了就执行 `callback(err)`。
2. 第二个参数（和剩下的，如果有的话）用来处理执行成功之后。然后 `callback(null, result1, result2…)` 被执行。

用单个 `回调` 函数来处理错误和成功。

## 恶魔金字塔

`回调` 看起来是一个可行的异步编程方案。实际上回调层级很少的时候，`回调` 确实是一个不错的解决方案。 

但是当回调层级过多的时候，我们的代码就会非常臃肿难看：

```js
loadScript('1.js', function(error, script) {

  if (error) {
    handleError(error);
  } else {
    // ...
    loadScript('2.js', function(error, script) {
      if (error) {
        handleError(error);
      } else {
        // ...
        loadScript('3.js', function(error, script) {
          if (error) {
            handleError(error);
          } else {
  *!*
            // ...直到世界的尽头 (*)
  */!*
          }
        });

      }
    })
  }
});
```

在上面的代码中：
1. 载入 `1.js`，如果没有出错的话然后继续。
2. 载入 `2.js`，如果没有出错的话然后继续。
3. 载入 `3.js`，如果没有出错的话 -- 继续执行回调 `(*)`。

当层级越来越多了以后，代码变得越来越深，非常难以管理，在实际写代码的时候会比上面的代码更加难以管理， 因为实际代码中会有各种循环，条件控制，而不是简单的 `...`。

我们通常把这种情况叫做 "回调地狱" 或者 "恶魔金字塔"。

![](callback-hell.png)

每一个异步操作都会让"金字塔"的层级变大。很快它就会膨胀到无法控制。

所以这种代码编写方式并不是特别好。

为了减轻问题，我们可以给每个异步操作定义一个单独的函数，就像下面这样：

```js
loadScript('1.js', step1);

function step1(error, script) {
  if (error) {
    handleError(error);
  } else {
    // ...
    loadScript('2.js', step2);
  }
}

function step2(error, script) {
  if (error) {
    handleError(error);
  } else {
    // ...
    loadScript('3.js', step3);
  }
}

function step3(error, script) {
  if (error) {
    handleError(error);
  } else {
    // ...所有脚本全部加载完成后继续 (*)
  }
};
```

看见了吧？它做了同样的事情，同时也没有了层级，因为我们给每个操作都定义了一个单独的顶层函数。

它确实解决了我们的一些问题，但是我们的代码却变得跟撕碎的 excel 表格似的。也许你已经察觉到了，它非常难以阅读。人们在阅读代码的时候需要去寻找所定义的函数在哪儿。这是非常不方便的，特别是当读者对代码不了解的时候，他们不知道去哪儿找到对应的函数。

同时，那些叫 `step*` 的函数都只是一次性使用，定义它们只是为了减轻 `恶魔金字塔`。没有人会在回调操作链之外使用到它们。所以它们会污染命名空间。

很明显，我们需要更好的解决方案。

幸运的是，我们有一些其它的方法来解决臃肿的金字塔。其中最好的方法是使用 "promises"，我们会在下一章节讨论它。
