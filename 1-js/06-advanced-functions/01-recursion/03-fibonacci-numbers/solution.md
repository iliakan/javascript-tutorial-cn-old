第一个能够应用的解法是使用递归。

斐波拉契数列按照定义是递归性质的:

```js run
function fib(n) {
  return n <= 1 ? n : fib(n - 1) + fib(n - 2);
}

alert( fib(3) ); // 2
alert( fib(7) ); // 13
// fib(77); // will be extremely slow!
```

...但这种解法在 `n` 很大时会非常慢。 例如： `fib(77)` 会将引擎挂起一段时间并消耗所有的 CPU 资源。

这是因为这样的函数需要调用太多次，相同的数字需要一再被计算。

以 `fib(5)` 为例，来看看有哪些调用与计算发生：

```js no-beautify
...
fib(5) = fib(4) + fib(3)
fib(4) = fib(3) + fib(2)
...
```

这里我们可以看到计算 `fib(5)` 与 `fib(4)` 时，都需要 `fib(3)` 的值。所有 `fib(3)` 需要至少两次完全独立的调用与计算。

完整的递归调用可以从这里的树状图看出：

![fibonacci recursion tree](fibonacci-recursion-tree.png)

我们可以很清晰地看到， `fib(3)` 被计算了两次，且 `fib(2)` 被计算了三次。 总的计算次数增长的速度超过了 `n` 的增长速度, 当 `n=77` 时这个差异就很明显了。

我们可以通过记住已经计算过的数字的方式来优化： 如果 `fib(3)` 的结果已经计算出，那么我们可以记住它并在以后的计算中直接使用。

另外一种变通方案是放弃递归，使用完全基于循环的算法。

与从 `n` 由大到小计算的方案相反，我们可以从 `1` 和 `2` 开始循环，得到 `fib(3)` 的值，然后根据之前两个值的和计算出 `fib(4)` ，然后计算出 `fib(5)` ，以此类推，直到计算到所需要的值。用这种方法，我们就只需要记下前两个值。

新算法的计算步骤如下。

开始：

```js
// a = fib(1), b = fib(2), these values are by definition 1
let a = 1, b = 1;

// get c = fib(3) as their sum
let c = a + b;

/* we now have fib(1), fib(2), fib(3)
a  b  c
1, 1, 2
*/
```

现在我们想通过 `fib(4) = fib(2) + fib(3)` 得到 `4` 的计算结果。

我们可以将 `fib(2),fib(3)` 在前一步计算出来的结果放入 `a,b` ， 那么 `c` 所包含的就是他们的和：

```js no-beautify
a = b; // now a = fib(2)
b = c; // now b = fib(3)
c = a + b; // c = fib(4)

/* now we have the sequence:
   a  b  c
1, 1, 2, 3
*/
```

再继续一轮，就可以得到下一个数列的值：

```js no-beautify
a = b; // now a = fib(3)
b = c; // now b = fib(4)
c = a + b; // c = fib(5)

/* now the sequence is (one more number):
      a  b  c
1, 1, 2, 3, 5
*/
```

以此类推，直到我们到达想要计算的值。 这种方法就比递归快很多，并且没有重复的计算。

完整的代码如下：

```js run
function fib(n) {
  let a = 1;
  let b = 1;
  for (let i = 3; i <= n; i++) {
    let c = a + b;
    a = b;
    b = c;
  }
  return b;
}

alert( fib(3) ); // 2
alert( fib(7) ); // 13
alert( fib(77) ); // 5527939700884757
```

因为数列中的第一个、第二个值已经通过变量赋值 `a=1`, `b=1` 进行了硬编码，因此循环可以从 `i=3` 开始。

这种方法又称之为：自底向上的动态编程 [dynamic programming bottom-up](https://en.wikipedia.org/wiki/Dynamic_programming) 。
