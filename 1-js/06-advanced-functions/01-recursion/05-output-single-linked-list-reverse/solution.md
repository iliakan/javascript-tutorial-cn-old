# 使用递归

递归逻辑在这里有点绕。

我们必须先输出列表的剩余部分，*然后* 再输出当前的这个元素：

```js run
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

function printReverseList(list) {

  if (list.next) {
    printReverseList(list.next);
  }

  alert(list.value);
}

printReverseList(list);
```

# 使用循环

使用循环的解法与上个章节相比也会复杂些。

我们既不能直接获取列表`list`中的最后一个值，也不能“走回头路”。

所以，我们要做的首先是按照正常次序遍历整个列表，将每个元素记忆在一个数组中，然后再将数组中的元素逆序输出：

```js run
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

function printReverseList(list) {
  let arr = [];
  let tmp = list;

  while (tmp) {
    arr.push(tmp.value);
    tmp = tmp.next;
  }

  for (let i = arr.length - 1; i >= 0; i--) {
    alert( arr[i] );
  }
}

printReverseList(list);
```

请注意，递归解法其实做法完全一样，它先沿着列表遍历，将列表中的每个元素记忆在嵌套调用中（即：执行上下文堆栈中），然后再输出他们。
