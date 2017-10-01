重要度: 5

---

# Sum all numbers till the given one

编写一个函数： `sumTo(n)`，计算以下数字的和： `1 + 2 + ... + n`.

例如:

```js no-beautify
sumTo(1) = 1
sumTo(2) = 2 + 1 = 3
sumTo(3) = 3 + 2 + 1 = 6
sumTo(4) = 4 + 3 + 2 + 1 = 10
...
sumTo(100) = 100 + 99 + ... + 2 + 1 = 5050
```

使用三种不同的解法:

1. 使用循环.
2. 使用递归，在`'n > 1`时， `sumTo(n) = n + sumTo(n-1)` .
3. 使用数学公式 [arithmetic progression](https://en.wikipedia.org/wiki/Arithmetic_progression) formula.

结果示例:

```js
function sumTo(n) { /*... your code ... */ }

alert( sumTo(100) ); // 5050
```

附注：那种方案最快？那种最慢？为啥？

又注：我们能用递归方法计算 `sumTo(100000)`吗? 
