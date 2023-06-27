---
title: "General"
description: ""
pubDate: "星期二 06 2023"
heroImage: "![photo by Marina T on Unsplash](https://images.unsplash.com/photo-1571678348855-32ee2bfcea89?crop=entropy&cs=srgb&fm=jpg&ixid=M3wzNjM5Nzd8MHwxfHJhbmRvbXx8fHx8fHx8fDE2ODc4NDQ3MzF8&ixlib=rb-4.0.3&q=85&w=1200&h=400)"
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










