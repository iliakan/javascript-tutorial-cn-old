# 数据类型

JavaScript 中的变量可以保存任何数据。一个变量在前一刻可以是个字符串，然后又收到一个数值：

```js
// 没有错误
let message = "hello";
message = 123456;
```

允许这种操作的编程的语言称之为“动态类型”（dynamically typed）的编程语言，意思是，拥有数据类型，但是变量并不限于数据类型中的任何一个。

在 JavaScript 中有七种基本数据类型。这一章我们会学习基本知识，下一章我们会详细介绍它们。

[cut]

## number 类型

```js
let n = 123;
n = 12.345;
```

*number* 类型用于整数和浮点数。

数字有很多操作，比如，乘法 `*`，除法 `/`，加法 `+`，减法 `-` 等等。

除了常规的数字，还包括所谓的 “特殊数值” 也属于这种类型：`Infinity`, `-Infinity` 和 `NaN`。

- `Infinity` 代表数学概念中的[无限](https://en.wikipedia.org/wiki/Infinity) ∞。是一个比任何数字都大的特殊值。

    我们可以通过除以0来得到它。

    ```js run
    alert( 1 / 0 ); // Infinity
    ```

    或者在代码中直接提及它。

    ```js run
    alert( Infinity ); // Infinity
    ```
- `NaN` 代表一个计算错误。它是一个不对的或者一个未定义的数学操作所得到的结果，比如：

    ```js run
    alert( "not a number" / 2 ); // NaN, 这样的除法是错误的
    ```

    `NaN` 是粘性的。任何对 `NaN` 的进一步操作都会给出 `NaN`：

    ```js run
    alert( "not a number" / 2 + 5 ); // NaN
    ```

    所以，如何在数学表达式有一个 `NaN`，它会传播到最终结果。

```smart header="数学运算是安全的"
在 JavaScript 中做数学运算是安全的。我们可以做任何事：除以0，将非数字字符串视为数字，等等。

脚本永远不会有致命的错误("死亡")。最坏的情况下，会得到 `NaN` 作为结果。
```

特殊的数值属于*number*类型。当然，在这个词一般认识下，它们并不是数字。

我们将在章节 <info:number> 了解更多有关使用数字的内容。

## A string

A string in JavaScript must be quoted.

```js
let str = "Hello";
let str2 = 'Single quotes are ok too';
let phrase = `can embed ${str}`;
```

In JavaScript, there are 3 types of quotes.

1. Double quotes: `"Hello"`.
2. Single quotes: `'Hello'`.
3. Backticks: <code>&#96;Hello&#96;</code>.

Double and single quotes are "simple" quotes. There's no difference between them in JavaScript.

Backticks are "extended functionality" quotes. They allow to embed variables and expressions into a string by wrapping them in `${…}`, for example:

```js run
let name = "John";

// embed a variable
alert( `Hello, *!*${name}*/!*!` ); // Hello, John!

// embed an expression
alert( `the result is *!*${1 + 2}*/!*` ); // the result is 3
```

The expression inside `${…}` is evaluated and the result becomes a part of the string. We can put anything there: a variable like `name` or an arithmetical expression like `1 + 2` or something more complex.

Please note that this only can be done in backticks, other quotes do not allow such embedding!
```js run
alert( "the result is ${1 + 2}" ); // the result is ${1 + 2} (double quotes do nothing)
```

We'll cover strings more thoroughly in the chapter <info:string>.

```smart header="There is no *character* type."
In some languages, there is a special "character" type for a single character. For example, in the C language and in Java it is `char`.

In JavaScript, there is no such type. There's only one type: `string`. A string may consist of only one character or many of them.
```

## A boolean (logical type)

The boolean type has only two values: `true` and `false`.

This type is commonly used to store yes/no values: `true` means "yes, correct", and `false` means the "no, incorrect".

For instance:

```js
let nameFieldChecked = true; // yes, name field is checked
let ageFieldChecked = false; // no, age field is not checked
```

Boolean values also come as a result of comparisons:

```js run
let isGreater = 4 > 1;

alert( isGreater ); // true (the comparison result is "yes")
```

We'll cover booleans more deeply later in the chapter <info:logical-operators>.

## The "null" value

The special `null` value does not belong to any type of those described above.

It forms a separate type of its own, which contains only the `null` value:

```js
let age = null;
```

In JavaScript `null` is not a "reference to a non-existing object" or a "null pointer" like in some other languages.

It's just a special value which has the sense of "nothing", "empty" or "value unknown".

The code above states that the `age` is unknown or empty for some reason.

## The "undefined" value

The special value `undefined` stands apart. It makes a type of its own, just like `null`.

The meaning of `undefined` is "value is not assigned".

If a variable is declared, but not assigned, then its value is exactly `undefined`:

```js run
let x;

alert(x); // shows "undefined"
```

Technically, it is possible to assign any variable to `undefined`:

```js run
let x = 123;

x = undefined;

alert(x); // "undefined"
```

...But it's not recommended to do that. Normally, we use `null` to write an "empty" or an "unknown" value into the variable, and `undefined` is only used for checks, to see if the variable is assigned or similar.

## Objects and Symbols

The `object` type is special.

All other types are called "primitive", because their values can contain only a single thing (be it a string or a number or whatever). In contrast, objects are used to store collections data and more complex entities. We'll deal with them later in the chapter <info:object> after we know enough about primitives.

The `symbol` type is used to create unique identifiers for objects. We have to mention it here for completeness, but it's better to study them after objects.

## The typeof operator [#type-typeof]

The `typeof` operator returns the type of the argument. It's useful when we want to process values of different types differently, or just want to make a quick check.

It supports two forms of syntax:

1. As an operator: `typeof x`.
2. Function style: `typeof(x)`.

In other words, it works both with the brackets or without them. The result is the same.

The call to `typeof x` returns a string with the type name:

```js
typeof undefined // "undefined"

typeof 0 // "number"

typeof true // "boolean"

typeof "foo" // "string"

typeof Symbol("id") // "symbol"

*!*
typeof Math // "object"  (1)
*/!*

*!*
typeof null // "object"  (2)
*/!*

*!*
typeof alert // "function"  (3)
*/!*
```

The last three lines may need additional explanations:

1. `Math` is a built-in object that provides mathematical operations. We will learn it in the chapter <info:number>. Here it serves just as an example of an object.
2. The result of `typeof null` is `"object"`. That's wrong. It is an officially recognized error in `typeof`, kept for compatibility. Of course, `null` is not an object. It is a special value with a separate type of its own. So, again, that's an error in the language.
3. The result of `typeof alert` is `"function"`, because `alert` is a function of the language. We'll study functions in the next chapters, and we'll see that there's no special "function" type in the language. Functions belong to the object type. But `typeof` treats them differently. Formally, it's incorrect, but very convenient in practice.


## Summary

There are 7 basic types in JavaScript.

- `number` for numbers of any kind: integer or floating-point.
- `string` for strings. A string may have one more more characters, there's no separate single-character type.
- `boolean` for `true`/`false`.
- `null` for unknown values -- a standalone type that has a single value `null`.
- `undefined` for unassigned values -- a standalone type that has a single value `undefined`.
- `object` for more complex data structures.
- `symbol` for unique identifiers.

The `typeof` operator allows to see which type is stored in the variable.

- Two forms: `typeof x` or `typeof(x)`.
- Returns a string with the name of the type, like `"string"`.
- For `null` returns `"object"` -- that's the error in the language, it's not an object in fact.

In the next chapters we'll concentrate on primitive values and once we're familiar with that, then we'll move on to objects.
