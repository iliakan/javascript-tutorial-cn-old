# 使用循环的方案

使用循环的方案：

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

function printList(list) {
  let tmp = list;

  while (tmp) {
    alert(tmp.value);
    tmp = tmp.next;
  }

}

printList(list);
```

请注意这里我们使用了一个临时变量 `tmp`来走遍列表. 技术上，我们可以使用一个函数参数 `list` ：

```js
function printList(list) {

  while(*!*list*/!*) {
    alert(list.value);
    list = list.next;
  }

}
```

...但这种方式可能不明智，未来我们可能需要扩展一个函数，对列表做些别的事情。如果我们改成 `list`，我们可能会丧失一些灵活性。

说到变量的命名，这里的 `list`是列表本身，也是列表的第一个元素。我们应该保持这种风格，因为可读性高且易于维护。

另外一面， `tmp` 的角色就是专门用于列表的遍历变量，如同`for` 循环中的 `i`一样。

# 递归解法

问题 `printList(list)` 的递归解法使用一个简单的逻辑：当需要输出一个列表时，输出当前元素 `list` ，然后再输出当前元素的下一个元素 `list.next`：

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

function printList(list) {

  alert(list.value); // 输出当前元素

  if (list.next) {
    printList(list.next); // 如法炮制，输出后面的。
  }

}

printList(list);
```

那现在看看哪个更好？

技术上来说，循环解法的效率更高。两种解法殊途同归，但循环解法无需为嵌套调用耗费额外的资源。

事情的另外一面是：递归解法代码更短并且很多时候这也意味着更容易理解。
