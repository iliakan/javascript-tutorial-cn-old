根据定义，阶乘 `n!` 等于 `n * (n-1)!`.

换言之，函数 `factorial(n)` 的结果可以通过 `n` 乘以函数 `factorial(n-1)`的结果. 然后针对 `n-1` 的计算可以以此类推，递归地降低下来直到 `1`.

```js run
function factorial(n) {
  return (n != 1) ? n * factorial(n - 1) : 1;
}

alert( factorial(5) ); // 120
```

递归的基本值可以是 `1`. 我们也可以选择 `0` 作为基本值, 除了多算一步之外没有差别（ `0` 的阶乘结果是 `1`:

```js run
function factorial(n) {
  return n ? n * factorial(n - 1) : 1;
}

alert( factorial(5) ); // 120
```
