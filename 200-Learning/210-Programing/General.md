---
title: "General"
description: ""
pubDate: "2023-06-27 16:23"
heroImage: "https://images.unsplash.com/photo-1521185496955-15097b20c5fe?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D&auto=format&fit=crop&w=640&q=80"
date created: "2023-06-26 09:46"
date modified: "2023-06-27"
---

#Programing

> So much technology, so little talent.
> — <cite>Vernor Vinge</cite>


## Bitmask 位掩码

[javascript - How is svelte making a component dirty - Stack Overflow](https://stackoverflow.com/questions/59541070/how-is-svelte-making-a-component-dirty)

svelte使用$bitmask$技术保存脏检查结果：
```js
component.$$.dirty[(i / 31) | 0] |= (1 << (i % 31));
```
- **(i / 31) | 0  计算位掩码的索引**
- **(1 << (i % 31)) 提供位掩码该位的值**
- **|= 将该位默认位1**
- **-1** **用于表示该组件非脏节点**










