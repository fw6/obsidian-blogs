---
title: "JS"
description: ""
pubDate: "2023-06-27 16:23"
heroImage: "https://images.unsplash.com/photo-1682687979601-082a1295b78f?crop=entropy&cs=srgb&fm=jpg&ixid=M3wzNjM5Nzd8MHwxfHJhbmRvbXx8fHx8fHx8fDE2ODc4NDQ4MjR8&ixlib=rb-4.0.3&q=85&w=1200&h=400"
date created: "2023-06-26 11:00"
date modified: "2023-06-27"
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
