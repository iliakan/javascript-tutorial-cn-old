# 错误处理，“try..catch”

无论我们多么擅长编程，有时候我们的脚本会有错误。错误的发生可能是由于我们的失误，或者意外的用户输入，或者一个错误的服务器响应等成千上万种原因。

通常，出现错误的时候脚本会“挂掉”（立即终止），将错误输出至控制台。

但有一个语法结构 `try..catch` 允许“捕获”错误并做一些更合理的处理而不是挂掉。

[cut]

## “try..catch”语法

`try..catch` 结构有两个主要的块：`try` 和 `catch`：

```js
try {

  // code...

} catch (err) {

  // error handling

}
```

它是这样工作的：

1. 首先，`try {...}` 里的代码被执行。
2. 如果没有错误，那么 `catch(err)` 被忽略：执行到 `try` 末尾并跳过 `catch`。
3. 如果有错误发生，那么 `try` 的执行被终止，控制流转至 `catch(err)` 的开始。`err` 变量（可以使用任何名称）包含了一个错误对象，其中有关于错误发生的详细信息。

![](try-catch-flow.png)

所以，`try {...}` 块里的错误并不会终止脚本：我们有机会在 `catch` 中处理它。

让我们看更多的例子。

- 一个没有错误的例子：显示 `alert` `(1)` 和 `(2)`：

    ```js run
    try {

      alert('Start of try runs');  // *!*(1) <--*/!*

      // ...no errors here

      alert('End of try runs');   // *!*(2) <--*/!*

    } catch(err) {

      alert('Catch is ignored, because there are no errors'); // (3)

    }

    alert("...Then the execution continues");
    ```
- 一个有错误的例子：显示 `(1)` 和 `(3)`：

    ```js run
    try {

      alert('Start of try runs');  // *!*(1) <--*/!*

    *!*
      lalala; // error, variable is not defined!
    */!*

      alert('End of try (never reached)');  // (2)

    } catch(err) {

      alert(`Error has occured!`); // *!*(3) <--*/!*

    }

    alert("...Then the execution continues");
    ```


````warn header="`try..catch` 仅对运行时错误有效"
要想 `try..catch` 起作用，代码必须是可运行的。换句话说，它应该是有效的 JavaScript。

如果代码里有语法错误，它是不会起作用的，例如代码里有不匹配的花括号：

```js run
try {
  {{{{{{{{{{{{
} catch(e) {
  alert("The engine can't understand this code, it's invalid");
}
```

JavaScript 引擎首先读取代码，然后运行它。发生在读取阶段的错误被称为“解析时”错误并且是不可恢复的（从代码内部）。这是因为引擎无法理解这些代码。

所以，`try..catch` 仅可以处理发生在有效代码里的错误。这样的错误被称为“运行时错误”，或者有时候称之为“异常”。
````


````warn header="`try..catch` 同步工作"
如果异常发生在“被调度”的代码中，像在 `setTimeout` 中，那么 `try..catch` 将不会捕获它：

```js run
try {
  setTimeout(function() {
    noSuchVariable; // script will die here
  }, 1000);
} catch (e) {
  alert( "won't work" );
}
```

那是因为实际上 `try..catch` 包裹的是 `setTimeout`，由它来调度那个函数。但函数本身是后来执行的，那个时候引擎已经跳出 `try..catch` 结构了。

要捕获被调度的函数里的异常，`try..catch` 必须要在那个函数里面：
```js run
setTimeout(function() {
  try {    
    noSuchVariable; // try..catch handles the error!
  } catch (e) {
    alert( "error is caught here!" );
  }
}, 1000);
```
````

## 错误对象

当错误发生时，JavaScript 产生一个包含关于它的详细信息的对象。该对象被当作参数传递给 `catch`：

```js
try {
  // ...
} catch(err) { // <-- the "error object", could use another word instead of err
  // ...
}
```

对于所有的内建错误类型，`catch` 块中的错误对象有两个主要的属性：

`name`
: 错误名称。对于一个未定义的变量那就是 `"ReferenceError"`。

`message`
: 关于错误详情的文本消息。

还有一些在大多数环境都可用的非标准属性。其中一个最广泛使用和受支持的是：

`stack`
: 当前的调用栈：一个字符串，包含关于导致错误的嵌套调用序列的信息。用于调试目的。

例如：

```js run untrusted
try {
*!*
  lalala; // error, variable is not defined!
*/!*
} catch(err) {
  alert(err.name); // ReferenceError
  alert(err.message); // lalala is not defined
  alert(err.stack); // ReferenceError: lalala is not defined at ...

  // Can also show an error as a whole
  // The error is converted to string as "name: message"
  alert(err); // ReferenceError: lalala is not defined
}
```


## 使用“try..catch”

让我们探索一个 `try..catch` 的真实用例。

我们已经知道，JavaScript 支持通过方法 [JSON.parse(str)](mdn:js/JSON/parse) 来读取 JSON 编码的值。

通常它被用来解码通过网络从服务器或者其他源接收到的数据。

我们接收它并调用 `JSON.parse`，像这样：

```js run
let json = '{"name":"John", "age": 30}'; // data from the server

*!*
let user = JSON.parse(json); // convert the text representation to JS object
*/!*

// now user is an object with properties from the string
alert( user.name ); // John
alert( user.age );  // 30
```

更多关于 JSON 的详细信息，你可以在这一章 <info:json> 找到。

**如果 `json` 异常，`JSON.parse` 产生一个错误，脚本“挂掉”。**

我们该满足于此吗？当然不！

如果这样的话，要是数据出错了，访问者将无从知道（除非他打开开发者控制台）。并且人们真的真的不喜欢“直接挂掉”，一点错误信息都没有。

让我们使用 `try..catch` 来处理错误：

```js run
let json = "{ bad json }";

try {

*!*
  let user = JSON.parse(json); // <-- when an error occurs...
*/!*
  alert( user.name ); // doesn't work

} catch (e) {
*!*
  // ...the execution jumps here
  alert( "Our apologies, the data has errors, we'll try to request it one more time." );
  alert( e.name );
  alert( e.message );
*/!*
}
```

这里我们仅仅使用 `catch` 块来显示消息，但我们可以做更多：一个新的网络请求，给访问者提供其他可选方案，发送关于错误的信息到日志设施……所有这些都好过直接挂掉。

## 抛出我们自己的错误

如果 `json` 语法上正确……但缺少一个必需的 `"name"` 属性呢？

像这样：

```js run
let json = '{ "age": 30 }'; // incomplete data

try {

  let user = JSON.parse(json); // <-- no errors
*!*
  alert( user.name ); // no name!
*/!*

} catch (e) {
  alert( "doesn't execute" );
}
```

这里 `JSON.parse` 正常执行，但对我们来说 `"name"` 的缺失确实是一个错误。

为了统一错误处理，我们将使用 `throw` 操作符。

### “Throw”操作符

`throw` 操作符产生一个错误。

语法如下：

```js
throw <error object>
```

从技术上讲，我们可以使用任何东西作为错误对象。可以是原始类型，像一个数字或一个字符串，但最好使用对象，带有 `name` 和 `message` 属性（与内建的错误类型保持一定的兼容）。

对于标准的错误类型，JavaScript 有许多内建的构造器：`Error`，`SyntaxError`，`ReferenceError`，`TypeError` 以及其他的。我们也可以使用它们来创建错误对象。

它们的语法是：

```js
let error = new Error(message);
// or
let error = new SyntaxError(message);
let error = new ReferenceError(message);
// ...
```

对于内建的错误对象（不是指任何对象，仅是错误对象），`name` 属性就是构造器的名称。而 `message` 则从参数中取得。

例如：

```js run
let error = new Error("Things happen o_O");

alert(error.name); // Error
alert(error.message); // Things happen o_O
```

让我们看看 `JSON.parse` 产生了什么类型的错误：

```js run
try {
  JSON.parse("{ bad json o_O }");
} catch(e) {
*!*
  alert(e.name); // SyntaxError
*/!*
  alert(e.message); // Unexpected token o in JSON at position 0
}
```

如我们所见，那是一个 `SyntaxError`。

……在我们的例子里，`name` 的缺失也可以被当作语法错误，假设用户必须有一个 `"name"`。

所以让我们抛出它：

```js run
let json = '{ "age": 30 }'; // incomplete data

try {

  let user = JSON.parse(json); // <-- no errors

  if (!user.name) {
*!*
    throw new SyntaxError("Incomplete data: no name"); // (*)
*/!*
  }

  alert( user.name );

} catch(e) {
  alert( "JSON Error: " + e.message ); // JSON Error: Incomplete data: no name
}
```

在行 `(*)` `throw` 操作符用给定的 `message` 产生了一个 `SyntaxError`，跟 JavaScript 生成它的方式一致。`try` 的执行立即终止并且控制流跳转至 `catch` 里。

现在 `catch` 变成一个处理所有错误的地方：用于 `JSON.parse` 和其他情形。

## 重新抛出

在上面的例子中，我们使用 `try..catch` 来处理错误的数据。但有没有可能在 `try {...}` 块发生**另一个意外的错误**？像变量未定义或者其他的，不只是“数据错误”。

像这样：

```js run
let json = '{ "age": 30 }'; // incomplete data

try {
  user = JSON.parse(json); // <-- forgot to put "let" before user

  // ...
} catch(err) {
  alert("JSON Error: " + err); // JSON Error: ReferenceError: user is not defined
  // (not JSON Error actually)
}
```

当然，一切皆有可能！程序员确实会犯错误。即使是那些成千上万人使用了几十年的开源工具————突然间，一个疯狂的漏洞被发现，导致可怕的黑客入侵（就像发生在 `ssh` 工具上的那样）。

在我们的例子中，`try..catch` 意图要捕获“错误数据”的错误。但由于其性质，`catch` 会从 `try` 得到**所有**错误。在这里，它得到一个意外的错误，但仍然显示相同的 `"JSON Error"` 消息。这样是不对的并且会让代码更加难以调试。

幸运的是，我们可以确认我们得到的是哪一个错误，例如从它的 `name`：

```js run
try {
  user = { /*...*/ };
} catch(e) {
*!*
  alert(e.name); // "ReferenceError" for accessing an undefined variable
*/!*
}
```

规则很简单:

**catch 应该仅处理它知道的错误，“重新抛出”所有其他的。**

“重新抛出”的技术可以被详细解释如下：

1. catch 捕获所有错误。
2. 在 `catch(err) {...}` 块里我们分析错误对象 `err`。
3. 如果我们不知道如何处理它，那么我们就 `throw err`。

在下面的代码中，我们使用重新抛出，因此 `catch` 仅处理 `SyntaxError`：

```js run
let json = '{ "age": 30 }'; // incomplete data
try {

  let user = JSON.parse(json);

  if (!user.name) {
    throw new SyntaxError("Incomplete data: no name");
  }

*!*
  blabla(); // unexpected error
*/!*

  alert( user.name );

} catch(e) {

*!*
  if (e.name == "SyntaxError") {
    alert( "JSON Error: " + e.message );
  } else {
    throw e; // rethrow (*)
  }
*/!*

}
```

`catch` 块里 `(*)` 行抛出的错误“跳出” `try..catch`，并且，要么被外层的 `try..catch` 结构捕获（如果有的话），要么它让脚步挂掉。

所以，`catch` 块实际上只处理它知道的该如何处理的错误并“跳过”所有其他的。

以下的例子演示了这样的错误如何被更外一层的 `try..catch` 捕获：

```js run
function readData() {
  let json = '{ "age": 30 }';

  try {
    // ...
*!*
    blabla(); // error!
*/!*
  } catch (e) {
    // ...
    if (e.name != 'SyntaxError') {
*!*
      throw e; // rethrow (don't know how to deal with it)
*/!*
    }
  }
}

try {
  readData();
} catch (e) {
*!*
  alert( "External catch got: " + e ); // caught it!
*/!*
}
```

这里 `readData` 只知道如何处理 `SyntaxError`，而外层的 `try..catch` 知道如何处理所有错误。

## try..catch..finally

等一下，这还不是全部。

`try..catch` 结构还可以有一个额外的子句：`finally`.

如果它存在，它在所有情形下都会运行：

- 在 `try` 之后，如果没有错误发生，
- 在 `catch` 之后，如果有错误发生。

扩展的语法看起来像这样：

```js
*!*try*/!* {
   ... try to execute the code ...
} *!*catch*/!*(e) {
   ... handle errors ...
} *!*finally*/!* {
   ... execute always ...
}
```

试着运行这段代码：

```js run
try {
  alert( 'try' );
  if (confirm('Make an error?')) BAD_CODE();
} catch (e) {
  alert( 'catch' );
} finally {
  alert( 'finally' );
}
```

这段代码有两种执行方式：

1. 如果对 "Make an error?" 点了“确定”，那么 `try -> catch -> finally`。
2. 如果你点了“取消”，那么 `try -> finally`。

如果我们在 `try..catch` 前做了某些工作并且希望无论在哪种情形下都要完成它，那么`finally` 子句经常派上用场。

例如，我们想要测量一个斐波那契数列函数 `fib(n)` 的耗时。自然地，我们可以在它运行前开始计时，在它之后结束。但如果在函数调用的过程中出错了呢？特别是，下面代码对 `fib(n)` 的实现在遇到负数和非整数的时候会返回错误。

无论如何，`finally` 子句是完成测量的最好地方。

这里 `finally` 保证了时间在两种情形下都将被正确测量————`fib` 成功执行以及执行出错：

```js run
let num = +prompt("Enter a positive integer number?", 35)

let diff, result;

function fib(n) {
  if (n < 0 || Math.trunc(n) != n) {
    throw new Error("Must not be negative, and also an integer.");
  }
  return n <= 1 ? n : fib(n - 1) + fib(n - 2);
}

let start = Date.now();

try {
  result = fib(num);
} catch (e) {
  result = 0;
*!*
} finally {
  diff = Date.now() - start;
}
*/!*

alert(result || "error occured");

alert( `execution took ${diff}ms` );
```

你可以运行代码，在 `prompt` 时输入 `35` 来进行检查————它正常执行，`try` 之后 `finally`。然后输入 `-1`————将立即产生错误，执行将耗时 `0ms`。两种测量都正确完成。

换句话说，有两种退出函数的方式：要么 `return` 要么 `throw`。`finally` 会处理两种情形。


```smart header="`try..catch..finally` 里的变量是局部的"
请注意在上面的代码中 `result` 和 `diff` 变量在 `try..catch` **之前**声明。

否则，如果 `let` 放在 `{...}` 块里，它仅在里面可见。
```

````smart header="`finally` 和 `return`"
finally 子句对**任何**从 `try..catch` 中的退出都有效。包括显式地 `return`。

在下面的例子中，`try` 块里有一个 `return`。在这种情形下，`finally` 会在控制跳转至外部代码前被执行。

```js run
function func() {

  try {
*!*
    return 1;
*/!*

  } catch (e) {
    /* ... */
  } finally {
*!*
    alert( 'finally' );
*/!*
  }
}

alert( func() ); // first works alert from finally, and then this one
```
````

````smart header="`try..finally`"

`try..finally` 结构，没有 `catch` 子句，也很有用。当我们不想在这里处理错误，但想确保已经开始的过程被完成，我们可以使用它。

```js
function func() {
  // start doing something that needs completion (like measurements)
  try {
    // ...
  } finally {
    // complete that thing even if all dies
  }
}
```
在上面的代码中，`try` 块里的代码总会跳出，因为没有 `catch`。但 `finally` 会在执行流跳转至外部前起作用。
````

## 全局捕获

```warn header="特定环境的"
这一节的信息不是 JavaScript 核心的一部分。
```

让我们想象一下，我们在 `try..catch` 之外遇到了一个致命错误。像编程错误或者其他糟糕的东西。

在这种情况下是否有办法作出响应呢？也许我们想要记录错误，显示某些东西给用户（正常情况下他看不到错误信息）等等。

规范里并没有，但环境会提供它，因为真的有用。例如，Node.JS 有 [process.on('uncaughtException')](https://nodejs.org/api/process.html#process_event_uncaughtexception)。在浏览器中我们可以赋值一个函数给特殊的 [window.onerror](mdn:api/GlobalEventHandlers/onerror) 属性。它会在出现未捕获的错误时运行。

语法：

```js
window.onerror = function(message, url, line, col, error) {
  // ...
};
```

`message`
: 错误信息。

`url`
: 发生错误的脚本 URL。

`line`, `col`
: 错误发生的行数列数。

`error`
: 错误对象。

例如：

```html run untrusted refresh height=1
<script>
*!*
  window.onerror = function(message, url, line, col, error) {
    alert(`${message}\n At ${line}:${col} of ${url}`);
  };
*/!*

  function readData() {
    badFunc(); // Whoops, something went wrong!
  }

  readData();
</script>
```

全局处理器 `window.onerror` 的角色并不是用来恢复脚本执行的————如果是编程错误的话那几乎是不可能的，而是发送错误消息给开发者。

有一些为这样的场景提供错误日志的 Web 服务，像 <https://errorception.com> 或者 <http://www.muscula.com>。

它们是这样工作的：

1. 我们在服务上注册，获取一段 JS （或是脚本的 URL），把它们插入页面。
2. 这些 JS 脚本有一个自定义的 `window.onerror` 函数。
3. 当发生错误时，脚本会发送一个关于错误的网络请求到它的服务。
4. 我们可以登入服务的 Web 界面查看错误。

## 总结

`try..catch` 结构运行处理运行时错误。意即它允许尝试运行代码并捕获可能发生的错误。

语法是：

```js
try {
  // run this code
} catch(err) {
  // if an error happened, then jump here
  // err is the error object
} finally {
  // do in any case after try/catch
}
```

可以没有 `catch` 部分或者没有 `finally`，所以 `try..catch` 和 `try..finally` 也是有效的。

错误对象有如下属性：

- `message`————人类可读的错误消息。
- `name`————包含错误名称的字符串（错误构造器的名称）。
- `stack`（非标准）————错误产生时的调用栈。

我们也可以使用 `throw` 操作符来产生我们自己的错误。从技术上讲，`throw` 的参数可以是任意值，但它通常都是一个继承自内建的 `Error` 类的错误对象。更多内容在下一章的扩展错误。

重新抛出是错误处理的基本模式：`catch` 块通常期望和知道该如何处理特定的错误类型，所以它应该抛出它不知道的错误。

即使我们没有 `try..catch`，大多数环境都允许我们设置一个“全局”的错误处理器来捕获“跳出”的错误。在浏览器里那便是 `window.onerror`。
