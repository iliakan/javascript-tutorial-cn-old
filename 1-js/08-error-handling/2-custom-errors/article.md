# 自定义错误，扩展 Error

当我们开发一些东西的时候，我们经常需要自己的错误类型来反映我们任务中可能出错的特定的操作。对于网络操作的错误我们可能需要 `HttpError`，对于数据库操作可能是 `DbError`，对于搜索操作可能是 `NotFoundError` 等等。

我们的错误类型应该支持基本的错误属性像 `message`，`name`，另外，最好有 `stack`。但是它们也可以有自己的属性，例如 `HttpError` 对象可能有 `statusCode` 属性，其取值可能像是 `404` 或者 `403` 或者 `500`。

JavaScript 允许使用 `throw` 抛出任何参数，所以从技术上讲我们的自定义错误类型并不需要继承自 `Error`。但如果我们继承它，我们便可以使用 `obj instanceof Error` 来识别错误对象。所以最好继承它。

随着我们应用的开发，我们的错误类型自然地形成了一个层次结构，例如 `HttpTimeoutError` 可能继承自 `HttpError` 等等。

## 扩展 Error

作为一个例子，让我们考虑一下一个函数 `readUser(json)`，它应该读取 JSON 格式的用户数据。

如下是一个有效的 `json` 看起来的样子：
```js
let json = `{ "name": "John", "age": 30 }`;
```

在函数内部，我们将使用 `JSON.parse`。如果它接收到一个异常的 `json`，那么它会抛出 `SyntaxError`。

但即使 `json` 在语法上是正确的，那也不意味着它就是一个有效的用户，对吧？它可能缺失必要的数据。例如，它可能缺少对我们的用户来说重要的 `name` 和 `age` 属性。

我们的 `readUser` 函数将不仅读取 JSON，也会检查(“验证”)数据。如果没有必需字段，或者格式出错，那就是一个错误。并且，那不是一个 `SyntaxError`，因为数据是语法正确的, 而是另一种错误。我们将称它为 `ValidationError` 并为之创建一个类。一个这样的错误也应该携带关于违规字段的信息。

我们的 `ValidationError` 类应该继承自内建的 `Error` 类。

`Error` 类是内建的，但我们心中应该有它大致的代码以便理解我们要扩展什么。

如下：

```js
// The "pseudocode" for the built-in Error class defined by JavaScript itself
class Error {
  constructor(message) {
    this.message = message;
    this.name = "Error"; // (different names for different built-in error classes)
    this.stack = <nested calls>; // non-standard, but most environments support it
  }
}
```

现在让我们继续，`ValidationError` 继承自它：

```js run untrusted
*!*
class ValidationError extends Error {
*/!*
  constructor(message) {
    super(message); // (1)
    this.name = "ValidationError"; // (2)
  }
}

function test() {
  throw new ValidationError("Whoops!");
}

try {
  test();
} catch(err) {
  alert(err.message); // Whoops!
  alert(err.name); // ValidationError
  alert(err.stack); // a list of nested calls with line numbers for each
}
```

请看一下它的构造器：

1. 在行 `(1)` 我们调用了父类的构造器。JavaScript 要求我们在子类构造器中调用 `super`，所以那是强制的。父类构造器设置了 `message` 属性。
2. 父类构造器也设置了 `name` 属性，所以在行 `(2)` 我们将它重置为正确的值。

让我们尝试在 `readUser(json)` 中使用它：

```js run
class ValidationError extends Error {
  constructor(message) {
    super(message);
    this.name = "ValidationError";
  }
}

// Usage
function readUser(json) {
  let user = JSON.parse(json);

  if (!user.age) {
    throw new ValidationError("No field: age");
  }
  if (!user.name) {
    throw new ValidationError("No field: name");
  }

  return user;
}

// Working example with try..catch

try {
  let user = readUser('{ "age": 25 }');
} catch (err) {
  if (err instanceof ValidationError) {
*!*
    alert("Invalid data: " + err.message); // Invalid data: No field: name
*/!*
  } else if (err instanceof SyntaxError) { // (*)
    alert("JSON Syntax Error: " + err.message);
  } else {
    throw err; // unknown error, rethrow it (**)
  }
}
```

在上面的代码中，`try..catch` 块处理了我们的 `ValidationError`，也处理了来自 `JSON.parse` 的内建的 `SyntaxError`。

请看看在行 `(*)` 我们如何使用 `instanceof` 来检查特定的错误类型。

我们也可以通过查看 `err.name`，像这样：

```js
// ...
// instead of (err instanceof SyntaxError)
} else if (err.name == "SyntaxError") { // (*)
// ...
```  

使用 `instanceof` 的版本好得多，因为在未来，我们将扩展 `ValidationError`，创建它的子类型，像 `PropertyRequiredError`。`instanceof` 检查仍将可以用于新的继承出来的类。所以它是面向未来的。

另外，同样重要的是，如果 `catch` 遇到未知的错误，那么它会在行 `(**)` 重新抛出它。`catch` 只知道如何处理验证错误和语法错误，其他类型的错误（由于代码中的拼写错误或者其他原因）应该跳出。

## 更进一步的继承

`ValidationError` 类是非常泛化的。许多方面都可能出错。属性可能缺失或者格式错误（像 `age` 属性有一个字符串类型的值）。让我们创建一个更加具体的类型 `PropertyRequiredError`，明确地用于属性缺失。它将携带额外的关于缺失属性的信息。

```js run
class ValidationError extends Error {
  constructor(message) {
    super(message);
    this.name = "ValidationError";
  }
}

*!*
class PropertyRequiredError extends ValidationError {
  constructor(property) {
    super("No property: " + property);
    this.name = "PropertyRequiredError";
    this.property = property;
  }
}
*/!*

// Usage
function readUser(json) {
  let user = JSON.parse(json);

  if (!user.age) {
    throw new PropertyRequiredError("age");
  }
  if (!user.name) {
    throw new PropertyRequiredError("name");
  }

  return user;
}

// Working example with try..catch

try {
  let user = readUser('{ "age": 25 }');
} catch (err) {
  if (err instanceof ValidationError) {
*!*
    alert("Invalid data: " + err.message); // Invalid data: No property: name
    alert(err.name); // PropertyRequiredError
    alert(err.property); // name
*/!*
  } else if (err instanceof SyntaxError) {
    alert("JSON Syntax Error: " + err.message);
  } else {
    throw err; // unknown error, rethrow it
  }
}
```

新的 `PropertyRequiredError` 类型使用起来很简单：我们只需要传递属性名称：`new PropertyRequiredError(property)`。人类可读的 `message` 由构造器生成。

请注意，在 `PropertyRequiredError` 的构造器中，`this.name` 被再一次手动赋值。可能有点冗长————每次创建自定义错误类型的时候都要赋值 `this.name = <class name>`。但有一个解决方案。我们可以创建自己的“基本错误”类型，在它的构造器使用 `this.constructor.name` 替代 `this.name` 来消除这个负担。然后继承它。

让我们称之为 `MyError`。

如下是包含 `MyError` 和其他自定义错误类型代码的简化版本：

```js run
class MyError extends Error {
  constructor(message) {
    super(message);
*!*
    this.name = this.constructor.name;
*/!*
  }
}

class ValidationError extends MyError { }

class PropertyRequiredError extends ValidationError {
  constructor(property) {
    super("No property: " + property);
    this.property = property;
  }
}

// name is correct
alert( new PropertyRequiredError("field").name ); // PropertyRequiredError
```

现在自定义错误类型更短了，特别是 `ValidationError`，因为在构造器中我们摆脱了 `"this.name = ..."` 这一行。

## 封装异常

在上面的代码中函数 `readUser` 的目的是“读取用户数据”，对吧？在这个过程中可能出现不同类型的错误。现在我们有 `SyntaxError` 和 `ValidationError`，但在未来函数 `readUser` 可能扩展：新的代码可能产生其他类型的错误。

调用 `readUser` 的代码应该处理这些错误。现在它在 `catch` 块中使用多个 `if` 来检查不同的错误类型并重新抛出未知的错误。但如果 `readUser` 函数产生多种类型的错误 ———— 那么我们应该问问自己：我们真的希望在调用 `readUser` 的代码中一个一个地检查所有错误类型吗？

通常答案是“不”：外部代码希望在“所有错误的一层之上”。它希望有某种“数据读取错误”。错误究竟如何发生————通常是不相干的（出错信息描述了它）。或者，更好的是我们有办法获取错误详情，但仅当我们需要这样做的时候。

那让我们来创建一个新的类 `ReadError` 来表示这样的错误。如果在 `readUser` 里发生错误，我们将捕获它并生成 `ReadError`。我们也将在 `cause` 属性中保存对原始错误的引用。外部代码仅需要检查 `ReadError`。

如下代码定义了 `ReadError` 并演示了它在 `readUser` 和 `try..catch` 中的使用：

```js run
class ReadError extends Error {
  constructor(message, cause) {
    super(message);
    this.cause = cause;
    this.name = 'ReadError';
  }
}

class ValidationError extends Error { /*...*/ }
class PropertyRequiredError extends ValidationError { /* ... */ }

function validateUser(user) {
  if (!user.age) {
    throw new PropertyRequiredError("age");
  }

  if (!user.name) {
    throw new PropertyRequiredError("name");
  }
}

function readUser(json) {
  let user;

  try {
    user = JSON.parse(json);
  } catch (err) {
*!*
    if (err instanceof SyntaxError) {
      throw new ReadError("Syntax Error", err);
    } else {
      throw err;
    }
*/!*
  }

  try {
    validateUser(user);
  } catch (err) {
*!*
    if (err instanceof ValidationError) {
      throw new ReadError("Validation Error", err);
    } else {
      throw err;
    }
*/!*
  }

}

try {
  readUser('{bad json}');
} catch (e) {
  if (e instanceof ReadError) {
*!*
    alert(e);
    // Original error: SyntaxError: Unexpected token b in JSON at position 1
    alert("Original error: " + e.cause);
*/!*
  } else {
    throw e;
  }
}
```

在上面的代码中，`readUser` 像描述的那样工作————捕获语法和验证错误并抛出 `ReadError` 错误（未知错误依旧重新抛出）。

所以外部代码检查 `instanceof ReadError`，仅此而已。不需要罗列所有可能的错误类型。

这个方法叫做“封装异常”，因为我们取得“低级异常”并把它们“封装”进 `ReadError`，对于调用端的代码来说这样更加抽象并易于使用。它在面向对象编程中被广泛使用。

## 总结

- 通常，我们可以继承 `Error` 和其他内建的错误类型，只需要修改 `name` 属性并且不要忘记调用 `super`。
- 大多数时候，我们应该使用 `instanceof` 来检查特定的错误。它对继承同样有效。但有时我们有一个来自第三方库的错误对象并且没有简单的途径获取它的类。这个时候 `name` 属性可以被用来做这样的检查。
- 当一个函数处理低级错误并创建一个更高级的对象来报告错误的时候，封装异常是一个广泛使用的技术。低级异常有时成为这个对象的属性，像上面例子中的 `err.cause`，但这不是严格必需的。
