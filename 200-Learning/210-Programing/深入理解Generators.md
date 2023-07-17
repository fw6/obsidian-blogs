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