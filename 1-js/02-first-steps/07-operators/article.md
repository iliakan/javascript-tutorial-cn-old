# 运算符

我们在学校学过很多运算符，例如加号`+`，乘号`*`，减号`-`等等。

在本章中，我们将重点放在学校算术未涉及的部分。
[cut]

## 术语: "一元", "二元", "操作数"

在我们继续学习之前，先来了解一些通用术语。

- *操作数* -- 是运算的对象。例如在 `5 * 2`这个乘法运算中就有2个操作数:左操作数 
`5`, 和右操作数`2`。有时也会把操作数称为参数。
- 一个运算符如果只有一个操作数则为一元运算符。例如一元求反运算符`-`可以反转一个数字的正负：

    ```js run
    let x = 1;

    *!*
    x = -x;
    */!*
    alert( x ); // -1, unary minus was applied
    ```
- 一个运算符如果有两个操作数则为二元运算符，例如减号：

    ```js run no-beautify
    let x = 1, y = 3;
    alert( y - x ); // 2, binary minus substracts values
    ```
    
    形式上，这里我们讨论的是两个不同的运算符(虽然它们符号相同)：一元求反运算符(只有一个操作数，反转它的符号)和二元运算符减号(两个操作数，作减法)。

## 使用二元运算符`+`连接字符串

现在让我们来看看JavaScript运算符的特别功能，和学校所学的不太一样。

通常我们使用`+`对两个数字进行加法运算。

但如果`+`应用于两个字符串，则会合并它们(连接)：

```js
let s = "my" + "string";
alert(s); // mystring
```

注意，如果有任意一个操作数是字符串，那么另外一个操作数也会转换成字符串

例如:

```js run
alert( '1' + 2 ); // "12"
alert( 2 + '1' ); // "21"
```

看到了吗，第一个操作数和第二个操作数谁是字符串并不重要。规则很简单：如果任一操作数是一个字符串，则将另一个操作数转换为一个字符串。

但是要注意一点，运算操作是从左往右的(满足数学运算法则)，如果在两个数字后面跟着一个字符串，那么这两个数字会先相加然后再转换为字符串：

```js
alert(2 + 2 + '1'); //"41" and not "221"
```

字符串的连接和转换是二元运算符`+`的特别功能。其他算术运算符只会对数字起作用。它们总会把操作数转化为数字。

例如减法和除法：

```js run
alert( 2 - '1' ); // 1
alert( '6' / '2' ); // 3
```

## Numeric conversion, unary +

##使用一元运算符`+`进行数字转换

加号`+`存在两种形态，上面我们使用的二元形态和下面的一元形态

一元形态的加号，或者说是加号运算符`+`应用于一个操作数的时候，不会对这个操作数做任何操作，但如果这个操作数不是一个数字，那么就会把它转换为数字。

例如:

```js run
// No effect on numbers
let x = 1;
alert( +x ); // 1

let y = -2;
alert( +y ); // -2

*!*注意
// 转换了一个非数字
alert( +true ); // 1
alert( +"" );   // 0
*/!*
```

这和使用`Number(..)`的结果是一致的，但更简洁。

将字符串转换为数字的需求是很常见的。假设我们从HTML输入域获取到的一些值，这些值一般都是字符串。

这个时候如果我们将这些值相加，会怎样？

二元加号会把它们连接成一个字符串：

```js run
let apples = "2";
let oranges = "3";

alert( apples + oranges ); // "23", the binary plus concatenates strings
```

如果想把它们当作一个数字对待，那我们可以先进行转换然后再相加：

```js run
let apples = "2";
let oranges = "3";

*!*
// both values converted to numbers before the binary plus
alert( +apples + +oranges ); // 5
*/!*

// the longer variant
// alert( Number(apples) + Number(oranges) ); // 5
```

对于一个数学家而言，这样的行为似乎有点奇怪。但从一个程序员的角度来看，这并没有什么特别：一元加号首先将字符串转换为数字，然后二元加号将它们加起来。

为什么一元加号的操作先于二元加号？这就是下面要说的，它们之间存在优先级。

## 运算符的优先级

如果一个表达式有超过一个运算符，那么执行顺序由它们的优先级来决定，换句话说，运算符之间存在隐式的优先级。

我们都知道，表达式`1 + 2 * 2`中的乘法应该在加法之前计算。这正是一种优先关系。乘法比加法有更高的优先级。

圆括号会覆盖所有优先级(即圆括号拥有最高优先级)，因此如果我们对现在的执行顺序不满意，我们可以使用圆括号来改变它，像这样：`(1 + 2) * 2`。

在JavaScript中有许多的运算符，每一个运算符都有一个与之对应的优先度，优先度较大的运算符先执行。如果两个运算符优先度相等，则从左往右执行。

这是一个优先度表[precedence table](https://developer.mozilla.org/en/JavaScript/Reference/operators/operator_precedence)
(你不需要记住所有的优先度，但是要注意一点，一元运算符的优先度高于其二元形态):

| Precedence | Name | Sign |
|------------|------|------|
| ... | ... | ... |
| 15 | unary plus | `+` |
| 15 | unary minus | `-` |
| 14 | multiplication | `*` |
| 14 | division | `/` |
| 13 | addition (binary) | `+` |
| 13 | subtraction | `-` |
| ... | ... | ... |
| 3 | assignment | `=` |
| ... | ... | ... |


如上表所示，一元加号(unary plus)的优先度是15，比"加法"(二元加号)的13要高。
这就是为什么表达式`"+apples + +oranges"`中的一元加号先执行，然后再做加法。

## 赋值

赋值`=`也是一个运算符，根据上面的优先度表所示，它的优先度是3。

这就是为什么当我们对一个变量进行赋值操作时，例如`x = 2 * 2 + 1`，会先进行计算，再进行赋值，把结果储存到`x`中

```js
let x = 2 * 2 + 1;

alert( x ); // 5
```

允许链式赋值：

```js run
let a, b, c;

*!*
a = b = c = 2 + 2;
*/!*

alert( a ); // 4
alert( b ); // 4
alert( c ); // 4
```


链式赋值的运算从右到左。首先最右边的表达式`2+2`先执行，然后赋值给左边的变量`c`，`b`，`a`。最终，所有的变量都会用到同一个值。

````smart header="赋值运算符`\"=\"`会返回一个值"
An operator always returns a value. That's obvious for most of them like an addition `+` or a multiplication `*`. But the assignment operator follows that rule too.

The call `x = value` writes the `value` into `x` *and then returns it*.

Here's the demo that uses an assignment as the part of a more complex expression:

```js run
let a = 1;
let b = 2;

*!*
let c = 3 - (a = b + 1);
*/!*

alert( a ); // 3
alert( c ); // 0
```

In the example above, the result of `(a = b + 1)` is the value which is assigned to `a` (that is `3`). It is then used to substract from `3`.

Funny code, isn't it? We should understand how it works, because sometimes we can see it in 3rd-party libraries, but shouldn't write anything like that ourselves. Such tricks definitely don't make the code clearer and readable.
````

## Remainder %

The remainder operator `%` despite it's look does not have a relation to percents.

The result of `a % b` is the remainder of the integer division of `a` by `b`.

For instance:

```js run
alert( 5 % 2 ); // 1 is a remainder of 5 divided by 2
alert( 8 % 3 ); // 2 is a remainder of 8 divided by 3
alert( 6 % 3 ); // 0 is a remainder of 6 divided by 3
```

## Exponentiation **

The exponentiation operator `**` is a recent addition to the language.

For a natural number `b`, the result of `a ** b` is `a` multiplied by itself `b` times.

For instance:

```js run
alert( 2 ** 2 ); // 4  (2 * 2)
alert( 2 ** 3 ); // 8  (2 * 2 * 2)
alert( 2 ** 4 ); // 16 (2 * 2 * 2 * 2)
```

The operator works for non-integer numbers of `a` and `b` as well, for instance:

```js run
alert( 4 ** (1/2) ); // 2 (power of 1/2 is the same as a square root, that's maths)
alert( 8 ** (1/3) ); // 2 (power of 1/3 is the same as a cubic root)
```

## Increment/decrement

<!-- Can't use -- in title, because built-in parse turns it into – -->

Increasing or decreasing a number by one is among the most common numerical operations.

So, there are special operators for that:

- **Increment** `++` increases a variable by 1:

    ```js run no-beautify
    let counter = 2;
    counter++;      // works same as counter = counter + 1, but shorter
    alert( counter ); // 3
    ```
- **Decrement** `--` decreases a variable by 1:

    ```js run no-beautify
    let counter = 2;
    counter--;      // works same as counter = counter - 1, but shorter
    alert( counter ); // 1
    ```

```warn
Increment/decrement can be applied only to a variable. An attempt to use it on a value like `5++` will give an error.
```

Operators `++` and `--` can be placed both after and before the variable.

- When the operator goes after the variable, it is called a "postfix form": `counter++`.
- The "prefix form" is when the operator stands before the variable: `++counter`.

Both of these records do the same: increase `i` by `1`.

Is there any difference? Yes, but we can only see it if we use the retured value of `++/--`.

Let's clarify. As we know, all operators return a value. Increment/decrement is not an exception here. The prefix form returns the new value, while the postfix form returns the old value (prior to increment/decrement).

To see the difference -- here's the example:

```js run
let counter = 1;
let a = ++counter; // (*)

alert(a); // *!*2*/!*
```

Here in the line `(*)` the prefix call `++counter` increments `i` and returns the new value that is `2`. So the `alert` shows `2`.

Now let's use the postfix form:

```js run
let counter = 1;
let a = counter++; // (*) changed ++counter to counter++

alert(a); // *!*1*/!*
```

In the line `(*)` the *postfix* form `counter++` also increments `i`, but returns the *old* value (prior to increment). So the `alert` shows `1`.

To summarize:

- If the result of increment/decrement is not used, then there is no difference which form to use:

    ```js run
    let counter = 0;
    counter++;
    ++counter;
    alert( counter ); // 2, the lines above did the same
    ```
- If we'd like to increase the value *and* use the result of the operator right now, then we need the prefix form:

    ```js run
    let counter = 0;
    alert( ++counter ); // 1
    ```
- If we'd like to increment, but use the previous value, then we need the postfix form:

    ```js run
    let counter = 0;
    alert( counter++ ); // 0
    ```

````smart header="Increment/decrement among other operators"
Operators `++/--` can be used inside an expression as well. Their precedence is higher than most other arithmetical operations.

For instance:

```js run
let counter = 1;
alert( 2 * ++counter ); // 4
```

Compare with:

```js run
let counter = 1;
alert( 2 * counter++ ); // 2, because counter++ returns the "old" value
```

Though technically allowable, such notation usually makes the code less readable. One line does multiple things -- not good.

While reading the code, a fast "vertical" eye-scan can easily miss such `counter++`, and it won't be obvious that the variable increases.

The "one line -- one action" style is advised:

```js run
let counter = 1;
alert( 2 * counter );
counter++;
```
````

## Bitwise operators

Bitwise operators treat arguments as 32-bit integer numbers and work on the level of their binary representation.

These operators are not JavaScript-specific. They are supported in most programming languages.

The list of operators:

- AND ( `&` )
- OR ( `|` )
- XOR ( `^` )
- NOT ( `~` )
- LEFT SHIFT ( `<<` )
- RIGHT SHIFT ( `>>` )
- ZERO-FILL RIGHT SHIFT ( `>>>` )

These operators are used very rarely. To understand them, we should delve into low-level number representation, and it would not be optimal to do that right now. Especially because we won't need them any time soon. If you're curious, you can read the [Bitwise Operators](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Bitwise_Operators) article in MDN. It would be more practical to do that when a real need arises.

## Modify-in-place

We often need to apply an operator to a variable and store the new result in it.

For example:

```js
let n = 2;
n = n + 5;
n = n * 2;
```

This notation can be shortened using operators `+=` and `*=`:

```js run
let n = 2;
n += 5; // now n=7 (same as n = n + 5)
n *= 2; // now n=14 (same as n = n * 2)

alert( n ); // 14
```

Short "modify-and-assign" operators exist for all arithmetical and bitwise operators: `/=`, `-=` etc.

Such operators have the same precedence as a normal assignment, so they run after most other calculations:

```js run
let n = 2;

n *= 3 + 5;

alert( n ); // 16  (right part evaluated first, same as n *= 8)
```

## Comma

The comma operator `','` is one of most rare and unusual operators. Sometimes it's used to write shorter code, so we need to know it in order to understand what's going on.

The comma operator allows to evaluate several expressions, dividing them with a comma `','`. Each of them is evaluated, but result of only the last one is returned.

For example:

```js run
*!*
a = (1+2, 3+4);
*/!*

alert( a ); // 7 (the result of 3+4)
```

Here, the first expression `1+2` is evaluated, and it's result is thrown away, then `3+4` is evaluated and returned as the result.

```smart header="Comma has a very low precedence"
Please note that the comma operator has very low precedence, lower than `=`, so parentheses are important in the example above.

Without them: `a=1+2,3+4` evaluates `+` first, summing the numbers into `a=3,7`, then the assignment operator `=` assigns `a=3`, and then the number after the comma `7` is not processed anyhow, so it's ignored.
```

Why do we need such an operator which throws away everything except the last part?

Sometimes people use it in more complex constructs to put several actions in one line.

For example:

```js
// three operations in one line
for (*!*a = 1, b = 3, c = a * b*/!*; a < 10; a++) {
 ...
}
```

Such tricks are used in many JavaScript frameworks, that's why we mention about them. But usually they don't improve the code readability, so we should think well before writing like that.
