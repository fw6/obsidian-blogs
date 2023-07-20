---
title: "JavaScript尾后逗号"
description: ""
pubDate: "2023-07-19 21:41"
heroImage: "https://raw.githubusercontent.com/fw6/assets/main/toy_docs/20230719215441.png"
date created: 2023-07-19
date modified: 2023-07-19
tags: 
    - notes
    - JavaScript
    - Programming
---

## TOC

ECMAScript 源码文本会被从左到右扫描，并被转换为一系列的输入元素，包括 token、控制符、行终止符、注释和空白符。ECMAScript 定义了一些关键字、字面量以及行尾分号补全的规则。

### 格式控制符

格式控制符用于控制对源码文本的解释，但是并不会显示出来。

| 码点   | 名称                  | 缩写     | 说明 |
| ------ | --------------------- | -------- | ---- |
| U+200C | zero width non-joiner | \<ZWNJ\> |  放置在一些经常会被当成连字的字符之间，用于将它们分别以独立形式显示（[Wikipedia](http://en.wikipedia.org/wiki/Zero-width_non-joiner)）    |
| U+200D | zero width joiner | \<ZWJ\> |   放置在一些通常不会被标记为连字的字符之间，用于将这些字符以连字形式显示（[Wikipedia](http://en.wikipedia.org/wiki/Zero-width_joiner)）   |
| U+FEFF       |      Byte order mark                 |     \<BOM\>     | 在脚本开头使用，除了将脚本标记为 Unicode 格式以外，还用来标记文本的字节流方向（[Wikipedia](http://en.wikipedia.org/wiki/Byte_order_mark)）     |
在JavaScript源码中，`<ZWNJ>`和`<ZWJ>`被视为标识符部分，而（当不在文本开头时也被称为零宽不间断空格）被视为空白符。


## 空白符

空白符提升了源码的可读性，并将标记 (tokens) 区分开。这些符号通常不影响源码的功能。通常可以用[压缩器](http://en.wikipedia.org/wiki/Minification_%28programming%29)来移除源码中的空白，减少数据传输量。

| 代码点 | 名称              | 缩写     | 说明                                                                                                        | 转义序列 |
| ------ | ----------------- | -------- | ----------------------------------------------------------------------------------------------------------- | -------- |
| U+0009 | 制表符            | `<HT>`   | 水平制表符                                                                                                  | \t       |
| U+000B | 垂直制表符        | `<VT>`   | 垂直制表符                                                                                                  | \v       |
| U+000C | 分页符            | `<FF>`   | 分页符（[Wikipedia](http://en.wikipedia.org/wiki/Page_break#Form_feed)）                                    | \f       |
| U+0020 | 空格              | `<SP>`   | 空格                                                                                                        |          |
| U+00A0 | 无间断空格        | `<NBSP>` | 在该空格处不会换行                                                                                          |          |
| Others | 其他 Unicode 空白 | `<USP>`  | [Wikipedia 上对 Unicode 空白的介绍](http://en.wikipedia.org/wiki/Space_%28punctuation%29#Spaces_in_Unicode) |          |


## 行终止符

除了空白符之外，行终止符也可以提高源码的可读性。不同的是，行终止符可以影响 JavaScript 代码的执行。行终止符也会影响[自动分号补全](#自动分号补全)的执行。在正则表达式中，行终止符会被 `\s` 匹配。

在 ECMAScript 中，只有下列 Unicode 字符会被当成行终止符，其他的行终止符（比如 Next Line、NEL、U+0085 等）都会被当成空白符。

| 编码   | 名称     | 缩写   | 说明                                              | 转义序列 |
| ------ | -------- | ------ | ------------------------------------------------- | -------- |
| U+000A | 换行符   | `<LF>` | 在 UNIX 系统中起新行                              | \n       |
| U+000D | 回车符   | `<CR>` | 在 Commodore 和早期的 Mac 系统中起新行            | \r       |
| U+2028 | 行分隔符 | `<LS>` | [Wikipedia](http://en.wikipedia.org/wiki/Newline) |          |
| U+2029 | 段分隔符 | `<PS>` | [Wikipedia](http://en.wikipedia.org/wiki/Newline) |          |

## 注释

注释用来在源码中增加提示、笔记、建议、警告等信息，可以帮助阅读和理解源码。在调试时，可以用来将一段代码屏蔽掉，防止其运行；这也是一个有价值的调试工具。

在 JavaScript 中，有两种常见的添加注释的方法：行注释和块注释。另外，也有一种特殊的 hashbang 注释语法。

### 行注释

是`//` 风格的注释，会将该行中符号以后的文本都视为注释：
```js
function comment() {
  // 这是单行注释
  console.log("Hello world!");
}
comment();
```

### 块注释
是 `/* */` 风格的注释，这种方式更加灵活，可以在单行内使用多行注释，也可以用来实现多行注释，也可以用于行内注释：
```js
function comment(x) {
  /* 这是单行注释 */
  
  /* 
    这是多行注释， 
    注意在写完注释前无需终止注释 
  */
  
  console.log("Hello " + x /* 引入 x 的值 */ + " !");
}
comment("world");
```

### Hashbang注释

**Hashbang 注释**是一种特殊的注释语法，其行为与单行注释（`//`）完全一样，只是它以 `#!` 开头，并且**只在脚本或模块的最开始处有效**。注意，`#!` 标志之前不能有任何空白字符。注释由 `#!` 之后的所有字符组成直到第一行的末尾；只允许有一条这样的注释。JavaScript 中的 hashbang 注释类似于 [Unix 中的 shebang](https://zh.wikipedia.org/wiki/Shebang)，它提供了一个特定的 JavaScript 解释器的路径，你想用它来执行这个脚本。在 hashbang 注释标准化之前，它已经在非浏览器主机（如 Node.js）中得到了事实上的实现，在那里，它在被传递给引擎之前被从源文本中剥离。示例如下：
```js
#!/usr/bin/env node
console.log("Hello world");
```

下面是`zx`JavaScript解释器的示例：
```js
#!/usr/bin/env zx
await $`cat package.json | grep name`
```

## 关键字

### [ECMAScript 6 中的保留关键字](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Lexical_grammar#ecmascript_6_%E4%B8%AD%E7%9A%84%E4%BF%9D%E7%95%99%E5%85%B3%E9%94%AE%E5%AD%97)

- [`break`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/break)
- [`case`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/switch)
- [`catch`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/try...catch)
- [`class`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/class)
- [`const`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/const)
- [`continue`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/continue)
- [`debugger`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/debugger)
- [`default` (en-US)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/switch "Currently only available in English (US)")
- [`delete`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/delete)
- [`do`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/do...while)
- [`else`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/if...else)
- [`export`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/export)
- [`extends`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/class)
- [`finally`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/try...catch)
- [`for`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/for)
- [`function`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function)
- [`if`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/if...else)
- [`import`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/import)
- [`in`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/in)
- [`instanceof`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/instanceof)
- [`new`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/new)
- [`return`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/return)
- [`super`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/super)
- [`switch`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/switch)
- [`this`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/this)
- [`throw`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/throw)
- [`try`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/try...catch)
- [`typeof`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/typeof)
- [`var`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/var)
- [`void`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/void)
- [`while`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/while)
- [`with`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/with)
- [`yield`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/yield)

### [未来保留关键字](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Lexical_grammar#%E6%9C%AA%E6%9D%A5%E4%BF%9D%E7%95%99%E5%85%B3%E9%94%AE%E5%AD%97)

在 ECMAScript 规格中，这些关键字被当成关键字保留。目前它们没有特殊功能，但是在未来某个时间可能会加上。所以这些关键字不能当成标识符使用。这些关键字在严格模式和非严格模式中均不能使用。

- `enum`

以下关键字只在严格模式中被当成保留关键字：

- `implements`
- `interface`
- [`let`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/let)
- `package`
- `private`
- `protected`
- `public`
- `static`

以下关键字只在模块代码中被当成保留关键字：

- `await`

#### 之前标准中的保留关键字

这里是之前版本中的 ECMAScript（1 到 3）中的保留关键字：

- `abstract`
- `boolean`
- `byte`
- `char`
- `double`
- `final`
- `float`
- `goto`
- `int`
- `long`
- `native`
- `short`
- `synchronized`
- `transient`
- `volatile`

另外，字面量 `null`、`true`和`false`同样不能被当成标识使用。
保留字是仅针对标识符（Identifier）的文法定义而言的（而不是标识符名（IdentifierName）的文法定义）。

```js
// 标识符用于函数声明式和函数表达式。
function import() {} // Illegal.
```

## 字面量

### 空字面量

```js
null
```

### 布尔字面量
```js
true
false
```

### 数值字面量

```js
// 十进制

1234567890
42
// 谨慎使用 0 开头的数值：
0888 // 转换为十进制 888
0777 // 转换为八进制 777，十进制 511

// 二进制
var FLT_SIGNBIT  = 0b10000000000000000000000000000000; // 2147483648
var FLT_EXPONENT = 0b01111111100000000000000000000000; // 2139095040
var FLT_MANTISSA = 0B00000000011111111111111111111111; // 8388607

// 八进制
var n = 0O755; // 493
var m = 0o644; // 420

// 十六进制
0xFFFFFFFFFFFFFFFFF // 295147905179352830000
0x123456789ABCDEF   // 81985529216486900
0XA                 // 10
```

>[!warning]
>十进制数值字面量可以以 0 开头，但是如果 0 以后的最高位比 8 小，数值将会被认为是八进制而不会报错。

### 对象字面量

```js
var o = { a: "foo", b: "bar", c: 42 };

// ES6 中的简略表示方法
var a = "foo", b = "bar", c = 42;
var o = {a, b, c};
// 不需要这样
var o = { a: a, b: b, c: c };
```

### 数组字面量

```js
[1954, 1974, 1990, 2014]
```

### 字符串字面量



## 自动分号补全

