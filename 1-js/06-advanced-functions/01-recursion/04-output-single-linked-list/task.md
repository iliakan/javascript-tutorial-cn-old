重要度: 5

---

# 输出单项链接的列表

假设我们有一个单向链接列表，如同<info:recursion>章节中所描述的：

```js
let list = {
  value: 1,
  next: {
    value: 2,
    next: {
      value: 3,
      next: {
        value: 4,
        next: null
      }
    }
  }
};
```

编写一个函数 `printList(list)` ，它能一个一个地输出每个列表中的元素。

请分别使用循环与递归实现。

使用递归与不使用递归，哪个更好？
