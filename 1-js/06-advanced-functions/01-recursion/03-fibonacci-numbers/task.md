重要度: 5

---

# Fibonacci numbers

满足公式 <code>F<sub>n</sub> = F<sub>n-1</sub> + F<sub>n-2</sub></code> 的数列被称为： [Fibonacci numbers](https://en.wikipedia.org/wiki/Fibonacci_number) ， 也即：下一个数是前两个数的和。

最前的两个数都是 `1`, 接着是 `2(1+1)`, 然后是 `3(1+2)`, `5(2+3)` ，以此类推，后续的数字为: `1, 1, 2, 3, 5, 8, 13, 21...`.

斐波拉契数列与黄金分割 [Golden ratio](https://en.wikipedia.org/wiki/Golden_ratio) 以及很多身边的自然现象有关。

请编写一个返回第n `n-th` 个斐波拉契数字的函数 `fib(n)` 。

示例:

```js
function fib(n) { /* your code */ }

alert(fib(3)); // 2
alert(fib(7)); // 13
alert(fib(77)); // 5527939700884757
```

附注： 这个函数运行起来应该足够快，调用 `fib(77)` 所需的时间应该不超过一秒钟。
