---
title: "你不知道的JS"
description: ""
pubDate: "2023-06-27 16:23"
heroImage: "https://images.unsplash.com/photo-1550063873-ab792950096b?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D&auto=format&fit=crop&w=640&q=80"
date created: 2023-07-09
date modified: 2023-07-18
tags:
    - notes
    - JS
    - Programming
---

#Programming #FE

> Go to your bosom: Knock there and ask your heart what it doth know.  
> — <cite>William Shakespeare</cite>

## TOC

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

`JavaScript` 有一个基于**事件循环**的并发模型，事件循环负责执行代码、收集和处理事件以及执行队列中的子任务。

如下图展示了现代`JavaScript`引擎在运行时的可视化描述  
![image.png](https://raw.githubusercontent.com/fw6/assets/main/toy_docs/20230717162354.png)

栈：函数调用形成了一个由若干帧组成的栈，帧中包含了函数的参数和局部变量（执行上下文）。当函数执行完毕所属栈被弹出⏏️  
堆：对象被分配在堆（一大块非结构化的内存区域）中。  
队列：一个`JavaScript`运行时包含了一个待处理的消息队列。每个消息都关联着一个用以处理这个消息的回调函数。

事件循环的常常以如下方式实现：

```js
while (queue.waitForMessage()) {
  queue.processNextMessage();
}
```

可以看到整个事件循环就是反复“等待-执行”。

EventLoop的定义 -> [WHATWG/task-queue](https://html.spec.whatwg.org/multipage/webappapis.html#task-queue)

事件循环有一个或多个任务队列，任务队列是一组任务。

>实际上任务队列是集合而非队列，因为事件循环处理模型是从所选队列中获取第一个可执行的任务，而不是使第一个任务出队。

>微任务队列不是任务队列！

每个事件循环都有一个微任务队列。微任务是指通过微任务算法队列创建的任务的通俗说法。  
每个事件循环都有一个执行微任务检查的布尔值，用于防治执行微任务检查点的重复执行。

当拿到一段`JavaScript`代码时，浏览器首先要做的是传递给`JavaScript`引擎，并要求它执行。执行`JavaScript`代码并非一次性，当宿主环境（浏览器、`Node`、`Deno`、小程序容器）遇到一些事时，会继续传递一段代码让`JavaScript`引擎执行。此外，我们还会提供API给`JavaScrip`t引擎，比如`setTimeout`（由宿主环境实现！）这种，它会允许JavaScript在特定时机执行。

在ES3和更早版本，`JavaScript`本身没有异步能力，这就意味着传递给`JavaScript`引擎一段代码，引擎直接顺序执行了，这个任务也就是宿主发起的任务。

在ES5之后，`JavaScript`引入了`Promise`，这样无序浏览器安排，`JavaScript`引擎本身就能发起任务了。

`JSC`引擎对任务的定义：宿主发起的称为宏观任务，JavaScript引擎发起的称为微观任务。  
`JavaScript`引擎等待宿主环境分配宏观任务，在操作系统中，通常等待的行为就是一个事件循环，所以在Node术语中，会把这部分称为事件循环。

宏观任务队列就相当于事件循环。

在宏观任务中，`JavaScript`还会产生异步代码，`JavaScrip`t必须保证这些异步代码在一个宏观任务中执行完成，因此每个宏观任务中又包含了一个微观任务队列。

有了宏观任务与微观任务机制，就可以实现`JavaScript`引擎级和宿主级的任务了。例如：`Promise`永远在队列尾部添加微观任务，`setTimeout`、``requestIdleCallback``等宿主API则会添加宏任务。

```js
var r = new Promise(function(resolve, reject){
	console.log("a");
	resolve()
});
setTimeout(()=>console.log("d"), 0)
r.then(() => console.log("c"));
console.log("b")
```

执行这段代码，可看到执行顺序为`a`->`b`->`c`->`d`。因为`Promise`产生的时微任务，在第一次宏任务执行`a`、`b`，`Promise`创建的微任务被执行，即打印了`c`，然后定时器执行触发内部新的宏任务，打印`d`。

微任务始终先于宏任务！

```js
// 下面这段代码进入执行栈时延时就已经开始了
setTimeout(()=>console.log("d"), 0)

var r = new Promise(function(resolve, reject){
	resolve()
});

r.then(() => {
	var begin = Date.now();
	while(Date.now() - begin < 1000);
	console.log("c1")
	new Promise(function(resolve, reject){
		resolve()
	}).then(() => console.log("c2"))
});
```

上方代码中在微任务中阻塞执行1s后创建了新的微任务，最终结果依旧是：`c1`->`c2`->`d`

## 执行上下文、闭包、作用域链、this值

`JavaScript`标准把一段代码（包括函数），执行所需的所有信息定义为：“执行上下文”。

执行上下文在ES5中包含了如下部分：

- `lexical environment`。词法环境，当获取变量时使用
- `variable environment`。变量环境，当声明变量时使用
- `this value`。`this`值

在ES2018中，执行上下文又变成了如下内容：

- `lexical environment`。词法环境，当获取变量和`this`值时使用
- `variable environment`。变量环境，当申明变量时使用
- `code evaluation state`。用于恢复代码执行位置
- `Function`。执行的任务是函数时使用，表示正在被执行的函数
- `ScriptOrModule`。执行的任务是脚本或模块时使用，表示正在被执行的代码
- `Realm`。使用的基础库和内置对象实例
- `Generator`。仅生成器上下文有此属性，表示当前生成器。

### this关键字的行为

>`this`是运行时，作用域是定义时

`this`是执行上下文中很重要的一个组成部分。同一个函数调用方式不同，得到的`this`值也不同，如下所示：

```js
function showThis(){
    console.log(this);
}

var o = {
    showThis: showThis
}

showThis(); // global
o.showThis(); // o
```

在上方示例中定义了函数`showThis`，把它赋值给一个对象o的属性，分别使用两个引用来调用同一个函数，结果得到了不同的`this`值。

普通函数的this值由”调用它所使用的引用“决定，我们获取函数的表达式，它实际上返回的并非函数本身，而是一个`Reference`类型。

`Reference`类型包含两部分：对象和属性值。`o.showThis`产生的`Reference`类型，即由对象o和属性“showThis”构成。

当做一些运算时，`Reference`类型会被解引用，即获取真正的值来参与运算，而类似函数调用、delete操作等，都需要使用到`Reference`类型中的对象。  
在上方例子中，`Reference`类型中的对象被当作`this`值，传入了执行函数的上下文中。

一言以蔽之：调用函数时，决定了函数运行时刻的`this`值。  
实际上从运行时的角度来看，`this`跟面向对象毫无关联，它是与函数调用时使用的表达式相关。

再来看“方法”，它的表现又不一样：

```js
class C {
    showThis() {
        console.log(this);
    }
}
var o = new C();
var showThis = o.showThis;

showThis(); // undefined
o.showThis(); // o
```

当使用`showThis`这个引用去调用方法时，得到了`undefined`

### `this`关键字的机制

函数能够引用定义时的变量，如上文⬆️，函数也能记住定义时的this，因此函数内部必定有一个机制来保存这些信息。

在JavaScript标准中，为函数规定了用来保存定义时上下文信息的私有属性`[[Environment]]`。

当一个函数被执行时，会创建一个新的执行环境记录，记录的外层词法环境（otuer lexical environment）会被设置成函数的`[[Environment]]`，这个动作就是切换上下文。

```js
var a = 1;
foo();

// 在别处定义了foo：

var b = 2;
function foo(){
    console.log(b); // 2
    console.log(a); // error
}
```

这里的foo能访问定义时的b却不能访问执行时的a，这就是执行上下文的切换机制。

JavaScript用一个栈来管理执行上下文，每个栈中的每一项又包含一个链表。如下所示：  
![image.png](https://raw.githubusercontent.com/fw6/assets/main/toy_docs/20230718155325.png)  
当函数调用时，会入栈一个新的执行上下文，函数调用时执行上下文出栈。

而this是个更复杂的机制，JavaScript标准定义了`[[thisMode]]`私有属性。  
`[[thisMode]]`包含3个取值：

- lexical：表示从上下文中找到this，这对应了箭头函数
- global。表示this为undefined时，取全局对象，对应了普通函数
- strict。当严格模式时使用，this严格按照调用时传入的值，可能为null或undefined

方法的行为和普通函数有差异，恰恰是因为class设计成了默认按照strict模式执行。

函数创建新的执行上下文中的词法环境记录时，会根据`[[thisMode]]`来标记新记录的`[[ThisBindingStatus]]`私有属性。  
代码执行到this时，会逐层检查当前词法环境记录中的`[[ThisBindingStatus]]`，当找到有this的环境记录时获取this的值。

## var、let、const

### var

*`var` 声明了作用于函数执行的作用域*。所以在同一个函数内，`for`、`if`语句块内的`var`申明在外部也能获取。

为解决该问题诞生了一个技巧——立即执行函数表达式（`IIFE`），通过创建一个函数并立即执行来构造一个新的作用域以此来控制var的范围。

```js
void (function() {
	var a
	// ...
})()
```

### `let`

`let`语句声明一个块级作用域的局部变量。

let是ES6之后引入的新的变量声明模式，为实现`let`，`JavaScrip`t在运行时引入了块级作用域。也就是说在let之前，`if`、`for`语句皆不产生作用域。

`var`和`let`的一个重要区别，let声明的变量不会在作用域中被提升，它是在编译时才初始化。  
`let`和`const`一样，不会在全局声明中创建`window`对象的属性。  
与`let`不同的是，`let`只是开始声明而非完整表达式，如下所示：

```js
if (true) let a = 1 // SyntaxError: Lexical declaration cannot appear in a single-statement context
```

`let`不允许重复声明（在同一个函数或块作用域），否则会抛出`SyntaxError`

### `const`

常量是块级范围的，非常类似用 [let](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/let) 语句定义的变量。但常量的值是无法（通过重新赋值）改变的，也不能被重新声明。

**`const` 声明**创建一个值的只读引用。但这并不意味着它所持有的值是不可变的，只是变量标识符不能重新分配。例如，在引用内容是对象的情况下，这意味着可以改变对象的内容（例如，其参数）。

一个常量不能和它所在作用域内的其他变量或函数拥有相同的名称。  
常量要求一个初始值

### 暂时性死区

从一个代码块的开始直到代码执行到声明变量的行之前，`let` 或 `const` 声明的变量都处于“暂时性死区”（Temporal dead zone，TDZ）中。

当变量处于暂时性死区之中时，其尚未被初始化，尝试访问变量将抛出 [`ReferenceError`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/ReferenceError)。当代码执行到声明变量所在的行时，变量被初始化为一个值。如果声明中未指定初始值，则变量将被初始化为 `undefined`。

与 [`var`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/var) 声明的变量不同，如果在声明前访问了变量，变量将会返回 `undefined`。以下代码演示了在使用 `let` 和 `var` 声明变量的行之前访问变量的不同结果。

```js
{ // TDZ starts at beginning of scope
  console.log(bar); // undefined
  console.log(foo); // ReferenceError
  var bar = 1;
  let foo = 2; // End of TDZ (for foo)
}

```

使用术语“temporal”是因为区域取决于执行顺序（时间），而不是编写代码的顺序（位置）。例如，下面的代码会生效，是因为即使使用 `let` 变量的函数出现在变量声明之前，但函数的执行是在暂时性死区的外面。

```js
{
  // TDZ starts at beginning of scope
  const func = () => console.log(letVar); // OK

  // Within the TDZ letVar access throws `ReferenceError`

  let letVar = 3; // End of TDZ (for letVar)
  func(); // Called outside TDZ!
}
```

以下代码会造成暂时性死区：

```js
function test() {
  var foo = 33;
  if(foo) {
    let foo = (foo + 55); // ReferenceError
  }
}
test();
```

由于外部变量 `foo` 有值，因此会执行 `if` 语句块，但是由于词法作用域，该值在块内不可用：`if` 块内的标识符 `foo` 是 `let foo`。表达式 `(foo + 55)` 会抛出 `ReferenceError` 异常，是因为 `let foo` 还没完成初始化，它仍然在暂时性死区里。

### 变量提升

变量提升（Hoisting）被认为是，Javascript 中执行上下文（特别是创建和执行阶段）工作方式的一种认识。
从概念的字面意义上说，“变量提升”意味着变量和函数的声明会在物理层面移动到代码的最前面，但这么说并不准确。实际上变量和函数声明在代码里的位置是不会动的，而是在编译阶段被放入内存中。

JavaScript 在执行任何代码段之前，将函数声明放入内存中的优点之一是，你可以在声明一个函数之前使用该函数。

	函数和变量相比，会被优先提升。这意味着函数会被提升到更靠前的位置。

即使我们在定义函数之前调用它，函数仍然可以工作。这是因为在 JavaScript 中**执行上下文**的工作方式造成的。
变量提升也适用于其他数据类型和变量。变量可以在声明之前进行初始化和使用。但是如果没有初始化，就不能使用它们。

JavaScript 只会提升声明，不会提升其初始化。如果一个变量先被使用再被声明和赋值的话，使用时的值是 undefined。

```js
console.log(num); // Returns undefined
var num;
num = 6;
```

如果你先赋值、再使用、最后声明该变量，使用时能获取到所赋的值

```js
num = 6;
console.log(num); // returns 6
var num;
```

