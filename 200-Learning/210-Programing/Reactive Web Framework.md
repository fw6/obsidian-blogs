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
1. 应用程序状态何时被改变？
2. 应用程序状态发生了什么变化？

