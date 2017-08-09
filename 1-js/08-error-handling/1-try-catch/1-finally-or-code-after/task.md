重要程度: 5

---

# 要使用 finally 吗?

比较两个代码片段.

1. 第一个使用 `finally` 来执行 `try..catch` 后的代码:

    ```js
    try {
      work work
    } catch (e) {
      handle errors
    } finally {
    *!*
      cleanup the working space
    */!*
    }
    ```
2. 第二个片段把清理工作放到 `try..catch` 之后:

    ```js
    try {
      work work
    } catch (e) {
      handle errors
    }

    *!*
    cleanup the working space
    */!*
    ```

在工作开始后, 我们确实需要进行清理工作, 无论是否发生了错误.

使用 `finally` 有什么优点或者两个代码片段功能一致? 如果有优点, 给出一个例子说明它什么时候很重要.
