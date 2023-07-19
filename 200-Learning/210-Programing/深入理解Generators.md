---
title: "深入理解JS-this"
description: ""
pubDate: "2023-07-12 18:36"
heroImage: "https://images.unsplash.com/photo-1688764896020-c448693ec24e?crop=entropy&cs=srgb&fm=jpg&ixid=M3wzNjM5Nzd8MHwxfHJhbmRvbXx8fHx8fHx8fDE2ODkxNTgyMTV8&ixlib=rb-4.0.3&q=85"
date created: "2023-07-12 18:36"
date modified: "2023-07-12"
draft: true
tags:
    - writings
---

>凭意兴做为随作随止，岂是不退之轮？ 从情识解悟者，有悟则有迷，终非常明之灯。 —《菜根谭》

[【转向 Javascript 系列】深入理解 Generators | AlloyTeam](http://www.alloyteam.com/2016/02/generators-in-depth/)
[Regenerator an ES2015 generator compiler to ES5](http://facebook.github.io/regenerator/)

> **`Generator`** 对象由[生成器函数](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function*)返回并且它符合[可迭代协议](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Iteration_protocols#%E5%8F%AF%E8%BF%AD%E4%BB%A3%E5%8D%8F%E8%AE%AE)和[迭代器协议](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Iteration_protocols#%E8%BF%AD%E4%BB%A3%E5%99%A8%E5%8D%8F%E8%AE%AE)。


## 生成器函数

```js
function *range(max, step) {
  var count = 0;
  step = step || 1;

  for (var i = 0; i < max; i += step) {
	count++;
	yield i;
  }

  return count;
}

var gen = range(20, 3), info;

while (!(info = gen.next()).done) {
  console.log(info.value);
}

console.log("steps taken: " + info.value);
```
**`function*`** 会定义一个生成器函数，它返回一个`Generator`对象。

定义一个生成器函数，可通过：
1. `function* 声明`
2. `function* 表达式`
3. 构造函数 `GeneratorFunction`

生成器函数能在执行时暂停，后面又能暂停处继续执行。

调用一个生成器函数并不会立即执行，而是返回这个生成器的迭代器（`iterator`）对象。当第一次执行迭代器对象上的`next`方法时，语句会执行到第一个出现`yield`的位置为止，`yield`紧跟迭代器此次返回的值，会使用`yield*`将执行权限交给另一个生成器（当前生成器暂停执行）。`next`方法返回一个对象，包含两个属性：`{ value: any; done: boolean }`。`value`表示本次`yield`返回的值，`done`表示生成器函数是否已执行完毕并返回。

调用`next`方法时可传值，作为上次 `yield`语法的左值变量。
```js
function *gen(){
    yield 10;
    x=yield 'foo';
    yield x;
}

var gen_obj=gen();
console.log(gen_obj.next());// 执行 yield 10，返回 10
console.log(gen_obj.next());// 执行 yield 'foo'，返回 'foo'
console.log(gen_obj.next(100));// 将 100 赋给上一条 yield 'foo' 的左值，即执行 x=100，返回 100
console.log(gen_obj.next());// 执行完毕，value 为 undefined，done 为 true
```

当在生成器中显示调用`return`时，会导致生成器立即变为完成状态。

`yield`关键字在JS中如何实现呢？

首先，生成器不是线程。支持线程的语言多段语言可以在同一时间运行，这经常会导致资源竞争。生成器则完全不同，JS执行引擎仍然是一个基于时间循环的单线程环境，当生成器运行的时候，它会在叫作`caller`的同一个线程中运行。执行顺序是有序的，永远不会产生并发。不同于系统线程，生成器只在内部使用`yield`时才会挂起。

综上生成器并非由引擎从底层提供的能力，根据[Regenerator](http://facebook.github.io/regenerator/)的实现，可猜测将生成器函数转为普通JS代码，经过转换的代码中需要支持：保存函数上下文信息、实现完善的迭代方法，使得多个`yield`表达式按序执行。

使用`babel/repl`对如下代码进行转译：
```js
function* example() {
  yield 1;
  yield 2;
  yield 3;
}
var iter=example();
iter.next();
```
```js
var _marked = /*#__PURE__*/_regeneratorRuntime().mark(example);
function example() {
  return _regeneratorRuntime().wrap(function example$(_context) {
    while (1) switch (_context.prev = _context.next) {
      case 0:
        _context.next = 2;
        return 1;
      case 2:
        _context.next = 4;
        return 2;
      case 4:
        _context.next = 6;
        return 3;
      case 6:
      case "end":
        return _context.stop();
    }
  }, _marked);
}
var iter = example();
iter.next();
```

`regenrator`将生成器函数转为`switch`语句，在每个case语句使用`_context.next`保存函数当前的上下文状态。
`switch`之外迭代器函数被`_regeneratorRuntime().mark`包装，返回被`_regeneratorRuntime().wrap`包装的迭代器对象。
`Regenerator` 通过工具函数将生成器函数包装，为其添加如 `next/return` 等方法。同时也对返回的生成器对象进行包装，使得对 next 等方法的调用，最终进入由 `switch case` 组成的状态机模型中。除此之外，利用闭包技巧，保存生成器函数上下文信息。


🎬场景1，简易[State Machine](https://stately.ai/docs/xstate-v5/state-machines-and-statecharts)：
```js
function* stateMachine(state) {
    let transition
    while (true) {
        if (transition === "INCREMENT") {
            state++
        }
        if (transition === "DECREMENT") {
            state--
        }
        transition = yield state
    }
}

const iterator = stateMachine(0)

console.log(iterator.next())
console.log(iterator.next("INCREMENT"))
console.log(iterator.next("INCREMENT"))
console.log(iterator.next("DECREMENT"))
```

场景2，生成扑克牌♠️：
```js
const cards = ({
  suits: ["♣️", "♦️", "♥️", "♠️"],
  court: ["J", "Q", "K", "A"],
  [Symbol.iterator]: function* () {
    for (let suit of this.suits) {
      for (let i = 2; i <= 10; i++) yield suit + i;
      for (let c of this.court) yield suit + c;
    }
  }
})
console.log([...cards])
```
```js
function *initializer(count, mapFunc = i => i) {
  for(let i = 0; i < count; i++) {
    if(mapFunc.constructor.name === 'GeneratorFunction') {
      yield* mapFunc(i, count);
    } else {
      yield mapFunc(i, count);
    }
  }
}

const cards = [...initializer(13, function *(i) {
  const p = i + 1;
  yield ['♠️', p];
  yield ['♣️', p];
  yield ['♥️', p];
  yield ['♦️', p];
})];

console.log(cards);
```

## 迭代协议

**迭代协议**并不是新的内置实现或语法，而是_协议_。这些协议可以被任何遵循某些约定的对象来实现。
迭代协议具体分为两个协议：可迭代协议和迭代器协议。

### 可迭代协议

**可迭代协议**允许 JavaScript 对象定义或定制它们的迭代行为，例如，在一个 [`for..of`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/for...of) 结构中，哪些值可以被遍历到。一些内置类型同时是内置的可迭代对象，并且有默认的迭代行为，比如 [`Array`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array) 或者 [`Map`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Map)，而其他内置类型则不是（比如 [`Object`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)）。

要成为**可迭代**对象，该对象必须实现 **`@@iterator`** 方法，这意味着对象（或者它原型链上的某个对象）必须有一个键为 `@@iterator` 的属性，可通过常量 [`Symbol.iterator`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Symbol/iterator) 访问该属性：

`[Symbol.iterator]`

一个无参数的函数，其返回值为一个符合迭代器协议的对象。

当一个对象需要被迭代的时候（比如被置入一个 [`for...of`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/for...of) 循环时），首先，会不带参数调用它的 `@@iterator` 方法，然后使用此方法返回的**迭代器**获得要迭代的值。

值得注意的是调用此无参数函数时，它将作为对可迭代对象的方法进行调用。因此，在函数内部，`this` 关键字可用于访问可迭代对象的属性，以决定在迭代过程中提供什么。

此函数可以是普通函数，也可以是生成器函数，以便在调用时返回迭代器对象。在此生成器函数的内部，可以使用 `yield` 提供每个条目。

### 迭代器协议

**迭代器协议**定义了产生一系列值（无论是有限个还是无限个）的标准方式，当值为有限个时，所有的值都被迭代完毕后，则会返回一个默认返回值。

只有实现了一个拥有以下语义（semantic）的 **`next()`** 方法，一个对象才能成为迭代器：
`next()`：无参数或者接受一个参数的函数，并返回符合 `IteratorResult` 接口的对象（见下文）。如果在使用迭代器内置的语言特征（例如 `for...of`）时，得到一个非对象返回值（例如 `false` 或 `undefined`），将会抛出 [`TypeError`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypeError)（`"iterator.next() returned a non-object value"`）。

所有迭代器协议的方法（`next()`、`return()` 和 `throw()`）都应返回实现 `IteratorResult` 接口的对象。它必须有以下属性：
- `done` 可选
    如果迭代器能够生成序列中的下一个值，则返回 `false` 布尔值。（这等价于没有指定 `done` 这个属性。）
    如果迭代器已将序列迭代完毕，则为 `true`。这种情况下，`value` 是可选的，如果它依然存在，即为迭代结束之后的默认返回值。
- `value` 可选
    迭代器返回的任何 JavaScript 值。`done` 为 `true` 时可省略。
实际上，两者都不是严格要求的；如果返回没有任何属性的对象，则实际上等价于 `{ done: false, value: undefined }`。
如果一个迭代器返回一个 `done: true` 的结果，则对任何 `next()` 的后续调用也返回 `done: true`，尽管这在语言层面不是强制的。

`next` 方法可以接受一个值，该值将提供给方法体。任何内置的语言特征都将不会传递任何值。传递给生成器 `next` 方法的值将成为相应 `yield` 表达式的值。

可选地，迭代器也实现了 **`return(value)`** 和 **`throw(exception)`** 方法，这些方法在调用时告诉迭代器，调用者已经完成迭代，并且可以执行任何必要的清理（例如关闭数据库连接，见[Explicit Resource Management](https://github.com/tc39/proposal-explicit-resource-management)）。

- `return(value)` 可选
    无参数或者接受一个参数的函数，并返回符合 `IteratorResult` 接口的对象，其 `value` 通常等价于传递的 `value`，并且 `done` 等于 `true`。调用这个方法表明迭代器的调用者不打算调用更多的 `next()`，并且可以进行清理工作。
- `throw(exception)` 可选
    无参数或者接受一个参数的函数，并返回符合 `IteratorResult` 接口的对象，通常 `done` 等于 `true`。调用这个方法表明迭代器的调用者监测到错误的状况，并且 `exception` 通常是一个 [`Error`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Error) 实例。


>[!warning]
>无法通过反射的方法确定（例如，没有实际调用 `next()` 并验证返回的结果）一个特定的对象是否实现了迭代器协议。

几乎所有语法和API都期待可迭代对象而非迭代器，见如下例子：
```js
const aGeneratorObject = (function* () {
  yield 1;
  yield 2;
  yield 3;
})();

console.log(typeof aGeneratorObject.next);
// "function"——它有 next 方法（返回正确的值），所以它是迭代器 

console.log(typeof aGeneratorObject[Symbol.iterator]);
// "function"——它有 @@iterator 方法（返回正确的迭代器），所以它是可迭代的

console.log(aGeneratorObject[Symbol.iterator]() === aGeneratorObject);
// true——它的 @@iterator 方法返回自身（一个迭代器），所以它是一个可迭代的迭代器
```

### 异步迭代器和异步可迭代协议

还有一对用于异步迭代的协议，命名为**异步迭代器**和**异步可迭代**协议。它们与可迭代和迭代器协议有着非常类似的接口，只是从调用迭代器方法的每个返回值都包装在一个 promise 中。

当对象实现以下方法时，它会实现异步可迭代协议：
`[Symbol.asyncIterator]`
返回对象的无参数函数，并且符合异步迭代器协议。

当对象实现next、return、throw方法（同上方迭代器协议）但返回值类型为`Promise<IteratorResult>`时，它会实现异步迭代器协议。


### 语言和迭代协议之间的交互

#### 内置的可迭代对象

`String`、`Array`、`TypedArray`、`Map`、`Set`以及 `Intl.Segments` 都是内置的可迭代对象，因为它们的每个 `prototype` 对象都实现了 `@@iterator` 方法。此外，`arguments`对象和一些 DOM 集合类型，如 `NodeList`也是可迭代的。目前，没有内置的异步可迭代对象.

生成器函数返回生成器对象，它们是可迭代的迭代器。异步生成器函数返回异步生成器对象，它们是异步可迭代的迭代器。

从内置迭代返回的迭代器实际上都继承了一个公共类（目前尚未暴露），该类实现了上述 `[Symbol.iterator]() { return this; }` 方法，使它们都是可迭代的迭代器。

```
console.log([][Symbol.iterator]());

Array Iterator {}
  [[Prototype]]: Array Iterator     ==> This is the prototype shared by all array iterators
    next: ƒ next()
    Symbol(Symbol.toStringTag): "Array Iterator"
    [[Prototype]]: Object           ==> This is the prototype shared by all built-in iterators
      Symbol(Symbol.iterator): ƒ [Symbol.iterator]()
      [[Prototype]]: Object         ==> This is Object.prototype
```

#### 接受可迭代对象的内置 API

有许多 API 接受可迭代对象，一些例子包括：
- Map()
- WeakMap()
- Set()
- WeakSet()
- Promise.all()
- Promise.allSettled()
- Promise.race()
- Promise.any()
- Array.from()

```js
const myObj = {};

new WeakSet(
  (function* () {
    yield {};
    yield myObj;
    yield {};
  })()
).has(myObj); // true

Array.from((function* () {
    for (let i=0;i<10;i++) {
        yield i;
    }
})()) // [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

### 期待迭代对象的语法

一些语句和表达式期望可迭代对象，例如 `for...of` 循环、数组和参数扩展、`yield*`和数组解构：
```js
for (const value of ["a", "b", "c"]) {
  console.log(value);
}
// "a"
// "b"
// "c"

console.log([..."abc"]); // ["a", "b", "c"]

function* gen() {
  yield* ["a", "b", "c"];
}

console.log(gen().next()); // { value: "a", done: false }

[a, b, c] = new Set(["a", "b", "c"]);
console.log(a); // "a"
```

当内置语法迭代迭代器，并且最后的结果中 `done` 为 `false`（即迭代器能够生成更多值）但不需要更多值时，如果存在，将调用 `return` 方法。例如，如果在 `for...of` 循环中遇到 `break` 或 `return`，或者数组解构中的标识符只是有限个的，则可能会发生这种情况。
```js
const obj = {
  [Symbol.iterator]() {
    let i = 0;
    return {
      next() {
        i++;
        console.log("Returning", i);
        if (i === 3) return { done: true, value: i };
        return { done: false, value: i };
      },
      return() {
        console.log("Closing");
        return { done: true };
      },
    };
  },
};

const [b] = obj;
// Returning 1
// Closing

const [a, b, c] = obj;
// Returning 1
// Returning 2
// Returning 3
// Already reached the end (the last call returned `done: true`),
// so `return` is not called

for (const b of obj) {
  break;
}
// Returning 1
// Closing
```

异步生成器函数（但不是同步生成器函数中的 `for await...of` 循环和 `yield*` 是与异步迭代交互的唯一方式。在不是同步迭代的异步迭代对象（即它有 `[@@asyncIterator]()` 但没有 `[@@iterator]()`）上使用 `for...of`、数组展开等将抛出 `TypeError：x is not iterable`。

#### 不符合标准的可迭代对象

如果一个可迭代对象的 `@@iterator` 方法不能返回迭代器对象，那么可以认为它是一个_不符合标准的_（Non-well-formed）可迭代对象。

使用这样的可迭代对象很可能会导致如下的运行时异常，或者不可预料的表现：
```js
const nonWellFormedIterable = {};
nonWellFormedIterable[Symbol.iterator] = () => 1;
[...nonWellFormedIterable]; // TypeError: [Symbol.iterator]() returned a non-object value
```

### 一些例子

使用生成器定义一个可迭代对象
```js
function* makeSimpleGenerator(array) {
  let nextIndex = 0;
  while (nextIndex < array.length) {
    yield array[nextIndex++];
  }
}

const gen = makeSimpleGenerator(["yo", "ya"]);

console.log(gen.next().value); // 'yo'
console.log(gen.next().value); // 'ya'
console.log(gen.next().done); // true

function* idMaker() {
  let index = 0;
  while (true) {
    yield index++;
  }
}

const it = idMaker();

console.log(it.next().value); // 0
console.log(it.next().value); // 1
console.log(it.next().value); // 2
// ...
```

使用类定义一个可迭代对象
```js
class SimpleClass {
  #data;

  constructor(data) {
    this.#data = data;
  }

  [Symbol.iterator]() {
    // Use a new index for each iterator. This makes multiple
    // iterations over the iterable safe for non-trivial cases,
    // such as use of break or nested looping over the same iterable.
    let index = 0;

    return {
      // Note: using an arrow function allows `this` to point to the
      // one of `[@@iterator]()` instead of `next()`
      next: () => {
        if (index < this.#data.length) {
          return { value: this.#data[index++], done: false };
        } else {
          return { done: true };
        }
      },
    };
  }
}

const simple = new SimpleClass([1, 2, 3, 4, 5]);

for (const val of simple) {
  console.log(val); // 1 2 3 4 5
}
```


