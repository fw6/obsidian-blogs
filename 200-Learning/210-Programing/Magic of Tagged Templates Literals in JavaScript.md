---
title: "Magic of Tagged Templates Literals in JavaScript"
description: ""
pubDate: "2023-07-19 16:23"
heroImage: "https://images.unsplash.com/photo-1626544827763-d516dce335e2?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D&auto=format&fit=crop&w=1200&q=80"
date created: 2023-07-19
date modified: 2023-07-19
tags:
    - writings
    - JavaScript
    - Programming
---

熟悉`styled-component`的一定对如下写法印象深刻：
```js
const Button = styled.button`
  background: transparent;
  border-radius: 3px;
  border: 2px solid #BF4F74;
  color: #BF4F74;
  margin: 0 1em;
  padding: 0.25em 1em;
`
```
在其官方也揭开了谜底：
>This unusual backtick syntax is a new JavaScript feature called a [tagged template literal](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals#Tagged_templates).

带标签的模板是模板字面量的一种更高级的形式，它允许你使用函数解析模板字面量。

简单来说就是让你能以另一种方式进行`function call`。通常我们熟知的`function call`都是使用小括号，并且在小括号中传入参数，但`tagged template literal`可以让我们利用模板字符串进行`function call`。
```js
console.log(123) // 123
console.log`123` // { 0: "123", length: 1, raw: ["123"] }
```

标签函数的第一个参数包含一个字符串数组，其余的参数与表达式相关。你可以用标签函数对这些参数执行任何操作，并返回被操作过的字符串（或者，也可返回完全不同的内容，见下面的示例）。用作标签的函数名没有限制。
```js
const person = "Mike";
const age = 28;

function myTag(strings, personExp, ageExp) {
  const str0 = strings[0]; // "That "
  const str1 = strings[1]; // " is a "
  const str2 = strings[2]; // "."

  const ageStr = ageExp > 99 ? "centenarian" : "youngster";

  // 我们甚至可以返回使用模板字面量构建的字符串
  return `${str0}${personExp}${str1}${ageStr}${str2}`;
}

const output = myTag`That ${person} is a ${age}.`;

console.log(output);
// That Mike is a youngster.
```

标签不必是普通的标识符，你可以使用任何优先级大于 16 的表达式，包括属性访问、函数调用、new 表达式，甚至其他带标签的模板字面量。
```js
console.log`Hello`; // [ 'Hello' ]
console.log.bind(1, 2)`Hello`; // 2 [ 'Hello' ]
new Function("console.log(arguments)")`Hello`; // [Arguments] { '0': [ 'Hello' ] }

function recursive(strings, ...values) {
  console.log(strings, values);
  return recursive;
}
recursive`Hello``World`;
// [ 'Hello' ] []
// [ 'World' ] []
```

标签函数甚至不需要返回字符串！

标签函数接收到的第一个参数是一个字符串数组。对于任何模板字面量，其长度等于替换次数（`${…}` 出现次数）加一，因此总是非空的。对于任何特定的带标签的模板字面量表达式，无论对字面量求值多少次，都将始终使用完全相同的字面量数组调用标签函数。
这允许标签函数以其第一个参数作为标识来缓存结果。为了进一步确保数组值不变，第一个参数及其 `raw` 属性都会被冻结，因此你将无法改变它们。

### `raw`属性

标签函数的第一个参数中存在一个特殊的属性`raw`，可通过它来访问模板字符串的原始字符串，而无需转义特殊字符。
```js
function tag(strings) {
  console.log(strings.raw[0]);
}

tag`string text line 1 \n string text line 2`;
// logs "string text line 1 \n string text line 2" ,
// including the two characters '\' and 'n'
```
使用 `String.raw()`方法创建原始字符串与标签函数中的raw属性结果一致。
```js
String.raw`Hi\n${2+3}!` // "Hi\\n5!"
String.raw({ raw: ['aaa', 'bbbbb', 'cc'] }, 1, 2) // "aaa1bbbbb2cc"
```

如果字面量不包含任何转义序列，`String.raw` 函数就像一个“identity”标签。
这对于许多工具来说很有用，它们要对以特定名称为标签的字面量作特殊处理。
```js
const html = (strings, ...values) => String.raw({ raw: strings }, ...values);
// Some formatters will format this literal's content as HTML
const doc = html`<!doctype html>
  <html lang="en-US">
    <head>
      <title>Hello</title>
    </head>
    <body>
      <h1>Hello world!</h1>
    </body>
  </html>`;
```


## 例子

高亮显示模板字符串内的表达式
```js
function highlight(strings, ...values) {
  let str = '';
  strings.forEach((string, i) => {
    str += string + typeof values[i] === 'string' ? `<mark>${values[i]}</mark>` : '';
  });
  return str;
}
const name = 'Snickers';
const age = '100';
const sentence = highlight`My dog's name is ${name} and he is ${age} years old`;
console.log(sentence); // My dog's name is <mark>Snickers</mark> and he is <mark>100</mark> years old
```

使用String.raw生成正则表达式，而无需使用反斜线
```js
function createNumberRegExp(english) { 
    const PERIOD = english ? String.raw`\.` : ','; // (A) 
    return new RegExp(`[0-9]+(${PERIOD}[0-9]+)?`); 
}
```

Shell command
```js
const proc = sh`ps ax | grep ${pid}`;
```
(Source: [David Herman](https://gist.github.com/dherman/6165867))

Byte strings
```js
const buffer = bytes`455336465457210a`;
```
(Source: [David Herman](https://gist.github.com/dherman/6165867))

HTTP requests
```js
POST`http://foo.org/bar?a=${a}&b=${b} 
    Content-Type: application/json 
    X-Credentials: ${credentials} { "foo": ${foo}, "bar": ${bar}}
    `
    (myOnReadyStateChangeHandler);
```
(Source: [Luke Hoban](https://github.com/lukehoban/es6features#template-strings))

Query language
```js
$`a.${className}[href*='//${domain}/']`
```

## tagged-template-literal相关学习资源

[tagged-template-literals · GitHub Topics · GitHub](https://github.com/topics/tagged-template-literals)

