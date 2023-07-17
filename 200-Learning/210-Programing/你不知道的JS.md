---
title: "你不知道的JS"
description: ""
pubDate: "2023-06-27 16:23"
heroImage: "https://images.unsplash.com/photo-1550063873-ab792950096b?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D&auto=format&fit=crop&w=640&q=80"
date created: 2023-07-09
date modified: 2023-07-17
tags: 
	- writings 
	- JS 
	- Programing
---

#Programing #FE

> Go to your bosom: Knock there and ask your heart what it doth know.  
> — <cite>William Shakespeare</cite>

## Diff DOM

[Virtual DOM: Back in Block | Million.js](https://million.dev/blog/virtual-dom)

>1. 静态分析阶段，将树的动态部分提取到 mappings 中;

>2. 通过脏检查比较数据来确定发生了哪些变化。状态变化则通过mappings更新DOM.

具体步骤

- 不使用React渲染jsx，而是使用million.js，用holes 表示动态变化的部分并传递到虚拟DOM，holes作为动态内容的占位符
- 一旦通过脏检查确定状态变化的内容，即可通过mappings找到各自的节点并直接更新DOM  
Block Virtual DOM适合的使用场景：
- 静态内容较多。此时可跳过大量静态部分
- 适用于稳定、变化不大的UI树，

## 在什么情况下 `a === a - 1`

1. 正负`Infinity`。

```js
const a = Infinity;
console.log(a === a - 1); // true

const b = -Infinity;
console.log(b === b - 1);  // true
```

2. 超过`Number.MAX_SAFE_INTEGER`或`Number.MIN_SAFE_INTEGER`的值。

```js
const a  = Number.MAX_SAFE_INTEGER + 4
console.log(a === a - 1); // true
```

扩展：在什么情况下`a == a - 1`

```js
const x = 1
// 将对象转为基本类型值（拆箱转换）
const a = { x, valueOf: () => a.x }
// 每次触发getter都会将x-1
Object.defineProperty(a, 'x', { get() { return --x } })
```

```js
// 每次获取a的时候，让a+1
{
	let count = 0;
	Object.defineProperty(globalThis, 'a', {
	  get() {
	    return ++count;
	  }
	});
}
```

## 如何判断对象某个属性可写？

1. 属性是`accessor property`，并且只有一个`getter`，这个属性不可写

```js
const obj = {
	get a() {
		return 'a'
	}
}
console.log(obj.a) // a
obj.a = 'b'
console.log(obj.a) // a
```

2. 这个属性的`Descriptor`设置了`writable`为`false`，这个属性不可写

```js
const obj = {};

Object.defineProperty(obj, 'a', {
  value: 'a',
  writable: false,
});

console.log(obj.a); // a
obj.a = 'b';
console.log(obj.a); // a
```

3. 目标对象被`Object.freeze`，实际上也是将对象上所有属性的`writable`设置为`false`

```js
const obj = {a: 'a'};
Object.freeze(obj);

console.log(obj.a); // a
obj.a = 'b';
console.log(obj.a); // a
```

```js
function isOwnPropertyWritable(obj, prop) {
  // 判断 null 和 undefined
  if(obj == null) return false;

  // 判断其他原始类型
  const type = typeof obj;
  if(type !== 'object' && type !== 'function') return false;

  // 判断是否被冻结
  if(Object.isFrozen(obj)) return false;

  // 判断sealed的新增属性
  if(!(prop in obj) && Object.isSealed(obj)) return false;

  // 判断属性描述符
  const des = Object.getOwnPropertyDescriptor(obj, prop);
  return des == null || des.writable || !!des.set;
}

function isPropertyWritable(obj, prop) {
  while(obj) {
	if(!isOwnPropertyWritable(obj, prop)) return false;
	obj = Object.getPrototypeOf(obj);
  }
  
  return true;
}
```


## `+0`与`-0`的区别
`JavaScript`的数值`Numbe`使用64位浮点数表示，首位是符号位，然后是52位的整数位和11位的小数位。如果符号位是1，其他各位都是0，那么这个数值会被表示为“`-0`”。所以JavaScript的“0”值有两个，`+0`和`-0`

使用二进制构造出`-0`：
```js
// 首先创建一个8位的ArrayBuffer
const buffer = new ArrayBuffer(8);
// 创建DataView对象操作buffer
const dataView = new DataView(buffer);

// 将第1个字节设置为0x80，即最高位为1
dataView.setUint8(0, 0x80);

// 将buffer内容当做Float64类型返回
console.log(dataView.getFloat64(0)); // -0
```

使用一般运算得出`-0`：
```js
// 使用-Infinity作为分母
console.log(1 / -Infinity); // -0
console.log(-1 / Infinity); // -0
// 负数除法超过最小可表示数
console.log(-Number.MIN_VALUE / 2); // -0
console.log(-1e-1000); // -0
```

一般情况下`-0`等于`0`：
```js
console.log(-0 === 0) // true
```

如何区分`-0`和`0`?
```js
console.log(Object.is(0, -0)) // false
// 作为分母判断是否为-Infinity
console.log(1/-0 === -Infinity) // true
```

其他补充：
1. 所有位运算都会把`-0`转为`0`。因为位运算首先会先转**int32**，而**int32**是没有`-0`的。另外**BigInt**也是没有`-0`的，所有`Object.is(-0n, 0n)`返回`true`
2. `JSON.parse("-0")`返回`-0`，然而`JSON.stringify(-0)`返回“`0`”，所以是不对称的



## 如何优雅的获取数值的整数部分和小数部分？

### 获取整数部分

1. Math.trunc
2. parseInt。缺点：函数名字符串转整数，结果虽正确但不合适；如果第一个参数不是字符串，会先转为字符串，有性能浪费；`parseInt(0.0000001) === 1`
3. 位或“`|`”操作。缺点：位操作的处理中会将操作数转为`int32`，所以它不能处理超过32位的数值，而JavaScript的有效整数范围是53位
4. 原数减去小数部分

```js
const num = 3.75;

// 1
console.log(parseInt(num)); // 3
// 2
console.log(Math.trunc(num)); // 3
// 3
console.log(num | 0); // 3
console.log(~~num); // 3
console.log(num >> 0); // 3
// 4
console.log(num - num % 1) // 3
```

### 获取小数部分

1. 原数值减去整数部分
2. 对`1`取模。缺点：可能精度丢失

```js
// 1
function fract(num) {
  return num - Math.trunc(num);
}
console.log(fract(3.75)); // 0.75
// 2
console.log(3.75 % 1); // 0.75
```

## 关于tc39提案[Explicit Resource Management](https://github.com/tc39/proposal-explicit-resource-management)

该提案已进入stage3，主要用于解决各类资源（内存、I/O等）的生命周期管理常见模式，包括资源的分配和显式释放能力。如：
```js
function * g() {
  const handle = acquireFileHandle(); // critical resource
  try {
    ...
  }
  finally {
    handle.release(); // cleanup
  }
}

const obj = g();
try {
  const r = obj.next();
  ...
}
finally {
  obj.return(); // calls finally blocks in `g`
}
```
生成器函数在离开某个作用域需要调用`return`方法释放，以确保清理掉该迭代器示例。

该提案提供了通用的解决方案：

```js
{
  await using obj = g(); // block-scoped declaration
  const r = await obj.next();
} // calls finally blocks in `g`
```
使用`using`声明语法可在离开作用域前，执行资源退出的相关处理`Symbol.dispose()`被执行（多个则按照相反的顺序执行），如果资源没有可调用的`Symbol.dispose`成员，则在跟踪资源时立即抛出`TypeError`

一些例子：
**WHATWG Streams API**
```js
{
  using reader = stream.getReader();
  const { value, done } = reader.read();
} // 'reader' is disposed
```
**NodeJS FileHandle**
```js
{
  using f1 = await fs.promises.open(s1, constants.O_RDONLY),
        f2 = await fs.promises.open(s2, constants.O_WRONLY);
  const buffer = Buffer.alloc(4092);
  const { bytesRead } = await f1.read(buffer);
  await f2.write(buffer, 0, bytesRead);
} // 'f2' is disposed, then 'f1' is disposed
```

内置的Disposables
1. `IteratorPrototype`
2. `AsyncIteratorPrototype`

如果一个对象符合如下接口，则可称为**disposable**或**async disposable**
```ts
interface Disposable {
  /**
   * Disposes of resources within this object.
   */
  [Symbol.dispose](): void;
}

interface AsyncDisposable {
  /**
   * Disposes of resources within this object.
   */
  [Symbol.asyncDispose](): Promise<void>;
}
```

## 关于`Symbol.toPrimitive`

**`Symbol.toPrimitive`** 是内置的 symbol 属性，其指定了一种接受首选类型并返回对象原始值的表示的方法。它被所有的[强类型转换制](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Data_structures#%E5%BC%BA%E5%88%B6%E7%B1%BB%E5%9E%8B%E8%BD%AC%E6%8D%A2)算法优先调用。

```js
const object1 = {
  [Symbol.toPrimitive](hint) {
    if (hint === 'number') {
      return 42;
    }
    return null;
  }
};

console.log(+object1); // 42
```

在 `Symbol.toPrimitive` 属性（用作函数值）的帮助下，对象可以转换为一个原始值。该函数被调用时，会被传递一个字符串参数 `hint`，表示要转换到的原始值的预期类型。`hint` 参数的取值是 `"number"`、`"string"` 和 `"default"` 中的任意一个。

`"number"` hint 用于[强制数字类型转换](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Data_structures#%E5%BC%BA%E5%88%B6%E6%95%B0%E5%AD%97%E7%B1%BB%E5%9E%8B%E8%BD%AC%E6%8D%A2)算法。`"string"` hint 用于[强制字符串类型转换](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String#%E5%AD%97%E7%AC%A6%E4%B8%B2%E5%BC%BA%E5%88%B6%E8%BD%AC%E6%8D%A2)算法。`"default"` hint 用于[强制原始值转换](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Data_structures#%E5%BC%BA%E5%88%B6%E5%8E%9F%E5%A7%8B%E5%80%BC%E8%BD%AC%E6%8D%A2)算法。`hint` 仅是作为首选项的偏弱的信号提示，实现时，可以自由忽略它（就像 [`Symbol.prototype[@@toPrimitive]()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Symbol/@@toPrimitive) 一样）。该语言不会在 `hint` 和结果类型之间强制校正，尽管 `[@@toPrimitive]()` 必须返回一个原始值，否则将抛出 [`TypeError`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypeError)。

没有 `@@toPrimitive` 属性的对象通过以不同的顺序调用 `valueOf()` 和 `toString()` 方法将其转换为原始值，这在**强制类型转换**部分进行了更详细的解释。`@@toPrimitive` 允许完全控制原始转换过程。例如，[`Date.prototype[@@toPrimitive]`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Date/@@toPrimitive) 将 `"default"` 视为 `"string"` 并且调用 `toString()` 而不是 `valueOf()`。[`Symbol.prototype[@@toPrimitive]`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Symbol/@@toPrimitive) 忽略 hint，并总是返回一个 symbol，这意味着即使在字符串上下文中，也不会调用 [`Symbol.prototype.toString()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Symbol/toString)，并且 `Symbol` 对象必须始终通过 `String()` 显式转换为字符串。

## 强制类型转换

强制类型转换用于得到一个期望的原始值。如果值已经时原始值，则不会进行任何转换。对象将按照以下顺序调用它的如下方法：
1. **\[\[@@toPrimitive\]\]**
2. valueOf
3. toString
通过如上方法转为原始值。

**\[\[@@toPrimitive\]\]** 方法如果存在，则必须返回原始值，返回对象则导致`TypeError`。对于`valueOf`和`toString`，如果其中一个返回对象，则忽略其返回值，从而使用另一个的返回值；如果两者都不存在，或两者都没返回原始值，则抛出`TypeError`。
```js
console.log({} + []); // "[object Object]"
```
`{}` 和 `[]` 都没有 `[@@toPrimitive]()` 方法。`{}` 和 `[]` 都从 [`Object.prototype.valueOf`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/valueOf) 继承 `valueOf()`，其返回对象自身。因为返回值是一个对象，因此它被忽略。因此，调用 `toString()` 方法。[`{}.toString()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/toString) 返回 `"[object Object]"`，而 [`[].toString()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/toString) 返回 `""`，因此这个结果为：`"[object Object]"`。

## JavaScript的并发模型与事件循环

JavaScript 有一个基于**事件循环**的并发模型，事件循环负责执行代码、收集和处理事件以及执行队列中的子任务。

如下图展示了现代JavaScript引擎在运行时的可视化描述
![image.png](https://raw.githubusercontent.com/fw6/assets/main/toy_docs/20230717162354.png)

栈：函数调用形成了一个由若干帧组成的栈，帧中包含了函数的参数和局部变量（执行上下文）。当函数执行完毕所属栈被弹出⏏️
堆：对象被分配在堆（一大块非结构化的内存区域）中。
队列：一个JavaScript运行时包含了一个待处理的消息队列。每个消息都关联着一个用以处理这个消息的回调函数。

事件循环的常常以如下方式实现：
```js
while (queue.waitForMessage()) {
  queue.processNextMessage();
}
```


### EventLoop
[WHATWG/task-queue](https://html.spec.whatwg.org/multipage/webappapis.html#task-queue)

事件循环有一个或多个任务队列，任务队列是一组任务。
>实际上任务队列是集合而非队列，因为事件循环处理模型是从所选队列中获取第一个可执行的任务，而不是使第一个任务出队。

>微任务队列不是任务队列！

每个事件循环都有一个微任务队列。微任务是指通过微任务算法队列创建的任务的通俗说法。
每个事件循环都有一个执行微任务检查的布尔值，用于防治执行微任务检查点的重复执行。

实现循环执行步骤如下：
1. 使oldestTask 和 taskStartTime 为 null
2. 如果事件循环有任务队列，且至少有一个可以执行，则：
	1. 定义一个taskQueue
	2. 定义taskStartTime为当前共享时间
	3. oldestTask设置为taskQueue第一个可执行任务，并从taskQueue移除
	4. 将事件循环的当前执行中任务设置为oldestTask
	5. 执行oldestTask若干步骤
	6. 将事件循环当前执行中任务设置为null
3. 执行微任务检查点
4. 

