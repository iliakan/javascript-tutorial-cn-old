重要程度: 5

---

# 继承 SyntaxError

创建一个 `FormatError` 类, 继承自内建的 `SyntaxError` 类.

它应该支持 `message`, `name` 和 `stack` 属性.

用法示例:

```js
let err = new FormatError("formatting error");

alert( err.message ); // formatting error
alert( err.name ); // FormatError
alert( err.stack ); // stack

alert( err instanceof FormatError ); // true
alert( err instanceof SyntaxError ); // true (because inherits from SyntaxError)
```
