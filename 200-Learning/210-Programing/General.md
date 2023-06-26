#Programing 

>语言通用的编程知识

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










