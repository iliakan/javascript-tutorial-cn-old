当我们把代码放到函数中时，差异变得很明显。

如果 `try..catch` 中有一个“跳出”。两者的行为是不一致的。

例如，当 `try..catch` 中有一个 `return` 语句的时候。`finally` 子句在 `try..catch` 中的**任何**退出都有效，甚至是 `return` 语句：在 `try..catch` 完成之后，在控制返回到调用的代码之前。

```js run
function f() {
  try {
    alert('start');
*!*
    return "result";
*/!*
  } catch (e) {
    /// ...
  } finally {
    alert('cleanup!');
  }
}

f(); // cleanup!
```

...或者有一个 `throw` 语句，像这里：

```js run
function f() {
  try {
    alert('start');
    throw new Error("an error");
  } catch (e) {
    // ...
    if("can't handle the error") {
*!*
      throw e;
*/!*
    }

  } finally {
    alert('cleanup!')
  }
}

f(); // cleanup!
```

这里，`finally` 保证了清理工作的完成。如果我们只是把清理代码放到 `f` 之后，它不会被执行。
