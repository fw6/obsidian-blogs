---
title: "JavaScript尾后逗号"
description: ""
pubDate: "2023-07-19 21:41"
heroImage: "https://raw.githubusercontent.com/fw6/assets/main/toy_docs/20230719214259.png"
date created: 2023-07-19
date modified: 2023-07-19
tags: 
    - notes
    - JavaScript
    - Programming
---

**尾后逗号** （有时叫做“终止逗号”）在向 JavaScript 代码添加元素、参数、属性时十分有用。如果你想要添加新的属性，并且上一行已经使用了尾后逗号，你可以仅仅添加新的一行，而不需要修改上一行。这使得版本控制的代码比较（diff）更加清晰，代码编辑过程中遇到的麻烦更少。

```js
const arr = [
    1,
    2,
    3, // final commas
];
arr.length; // 3

const obj = {
    a: 'a',
    b: 'b', // final commas
};

const func = (
    a,
    b,
    c,
) => {};

Math.max(10, 20,);
```

JavaScript 一开始就支持数组字面量中的尾后逗号，随后向对象字面量（ECMAScript 5）中添加了尾后逗号。最近（ECMAScript 2017），又将其添加到函数参数中。

但是，[JSON](https://developer.mozilla.org/zh-CN/docs/Glossary/JSON) 不允许尾后逗号！

### 不合法的尾后逗号

仅仅包含逗号的函数参数定义或者函数调用会抛出 `SyntaxError`。而且，当使用剩余参数的时候，并不支持尾后逗号：
```js
function f(,) {} // SyntaxError: missing formal parameter
(,) => {};       // SyntaxError: expected expression, got ','
f(,)             // SyntaxError: expected expression, got ','

function f(...p,) {} // SyntaxError: parameter after rest parameter
(...p,) => {}        // SyntaxError: expected closing parenthesis, got ','
```

### JSON中的尾后逗号

对象中的尾后逗号仅仅在 ECMAScript 5 中引入。由于 JSON 基于 ES5 之前的语法， **JSON 中并不允许尾后逗号**。

下面两行都会抛出 `SyntaxError`：
```js
JSON.parse('[1, 2, 3, 4, ]');
JSON.parse('{"foo" : 1, }');
// SyntaxError JSON.parse: unexpected character
// at line 1 column 14 of the JSON data
```
