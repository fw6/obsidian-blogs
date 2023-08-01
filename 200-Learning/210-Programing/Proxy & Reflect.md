---
title: "Proxy & Reflect"
description: ""
pubDate: "2023-07-31 16:20"
heroImage: "https://images.unsplash.com/photo-1688311483778-077a2079e32b?crop=entropy&cs=srgb&fm=jpg&ixid=M3wzNjM5Nzd8MHwxfHJhbmRvbXx8fHx8fHx8fDE2OTA3OTE2MTZ8&ixlib=rb-4.0.3&q=85"
date created: 2023-07-31
date modified: 2023-07-31
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

`Reflec`t是一个内置对象，提供了拦截JavaScript操作的方法，这些方法与`Proxy`相同。

:::warning
`Reflect`不是一个构造函数，所以不能通过`new`操作符对其进行调用，或者将`Reflect`作为函数调用。`Reflect`所有属性和方法都是静态的（类似`Math`）
:::

单独使用`Reflect`用处不大，与直接执行JavaScript操作相比，有如下用处：
1. 从`Reflect`对象上可以拿到语言内部的方法，如`Object.defineProperty`
2. 操作对象失败时返回`false`
3. 操作对象从命令式变为函数式
4. 操作更易读

```js
// 旧写法
try {
  Object.defineProperty(target, property, attributes);
} catch (err) {}

// 新写法
if (Reflect.defineProperty(target, property, attributes)){} 
else {}

// 老写法  
"assign" in Object; // true  
// 新写法  
Reflect.has(Object, "assign"); //

// 老写法  
Function.prototype.apply.call(Math.floor, undefined, [1.75]) // 1  

// 新写法  
Reflect.apply(Math.floor, undefined, [1.75]) // 1
```


## Reflect在Proxy中的作用

就功能而言，`Reflect.get()`和`Reflect.set()`方法和直接对象赋值没有区别，都是可以互相替代的。

但在实践中，`Proxy`一般搭配`Reflect`使用，原因有以下三点：
1. `Reflect`提供的静态方法和`Proxy`的`handler`参数方法一模一样
2. `Proxy get/set`方法需要的返回值正是`Reflect`的`get/set`方法的返回值。可天然配合使用，比直接对象赋值/取值更方便和准确
3. `receiver`参数具有不可替代性，见下方⬇️

### 关于`receiver`参数

`Proxy` `handler`的`get/set`方法都提供了第三个参数receiver，指代Proxy或继承Proxy的对象（但handler的set方法也可能在原型链上，或以其他方式被间接调用）。

:::info
备注： 假设有一段代码执行 `obj.name = "jen"`， `obj` 不是一个 proxy，且自身不含 `name` 属性，但是它的原型链上有一个 proxy，那么，那个 proxy 的 `set()` 处理器会被调用，而此时，`obj` 会作为 receiver 参数传进来。
:::

举个例子：
```js
const parent = {
    name: 'parent name',
    get value() {
        return this.name;
    },
};

const handler = {
    get(target, key, receiver) {
        return Reflect.get(target, key);
        // 这里相当于 return target[key]
    },
};

const proxy = new Proxy(parent, handler);

const child = {
  name: 'child name',
};

// 设置obj继承与parent的代理对象proxy
Object.setPrototypeOf(child, proxy);

console.log(child.value); // parent name
```
分析以上代码：
1. 获取`child.value`时，`child`本身没有`value`属性
2. 在上一步显式指定了`child`的原型对象，此时应从`proxy`上找`value`属性
3. `proxy`是代理对象，本身通过`handler.get`方法获取`value`属性
4. 在`handler.get`中直接从目标对象中获取`value`属性，此时目标对象是`parent`，返回的是`parent.value`

可看到这与我们的预期不符，当访问`child.value`时，因原型对象上有`value`且根据`parent`上的定义，应返回自身的`name`属性即`child.name`

要解决该问题，只需在Reflect.get时传递receiver：
```js {10}
const parent = {
    name: 'parent name',
    get value() {
        return this.name;
    },
};

const handler = {
    get(target, key, receiver) {
        return Reflect.get(target, key, receiver);
    },
};

const proxy = new Proxy(parent, handler);

const child = {
  name: 'child name',
};

Object.setPrototypeOf(child, proxy);
console.log(child.value); // child name
```

在`Reflect.get`的定义中，receiver表示：
**如果`target`对象指定了`getter`，`receiver`则为调用时的`this`值**

