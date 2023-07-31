---
title: "Proxy & Reflect"
description: ""
pubDate: "2023-07-31 16:20"
heroImage: "https://images.unsplash.com/photo-1688311483778-077a2079e32b?crop=entropy&cs=srgb&fm=jpg&ixid=M3wzNjM5Nzd8MHwxfHJhbmRvbXx8fHx8fHx8fDE2OTA3OTE2MTZ8&ixlib=rb-4.0.3&q=85"
date created: 2023-07-31
date modified: 2023-07-31
draft: true
tags: 
    - writings
    - JavaScript
---

## `Proxy`

:::cite
**Proxy** 对象用于创建一个对象的代理，从而实现基本操作的拦截和自定义（如属性查找、赋值、枚举、函数调用等）。
:::

语法：
```js
const p = new Proxy(target, handler)
```

- `target`是被`Proxy`包装的目标对象（可以是任何类型的对象，包括原生数组、函数，甚至另一个代理）
- `handler` 定义了若干属性的对象，通常属性值为函数，用于定义在对`Proxy`实例`p`执行各种操作时的行为。

其中`Proxy`还有个静态方法——`Proxy.revocable`，用于创建可撤销的`Proxy`对象。

### `handler`处理器对象

`handler`对象是一个容纳若干特定属性的占位符对象，包含了`Proxy`的各种捕获器（trap，另译为陷阱🪤）

所有捕获器都是可选的，不设置则保留源对象的默认行为。

- `handler.getPrototypeOf()`
    - `Object.getPrototypeOf` 方法的捕捉器。
- `handler.setPrototypeOf()`
    - `Object.setPrototypeOf` 方法的捕捉器。
- `handler.isExtensible()`
    - `Object.isExtensible` 方法的捕捉器。
- `handler.preventExtensions()`
    - `Object.preventExtensions` 方法的捕捉器。
- `handler.getOwnPropertyDescriptor()`
    - `Object.getOwnPropertyDescriptor` 方法的捕捉器。
- `handler.defineProperty()`
    - `Object.defineProperty` 方法的捕捉器。
- `handler.has()`
    - `in` 操作符的捕捉器。
- `handler.get()`
    - 属性读取操作的捕捉器。
- `handler.set()`
    - 属性设置操作的捕捉器。
- `handler.deleteProperty()`
    - `delete` 操作符的捕捉器。
- `handler.ownKeys()`
    - `Object.getOwnPropertyNames` 方法和 `Object.getOwnPropertySymbols` 方法的捕捉器。
- `handler.apply()`
    - 函数调用操作的捕捉器。
- `handler.construct()`
    - `new` 操作符的捕捉器。

| Operation                     | Intercepted as                  |
| ----------------------------- | ------------------------------- |
| `proxy[name]`                 | `handler.get(proxy, name)`      |
| `proxy[name] = val`           | `handler.set(proxy, name, val)` |
| `name in proxy`               | `handler.has(name)`             |
| `delete proxy[name]`          | `handler.delete(name)`          |
| `for (let name in proxy) {…}` | `handler.iterate()`             |
| `Object.keys(proxy)`          | `handler.keys()`                |


## `Reflect`

Reflect是一个内置对象，提供了拦截JavaScript操作的方法，这些方法与Proxy相同。

:::warning
`Reflect`不是一个构造函数，所以不能通过`new`操作符对其进行调用，或者将`Reflect`作为函数调用。`Reflect`所有属性和方法都是静态的（类似`Math`）
:::

