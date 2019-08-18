# 递归与堆栈

让我们回到函数做进一步的学习。

我们的第一个主题是 *递归*。

如果你不是一个编程新手，或许对此概念已经有所了解，那你可以跳过这个章节。

递归是一种编程模式，通常在以下情况下很有用：一个任务可以被自然地分解成多个类似但简单些的任务；或者是一个任务可以被简化成一个容易的动作并加上一个可以简化的、类似的任务变通；或者是为了应对一些特殊的数据结构。

在一个完整一个特定的函数当中，该函数可以调用多个其他的函数。能够在函数中调用该函数自身的，是一种特殊情形，这种特殊情形就叫着： *递归*。

[cut]

## 两种思考的方法

让我们从一个简单的事情开始 -- 编写一个函数 `pow(x, n)` ，能够 `x` 的自然数 `n` 的次方。 换言之， `x` 与自身相乘 `n` 次。

```js
pow(2, 2) = 4
pow(2, 3) = 8
pow(2, 4) = 16
```

这里有两种思路来实现它。

1. 迭代式思维: 使用 `for` 循环：

    ```js run
    function pow(x, n) {
      let result = 1;

      // multiply result by x n times in the loop
      for(let i = 0; i < n; i++) {
        result *= x;
      }

      return result;
    }

    alert( pow(2, 3) ); // 8
    ```

2. 递归式思维： 简化任务，调用自身：

    ```js run
    function pow(x, n) {
      if (n == 1) {
        return x;
      } else {
        return x * pow(x, n - 1);
      }
    }

    alert( pow(2, 3) ); // 8
    ```

请注意一下递归的形式有个根本的不同。

当 `pow(x, n)` 被调用, 执行过程分成两个分支：

```js
              if n==1  = x
             /
pow(x, n) =
             \       
              else     = x * pow(x, n - 1)
```

1. 当 `n==1`时， 所有东西都不重要。此时被称作递归的 *初步*， 因为函数立即产生一个明显的结果，那就是： `pow(x, 1)` 等于 `x`。
2. 否则， 我们可以将 `pow(x, n)` 用 `x * pow(x, n-1)`来表示。 在数学中可以写成这样的公式：  <code>x<sup>n</sup> = x * x<sup>n-1</sup></code>。 这被称为 *一个递归步骤*： 我们将任务转变为一个简单的动作（乘以 `x`）以及一个对相同任务的、更简单的调用（求小一点的 `n` 的次方 `pow` ）。 下一步继续简化直至 `n` 等于 `1`。

我们也可以称为： `pow` *递归地调用自身* 直到 `n == 1`.

![recursive diagram of pow](recursion-pow.png)


例如，计算 `pow(2, 4)` 时，递归方法会按照如下步骤进行计算：

1. `pow(2, 4) = 2 * pow(2, 3)`
2. `pow(2, 3) = 2 * pow(2, 2)`
3. `pow(2, 2) = 2 * pow(2, 1)`
4. `pow(2, 1) = 2`

所以，递归将一个函数调用复杂度降低，持续进行进一步简单的调用，直到求得最终的结果。

````smart header="递归通常更短"
一个递归式方案通常都比迭代式方案更短。

这里我们利用三元操作符 `?` 而不是 `if` 来改写 `pow(x, n)` ，使其更加简洁并保持很好的可读性：

```js run
function pow(x, n) {
  return (n == 1) ? x : (x * pow(x, n-1));
}
```
````

潜逃调用的最多次数（包括第一次调用）被称为 *递归深度*. 此例中，递归深度等于 `n`。

最大递归深度受限于 JavaScript 引擎。10000是一个可以基本保证的深度，有些引擎允许更深，但10000超过了大多数引擎允许的深度。有些自动优化的方法可以缓解此问题 ("tail calls optimizations")，但他们还没有得到大范围的支持，且很多时候只能支持简单的使用清醒。

虽然这限制了递归的应用场合，但它还是有着广泛的应用场合。有很多任务，使用递归方式思考的话，可以用更简单的代码来实现，从而更易于维护。

## The execution stack

我们通过进一步观察函数，再来审视一下递归调用的工作过程。 

函数运行有关的信息是储存在 *执行上下文* 中。

[执行上下文](https://tc39.github.io/ecma262/#sec-execution-contexts) 用来保存一个函数在执行过程中细节的数据结构： 当前的控制流在哪里， 当前的变量，`this` 当前的值（我们在这还没有用到）以及其他的一些内部细节。

一个函数调用一定有一个执行上下文对应着。

当一个函数进行嵌套调用时，以下会发生：

- 当前函数会暂停。
- 与之对应的执行上下文会保存在一个叫 *执行上下文堆栈* 的特殊数据结构中。
- 嵌套的调用开始执行
- 当嵌套调用结束后，老的执行上下文会被从堆栈中取出来，这样原来从外面调用的函数就可以从暂停的地方开始继续执行。

让我们看看 `pow(2, 3)` 调用过程中发生了什么。

### pow(2, 3)

在 `pow(2, 3)` 调用刚发出时，执行上下文会存储变量： `x = 2, n = 3`，执行流现在是在函数的第 `1` 行。

这个过程草图大致是：

<ul class="function-execution-context-list">
  <li>
    <span class="function-execution-context">上下文： { 第一行，x: 2, n: 3 }</span>
    <span class="function-execution-context-call">pow(2, 3)</span>
  </li>
</ul>

函数开始执行后，条件 `n == 1` 的结果是“否”，因此执行流前行到 `if` 的第二个分支：

```js run
function pow(x, n) {
  if (n == 1) {
    return x;
  } else {
*!*
    return x * pow(x, n - 1);
*/!*
  }
}

alert( pow(2, 3) );
```


变量还是相同的变量，但行变了，所以上下文变成了：

<ul class="function-execution-context-list">
  <li>
    <span class="function-execution-context">上下文: { 第5行，x: 2, n: 3 }</span>
    <span class="function-execution-context-call">pow(2, 3)</span>
  </li>
</ul>

为了计算 `x*pow(x, n-1)`，我们必须使用新的参数 `pow(2,2)` 来执行一个 `pow` 的子调用。

### pow(2, 2)

JavaScript 会在 *执行上下文堆栈* 中记住当前执行的上下文，以便进行一个嵌套调用。

这里我们调用相同的函数 `pow`，但这不是重点，所有的函数的执行过程都是相同的：

1. 当前上下文被“记忆”在堆栈的顶端。
2. 与子调用相关的上下文会被创建。
3. 当子调用结束后，前面的上下文会被从堆栈中弹出，从而可以继续执行。

以下是子调用 `pow(2, 2)` 发生时执行上下文堆栈的情况：

<ul class="function-execution-context-list">
  <li>
    <span class="function-execution-context">上下文: { 第1行，x: 2, n: 2 }</span>
    <span class="function-execution-context-call">pow(2, 2)</span>
  </li>
  <li>
    <span class="function-execution-context">上下文: { 第5行， x: 2, n: 3 }</span>
    <span class="function-execution-context-call">pow(2, 3)</span>
  </li>
</ul>

当前的执行上下文是在堆栈的顶端（粗体），前面记住的上下文在下面。

当自调用完成后，恢复前面的上下文很容易，因为它保存了所有的变量值以及停止时所在的代码行。此此处我们使用了“行”这个字但实际上图中题意表达得更精确。

### pow(2, 1)

上述过程不停地重复： 第5行会发出一个新的子调用，此时使用变量值 `x=2`, `n=1`。

一个新的执行上下文会被创建，前面的上下文会被压到堆栈的顶上。：

<ul class="function-execution-context-list">
  <li>
    <span class="function-execution-context">上下文: { 第1行， x: 2, n: 1 }</span>
    <span class="function-execution-context-call">pow(2, 1)</span>
  </li>
  <li>
    <span class="function-execution-context">上下文: { 第5行， x: 2, n: 2 }</span>
    <span class="function-execution-context-call">pow(2, 2)</span>
  </li>
  <li>
    <span class="function-execution-context">上下文: { 第5行， x: 2, n: 3}</span>
    <span class="function-execution-context-call">pow(2, 3)</span>
  </li>
</ul>

现在有两个老的上下文以及一个针对 `pow(2, 1)`的新上下文。

### The exit

执行 `pow(2, 1)` 时，与前面的不一样，条件 `n == 1` 判断为“真”，所以 `if` 的第一个分支开始工作:

```js
function pow(x, n) {
  if (n == 1) {
*!*
    return x;
*/!*
  } else {
    return x * pow(x, n - 1);
  }
}
```

后续再无嵌套调用，函数结束，返回 `2`。

当函数结束，关联的执行上下文就不再需要，所以会被从内存中移除，前面的执行上下文会被恢复到堆栈的顶上：


<ul class="function-execution-context-list">
  <li>
    <span class="function-execution-context">上下文: { 第5行， x: 2, n: 2}</span>
    <span class="function-execution-context-call">pow(2, 2)</span>
  </li>
  <li>
    <span class="function-execution-context">上下文: { 第5行， x: 2, n: 3}</span>
    <span class="function-execution-context-call">pow(2, 3)</span>
  </li>
</ul>

`pow(2, 2)`的执行得以恢复。 它有自调用 `pow(2, 1)`所带来的结果，因此 `x * pow(x, n-1)`也就可以求出结果，返回 `4`。

接下来前面的上下文又可以得到恢复：

<ul class="function-execution-context-list">
  <li>
    <span class="function-execution-context">上下文: { 第5行， x: 2, n: 3}</span>
    <span class="function-execution-context-call">pow(2, 3)</span>
  </li>
</ul>

当它也执行结束，我们就可以得出： `pow(2, 3) = 8`。

此处的递归深度是： **3**.

从上述示意可以看出，递归深度等于堆栈中上下文的最大值。

要注意一下对内存的需求。上下文会消耗内存。在我们这个例子中，求 `n` 次方实际上需要消耗 `n`个上下文的所占用的内存，以便支持所有小于 `n`的值计算。

基于循环的算法可以节省很多内存：

```js
function pow(x, n) {
  let result = 1;

  for(let i = 0; i < n; i++) {
    result *= x;
  }

  return result;
}
```

迭代式的 `pow`在执行过程中需要不停变化 `i` 以及 `result`，但只需要一个上下文。所需要的内存是较少且是固定的，与 `n`值大小无关。

**所有的递归都可以改写成循环。循环的方式通常都有更高的效率。**

...但有时候这种改写的工作量相当大，特别是函数根据条件有着不同的递归子调用或者分支条件错综复杂。此时的优化可能没有必要或者说不值得花费那么多的功夫。

递归带来更短的代码并且易于理解与维护。并不是所有的地方都需要优化，更多的时候我们是需要更好的代码，这就是为何递归得到应用的原因。

## 递归shi遍历

递归另外一个伟大的应用场合是递归式遍历

想象一下我们有一家公司，人员架构可以用一个对象来表示：

```js
let company = {
  sales: [{
    name: 'John',
    salary: 1000
  }, {
    name: 'Alice',
    salary: 600
  }],

  development: {
    sites: [{
      name: 'Peter',
      salary: 2000
    }, {
      name: 'Alex',
      salary: 1800
    }],

    internals: [{
      name: 'Jack',
      salary: 1300
    }]
  }
};
```

换言之，公司下面有部门。

- 一个部门中可能有人员数组。例如： `sales` （销售）部门有两个员工： John 与 Alice。
- 或者一个部门可能包含有两个子部门，例如 `development`（开发）部门下有两个分支：`site`与`internal`。每个分支都有各自的员工。
- 当一个子部门增长时，它可能被分成子子部门（或者 `team`，即：团队）。

    例如：`sites` 部门未来可能被分成两个团队，分别是： `siteA` 与 `siteB`。他们未来可能被进一步的细分。这些在图中没有表示出来，但你可以在脑中想象一下。

现在假如说我们需要一个函数，用来获取所有的薪水总数，那我们该如何做呢？

迭代式的方案此时就不容易实现，因为人员结构不简单。迭代式方法不容易实现，因为没有简单的结构。首先能想到的主意，或许是用 `for`循环遍历整个 `company`，然后用子循环来遍历所有的1级部门。但我们马上就需要更多的子循环遍历像 `sites`这样的2级部门。那接下来的3级子部门，我们是否需要更多的子循环呢？我们应该停在3级还是需要继续到4级？同时，我们将遍历3-4层的子循环的代码都放在一个单一对象内部，那代码本身也够丑陋的。

现在就是试试递归方式的时候了。

我们可以看到，当函数对一个部门进行求和时，会碰到两种情况：

1. 要么这个部门很简单，只有 *人员数组*。* -- 那么我们简单地用循环来累加薪水。
2. 要么这个部门是 *一个带有 `N` 个子部门的对象* -- 那么我们可以递归地调用 `N`次函数以求得各个子部门的薪水之和，然后再合并到一起。

上面所述的 (1) 就是递归的基础，一个不复杂的清醒。

而 (2) 就是递归的步骤. 一个复杂的任务被分解成针对各个下一级部门的子任务。子任务会被再次分解，但很快这些子任务都会到达 (1)的情形时截止。

通过代码来了解这个算法可能更简单：


```js run
let company = { // the same object, compressed for brevity
  sales: [{name: 'John', salary: 1000}, {name: 'Alice', salary: 600 }],
  development: {
    sites: [{name: 'Peter', salary: 2000}, {name: 'Alex', salary: 1800 }],
    internals: [{name: 'Jack', salary: 1300}]
  }
};

// The function to do the job
*!*
function sumSalaries(department) {
  if (Array.isArray(department)) { // case (1)
    return department.reduce((prev, current) => prev + current.salary, 0); // sum the array
  } else { // case (2)
    let sum = 0;
    for(let subdep of Object.values(department)) {
      sum += sumSalaries(subdep); // recursively call for subdepartments, sum the results
    }
    return sum;
  }
}
*/!*

alert(sumSalaries(company)); // 6700
```

上述代码短小且易于理解（希望如此），这就是递归的威力。它适应于任何层级子部门的嵌套情形。

以下是调用的示意图：

![recursive salaries](recursive-salaries.png)

我们很容易发现原则是：针对每个 `{...}` 发出的自调用，数组 `[...]` 是第归树上的 "叶子" 他们总是能够马上给出一个结果。

注意一下这里的代码使用了我们前面介绍过的一些聪明技术：

- `arr.reduce` 方法在章节 <info:array-methods> 中有介绍，用来获取一个数组的和。
- 循环 `for(val of Object.values(obj))` 可以用来遍历每个对象的值， `Object.values` 以数组的方式返回他们的值。


## 递归式结构

一个递归式的数据结构（递归式地定义）是指部分重复自身的结构。

我们刚刚看过了上面一个公司的结构。

一个公司的*部门*是:
- 要么一个人员数组
- 要么是包含了很多*部门*的对象.

对于一个Web开发人员来说，HTML与 XML 文档可能是更好理解的一个例子。

在一个HTML文档当中，一个*HTML标签* 可能包含了一个如下列表:
- 文本部分。
- HTML注释。
- 其他的 *HTML标签* （也即可以包含文本、HTML注释或者标签等等）。

这就又是一个递归式的定义。

为了更好地理解，我们将讨论另一个递归式的结构，名叫： "Linked list 链式列表" ，这个在很多情况下是比数组更好的一个替代。

### 链式列表

想象一下，我们需要存储一个有序的对象列表。

数组可能是一个自然的选择：

```js
let arr = [obj1, obj2, obj3];
```

...但数组有一个问题，“删除元素“与”插入元素“操作成本非常高。例如： `arr.unshift(obj)` 操作会为了腾出空间一个新的 `obj`而不得不重新编号所有的元素，当数组很大时，需要很多时间。 `arr.shift()`操作也有类似问题。

数组结构操作中唯一不需要大量重新编号的是在数组结尾处操作： `arr.push/pop`。所以，对于一个大的队列来说，数组操作起来很慢。

如果一定需要快速的插入、删除，那么可以选择另外一种叫做链式列表的数据结构做替代。[链式列表](https://en.wikipedia.org/wiki/Linked_list).

*链式列表中的元素* 是用递归方法定义的对象：
- `value` 值.
- `next` （下一个）属性指向下一个 *链式列表元素*，若是空 `null` 则表示这是尾部。

例如：

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

上述列表的可以用图形表示为：

![linked list](linked-list.png)

可以用下面的代码构建上述数据：

```js no-beautify
let list = { value: 1 };
list.next = { value: 2 };
list.next.next = { value: 3 };
list.next.next.next = { value: 4 };
```

这里我们可以更加清晰地看到这里有多个对象，每个对象都有 `value` 以及指向邻居的 `next`。  `list` 变量指向这个链条中的第一个对象，因此，顺着 `next` 指针我们找到任何对象。

列表可以很容地被分成多个部分，也可以很容易合并回来：

```js
let secondList = list.next.next;
list.next.next = null;
```

![linked list split](linked-list-split.png)

合并：

```js
list.next.next = secondList;
```

当然也可以在任何位置插入或者删除一个元素。

例如，我们要前置一个值，我们只需要更新列表的头部：

```js
let list = { value: 1 };
list.next = { value: 2 };
list.next.next = { value: 3 };
list.next.next.next = { value: 4 };

*!*
// prepend the new value to the list
list = { value: "new item", next: list };
*/!*
```

![linked list](linked-list-0.png)

当需要删除中间的一个值时，可以更改 `next` 指向下一个：

```js
list.next = list.next.next;
```

![linked list](linked-list-remove-1.png)

我们让 `list.next` 跳过了 `1` 直接指向 `2`。  `1`这个值就从链条中被排除出去。如果没有别的地方引用或者存储的话，那么它将会被从内存中自动回收。

不像数组，这里没有大批量的重新编号，我们可以很容易地重新安排元素。

自然地，列表并不是总是比数组表现好。否则，每个人都只用列表了。

我们不能通过编号直接访问一个元素是列表的主要缺陷，但对数组来说却很容易： `arr[n]` 就是直接访问元素的方法。但如果是一个列表的话，我们必须从列表的第一个元素开始，通过 `N`次的 `next`找到第N个元素。

...但我们并不总是需要这样的操作。比方说，我们可以使用一个队列真是一个[deque](https://en.wikipedia.org/wiki/Double-ended_queue) -- 此类有序的结构，便于快速地从两端增加、删除元素。

有时候值得增加一个 `tail`变量来跟踪列表中的最后一个元素，当然从尾部增加、删除元素时需要更新它。当元素很多时，使用列表与使用数组的速度差别还是很大的。

## 总结

术语：
- *递归* 时一个编程术语，表示一个"调用自身"的函数。这种函数可以优雅地解决一些特定的问题。

    当一个函数调用自身时，这叫着一个 *递归步骤*。 递归的*基础*是指简单到函数不用再做更多一次调用的函数参数。

- 一个递归式定义的[recursively-defined](https://en.wikipedia.org/wiki/Recursive_data_type) 数据结构是指可以使用自身来定义一种数据结构。

    例如：链式列表可以定义为指向列表或者空的对象构成的列表。

    ```js
    list = { value, next -> list }
    ```

    这个章节里面所谈到的HTML元素树或者部门树都是一种递归式的定义，他们有分支，且每个分支可以有更深一层的分支。

    如果求薪水和`sumSalary` 例子所示，递归函数可以用来遍历他们。

所有的递归式函数都可以被改写成迭代式的，特别是在要求做优化时。但递归式方案对于很多解决方案来说已经够快，并且更加容易编写与维护。
