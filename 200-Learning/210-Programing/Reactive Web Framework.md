---
title: "Reactive Web Framework"
description: ""
pubDate: "2023-08-01 10:39"
heroImage: "https://raw.githubusercontent.com/fw6/assets/main/toy_docs/20230801105044.png"
date created: "2023-08-01 10:39"
date modified: "2023-08-01"
draft: true
tags:
    - writings
    - JavaScript
    - FE
    - Programming
---

Web端的响应式（`Reactive`）泛指应用状态改变，进而自动触发UI更新。对广大使用响应式框架的开发者而言，只需管理应用状态，而无需关心如何将状态映射到UI层以及何时触发UI更新。

各个框架实现响应式的方式各不相同，这也对代码的性能和懒加载产生影响。本文将深入探讨响应式相关的内容。

## 何为`Reactivity`

现代web框架的核心是响应性。响应性是在应用状态发生更改时更新视图的能力。

下面是使用纯JavaScript写的计数器实例应用：
```js
const root = document.getElementById('app');
root.innerHTML = `
  <button>-</button>
  <span>0</span>
  <button>+</button>
`;

const [decrementBtn, incrementBtn] = root.querySelectorAll('button');
const span = root.querySelector('span');
let count = 0;
decrementBtn.addEventListener('click', () => {
    count--;
    span.innerText = count;
});
incrementBtn.addEventListener('click', () => {
    count++;
    span.innerText = count;
});
```

在Vue中，等价于：
```vue
<div>
    <button @click="counter -= 1">-</button>
    <span>{{ counter }}</span>
    <button @click="counter += 1">+</button>
</div>

<script>
export default {
    data() {
        return {
            counter: 0,
        };
    },
};
</script>
```

在React中，等价于：
```jsx
function App() {
    const [counter, setCounter] = React.useState(0);
    return (
        <>
            <button onClick={() => setCounter(counter => counter - 1)}>-</button>
            <span>{counter}</span>
            <button onClick={() => setCounter(counter => counter + 1)}>+</button>
        </>
    );
}
```

:::tldr
使用Web框架时，您的代码更多关注的是如何根据业务需求更新应用程序状态，并使用模板语法或JSX描述视图外观。框架将连接视图与应用状态，并在状态发生变化时在合适的时机更新视图
:::

与直接使用Web API操作DOM并更新状态相比，使用框架将帮助开发者忽略视图与应用状态同步的问题，也会减少视图与应用程序之间难以预料的错误🙅。

所以接下来的主题：Web框架如何实现响应性的呢？

## WHEN & WHAT

为实现响应式，框架需清楚如下两个问题：
1. 应用程序状态何时被改变？区分何时需要执行更新视图的任务
2. 应用程序状态发生了什么变化？用于判断视图更新的范围

### WHEN

`WHEN`通知框架应用程序状态已更改，以便框架知道它需要开始更新视图

不同框架使用不同策略来检测应用程序状态何时发生变化，但本质上它通常归结为调用框架的`schedule_update`。`schedule_update`通常为框架的去抖`update`函数，因为应用程序状态的变化会导致派生的状态变化，或者框架使用者会连续更改应用程序状态的不同部分。如果框架在每次状态更改时更新视图，则可能会过于频繁的更新视图导致效率低下，或者视图不一致。

例如下面的Ract示例：
```jsx {6-7}
function Todos () {
    const [todos, setTodos] = useState([]);
    const [totalTodos, setTotalTodos] = useState(0);

    const onAddTodo = todo => {
        setTodos(todos => [...todos, todo]);
        setTotalTodos(totalTodos => totalTodos + 1);
    };
    // ...
}
```

如果框架同步更新`todos`和`totalTodos`，则可能在一瞬间导致视图上的不一致（尽管看上去不太可能发生）
:::warning
不应该以此种方式设置`totalTodos`，而应该从`todos.length`中派生！
 ["Don't Sync State. Derive it!" by Kent C. Dodds.](https://kentcdodds.com/blog/dont-sync-state-derive-it)
:::

那么该如何知道应用程序何时发生了变化呢？

#### Mutation Tracking

正如其名，我们可以追踪状态突变。突变只能作用于对象，因为你无法改变`primitive`类型数据。

JavaScript的基本数据类型如数字、字符串、布尔值等按值传递到函数中，因此如果将基础值`reassignment`则无从观察到该变化:
```js
let data = 1;
render(data);
// changes to the data will not be propagated into the render function
data = 2;

function render (data) {
    // data is a value
    // however it is changed in the outside world
    // got nothing to do with me
    setInterval(() => {
        console.log(data); // will always console out `1`
    }, 1000);
}
```

另一方面，对象是引用传递。因此可以在内部观察到同一对象的任何更改：
```js
let data = { foo: 1 };
render(data);
// mutate data some time later
setTimeout(() => {
    data.foo = 2;
}, 1000);

function render (data) {
    // data is referenced to the same object
    // changes to data.foo can be observed here
    setInterval(() => {
        console.log(data.foo); // initially `1`, after mutation, its `2`
    }, 1000);
}
```

这也是为什么大多数框架的应用状态通过`this`访问——`this`是一个对象，框架可以观察/跟踪`this.appState`的更改。

现在来看看突变追踪是如何实现的。

在JavaScript对象中有两种常见的对象类型：普通对象和数组。
随之ES6 `Proxy`的引入，突变跟踪变得更加简单，现在，先让我们不使用`Proxy`来实现以下

##### Prior Proxy
要在没有`Proxy`的情况下追踪变化，可以为对象的所有属性自定义`getter`和`setter`。每当框架用户更改属性值时，都会调用自定义`setter`，就会知道哪些内容发生了变化：
```js
function getTrackableObject (obj) {
    if (obj[Symbol.for('isTracked')]) return obj;
    const tracked = Array.isArray(obj) ? [] : {};
    for (const key in obj) {
        Object.defineProperty(tracked, key, {
            configurable: true,
            enumerable: true,
            get () {
                return obj[key];
            },
            set (value) {
                if (typeof value === 'object') {
                    value = getTrackableObject(value);
                }
                obj[key] = value;
                console.log(`'${key}' has changed.`);
            },
        });
    }
    // marked as 'tracked'
    Object.defineProperty(tracked, Symbol.for('isTracked'), {
        configurable: false,
        enumerable: false,
        value: true,
    });
    return tracked;
}

// track app state
const appState = getTrackableObject({ foo: 1 });
appState.foo = 3; // log `'foo' has changed.`
```
Inspired by [Vue.js 2.0's observer](https://github.com/vuejs/vue/blob/22790b250cd5239a8379b4ec8cc3a9b570dac4bc/src/core/observer/index.js#L110)

但上面的代码只能在对象上已有的属性上定义getter和setter，可能会错过通过在对象上增删属性的更改。

如果没有更好的JavaScript API就无法解决该问题，一种解决方案时提供辅助函数，例如在Vue中使用`Vue.set(obj, 'newKey', value)`来替代`obj.newKey = value`

跟踪数组的`mutation`同样有问题，除了可以通过赋值改变数组项之外，还可以通过一些`mutating methods`例如：splice、reverse、sort、push、pop、unshift、shift。

要追踪这些方法导致的变化，需要`patch`以下：
```js
const TrackableArrayProto = Object.create(Array.prototype);
for (const method of [
    'push',
    'pop',
    'splice',
    'unshift',
    'shift',
    'sort',
    'reverse',
]) {
    const original = Array.prototype[method];
    TrackableArrayProto[method] = function () {
        const result = original.apply(this, arguments);
        console.log(`'${method}' was called`);
        if (method === 'push' || method === 'unshift' || method === 'splice') {
            // TODO track newly added item too!
        }
        return result;
    };
}
function getTrackableArray (arr) {
    const trackedArray = getTrackableObject(arr);
    // set the prototype to the patched prototype
    trackedArray.__proto__ = TrackableArrayProto;
    return trackedArray;
}

// track app state
const appState = getTrackableArray([1, 2, 3]);
appState.push(4); // log `'push' was called.`
appState[0] = 'foo'; // log `'0' has changed.
```
Inspired by [Vue.js 2.0's observer/array](https://github.com/vuejs/vue/blob/22790b250cd5239a8379b4ec8cc3a9b570dac4bc/src/core/observer/array.js)

总结下，在没有Proxy的情况下跟踪数组和对象的突变，需要为所有属性自定义getter/setter，以便能够捕获何时修改了属性。此外，还要修补数组中会导致原数组产生突变的方法。

然而，还是有无法覆盖的操作，如新增或删除属性。

##### With Proxy

Proxy允许我们定义目标对象上基本操作的自定义行为，如下：
```js
function getTrackableObject (obj) {
    for (const key in obj) {
        if (typeof obj[key] === 'object') {
            obj[key] = getTrackableObject(obj[key]);
        }
    }
    return new Proxy(obj, {
        set: function (target, key, value) {
            console.log(`'${key}' has changed`);
            if (typeof value === 'object') {
                value = getTrackableObject(value);
            }
            return (target[key] = value);
        },
        deleteProperty: function (target, key) {
            console.log(`'${key}' was deleted`);
            return delete target[key];
        },
    });
}

const appState = getTrackableObject({ foo: 1, bar: [2, 3] });
appState.foo = 3; // log `'foo' has changed.`
appState.bar.push(4); // log `'2' has changed.`, `'length' has changed`
appState.bar[0] = 'foo'; // log `'0' has changed.
```

#### 如何使用`Mutation Tracking`？

通过上一步我们已经初步拿到了何时应用状态被变更，那么我们可以在组件初始化期间设置`tracking`:
- 跟踪组件属性变更
- 跟踪组件实例变更
- 或介于二者之间的东西

```js
// track a property of the component
class Component {
    constructor(initialState) {
        this.state = getTrackableObject(initialState);
    }
}
class UserComponent extends Component {
    constructor() {
        super({ foo: 1 });
    }
    someHandler () {
        this.state.foo = 2; // Log `'foo' has changed`
        this.other.foo = 2; // Does not track this
    }
}

// track the component instance itself
class Component {
    constructor() {
        return getTrackableObject(this);
    }
}

class UserComponent extends Component {
    constructor() {
        super();
    }
    someHandler () {
        this.foo = 1; // Log `'foo' has changed`
    }
}
```

一旦能跟踪到状态的变更，接下来要做的就是调用`schedule_update`。


#### 不追踪突变的方式

受限于实现突变追踪的复杂性，与Proxy的兼容性，并非所有框架都使用该方案。
某些框架不会在更新应用程序状态时要求调用`schedule_update`，而是强制使用其提供的API来更改应用程序状态：
```js
// instead of
this.appState.one = '1';
scheduleUpdate();

// you have to use the frameworks API
this.setAppState({ one: '1' });
```

这位框架作者提供了更简单的设计和处理更少的边缘情况：
```js
class Component {
    setAppState (appState) {
        this.appState = appState;
        scheduleUpdate();
    }
}
```
Inspired by [React's setState](https://github.com/facebook/react/blob/0cf22a56a18790ef34c71bef14f64695c0498619/packages/react/src/ReactBaseClasses.js#L57)

然而这很容易对新人产生困惑😖：
```js
class MyComponent extends Component {
    someHandler () {
        // 如果依照直接直接使用该方式更新state，将不触发视图更新
        this.appState.one = '1';
    }
}
```
更新数组也看上去略显笨拙：
```js
class MyComponent extends Component {
    someHandler () {
        // this will not schedule update
        this.appState.list.push('one');
        // you need to call setAppState after the .push()
        this.setAppState({ list: this.appState.list });

        // or instead, for a one-liner
        this.setAppState({ list: [...this.appState.list, 'one'] });
    }
}
```

另一种两全其美的方案是在您认为可能产生更改的地方插入`schedule_update`，如：
- 事件处理程序event handler
- Timeout（如：setTimeout、setInterval）
- API handling，promise handling
- ...

因此框架用户不应强制用户使用`setAppState`，而应使用自定义timeout、api handlers等：
```js
// framework core code
function timeout (fn, delay) {
    setTimeout(() => {
        fn();
        scheduleUpdate();
    }, delay);
}

// user code
import { $timeout } from 'my-custom-framework';

class UserComponent extends Component {
    someHandler () {
        // will schedule update after the callback fires.
        $timeout(() => {
            this.appState.one = '1';
        }, 1000);

        setTimeout(() => {
            // this will not schedule update
            this.appState.two = '2';
        }, 1000);
    }
}
```
Inspired by [AngularJS's $timeout](https://github.com/angular/angular.js/blob/master/src/ng/timeout.js#L42)

现在框架用户可以按照自己想要的方式自由更改应用程序状态，只要在自定义处理函数内即可。因为在自定义处理程序之后，框架会调用`schedule_update`

同样，这种方式也会让开发者产生困惑😖，产生了新的心智负担。（Try search ["AngularJS $timeout vs window.setTimeout"](https://www.google.com/search?q=angularjs%20$timeout%20vs%20window.setTimeout)）

