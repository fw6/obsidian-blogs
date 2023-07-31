---
title: "Compile Svelte in Your Head（3）"
description: ""
pubDate: "2023-07-31 13:46"
heroImage: "https://images.unsplash.com/photo-1593720213428-28a5b9e94613?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D&auto=format&fit=crop&w=1200&q=80"
date created: "2023-07-31 13:46"
date modified: "2023-07-31"
draft: true
tags:
    - writings
    - svelte
---

本文将介绍3个DOM相关的指令：
- `on:` 事件绑定
- `bind:`属性的双向绑定
- `use:`绑定`action`（元素创建时调用的函数）

## 在Svelte模板如何使用

### `on:`event name

使用`on:`指令在DOM元素或Svelte组件上绑定事件

```svelte
<script>
function handleMouseMove(event) {}
function handleClick(event) {}
</script>

<!-- You can pass in as variable -->
<div on:mousemove={handleMouseMove} />

<!-- or you can inline the event handler -->
<div on:mousemove={event => { /*...*/ }} />

<!-- You can modify event handler with modifiers  -->
<div on:click|stopPropagation|once={handleClick}>
```

### `bind:`property

将变量绑定到元素的属性上，更新变量将更新元素的属性。表单元素绑定的值被修改也会导致变量更新。

```svelte
<script>
  let value, name, yes, text, selected;
</script>

<input bind:value />
<input bind:value={name} />

<textarea bind:value={text} />

<input type="checkbox" bind:checked={yes} />

<select bind:value={selected}>
    <option value={a}>a</option>
    <option value={b}>b</option>
    <option value={c}>c</option>
</select>
```

### `use:` actions

`use:`指令用于绑定`action`，`action`提供了另一种方式增强元素功能，当与第三方库一起使用时可使用。
`action`是一个函数，该函数将在绑定DOM元素创建时执行。
该函数返回一个对象，`destroy`在绑定到DOM元素从文档树中被移除时触发；当给action传递参数时，`update`在参数被更新时触发。

```svelte
<script>
/** @type {import('svelte/action').Action}  */
function foo(node) {
    // the node has been mounted in the DOM
    return {
        destroy() {
        // the node has been removed from the DOM
        }
    };
}
</script>

<div use:foo />
```