# 模式和标志

正则表达式是一种在字符串内进行搜索和替换的强大方式。

在 JavaScript 里，正则表达式是以内建类型 `RegExp` 实现，并集成在 string 里。

值得注意的是，正则表达式因编程语言而异。在本教程中，我们只专注于 JavaScript。诚然，它在 Perl、Ruby、PHP 等语言中有很多相似之处，但也略有所不同。

[cut]

## 正则表达式

一个正则表达式 (也称 "regexp"， 或简称 "reg") 由一个 *pattern* 和一个可选项 *flags* 组成。

有两种语法来创建正则表达式对象。

长语法：


```js
regexp = new RegExp("pattern", "flags");
```

...和短语法, 使用斜杠 `"/"`:

```js
regexp = /pattern/; // 没有标志
regexp = /pattern/gmi; // 有标志 g、m 和 i (后文会介绍)
```

斜杠 `"/"` 告诉 JavaScript 我们在创建一个正则表达式。它们与字符串的引号起着相同的作用。

## 用法

要搜索一个字符串, 我们可以使用 [search](mdn:js/String/search) 方法。

举个例子：

```js run
let str = "I love JavaScript!"; // 从此搜索

let regexp = /love/;
alert( str.search(regexp) ); // 2
```

`str.search` 方法寻找模式 `pattern:/love/` 并返回其在字符串中的位置。 正如我们所猜想的那样, `pattern:/love/` 是最简单的一种模式. 它所做的是一个简单的字符串搜索。

上述代码和下面效果一致：

```js run
let str = "I love JavaScript!"; // 从此搜索

let substr = 'love';
alert( str.search(substr) ); // 2
```

所以搜索 `pattern:/love/` 与搜索 `"love"` 相同。

但仅限于此。很快，我们将创建更复杂的、拥有更强搜索能力正则表达式。

```smart header="Colors"
后文的配色方案:

- 正则式 -- `pattern:red`
- 字符串 (从哪里搜索) -- `subject:blue`
- 结果 -- `match:green`
```


````smart header="何时使用 `new RegExp`？"
通常我们使用短语法 `/.../`。但它不允许任何变量插入，所以我们在编写代码时必须实时知道其准确的正则式。

然而，`new RegExp` 允许我们使用字符串动态创建模式。

所以我们可以动态确定我们需要搜索什么，然后从中创建 `new RegExp`

```js run
let search = prompt("What you want to search?", "love");
let regexp = new RegExp(search);

// 查找任何用户想要的东西
alert( "I love JavaScript".search(regexp));
```
````


## 标志

正则表达式可以包含一些影响其搜索的标志。

在 JavaScript 这类标志一共有5种。

`i`
：大小写不敏感 : `A` 与 `a` 之间没有不同（例子见下文）。

`g`
 使用此标志会搜索所有的匹配, 反之只搜索第一个匹配 (详见下章)。

`m`
: 多行模式（在章节 <info:regexp-multiline> 里讲述）。

`u`
: 开启全 unicode 支持。 此标志可正确处理代理对。详见 <info:regexp-unicode> 章节。

`y`
: 粘连模式（详见[下章]（info:regexp-methods#y-flag））


## "i" 标志

`i` 是最简单的标志

一个例子:

```js run
let str = "I love JavaScript!";

alert( str.search(/LOVE/) ); // -1 (not found)
alert( str.search(/LOVE/i) ); // 2
```

1. 第一个搜索返回 `-1`（未找到）， 因为搜索默认是大小写敏感的。
2. 使用标志的 `pattern:/LOVE/i` 搜索会在位置 2 处找到 `match:love` 。

所以标志 `i` 已经让正则表达式比简单的字符串搜索更加强大。但正则表达式的威力远不止如此，我们会在接下来的章节讲解其它的标志和功能。


## 概要

- 一个正则表达式由一个模式和一些可选标志（`g`, `i`, `m`, `u`, `y`）组成。
- 没有标志和一些我们即将学到的特殊符号，正则式搜索就和字符串搜索一样。
- 方法 `str.search(regexp)` 会返回匹配位置，或者如果没有匹配，则返回 -1 。
