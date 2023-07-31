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

### `on:`

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

### `bind:`

```svelte
<script>
  let name, yes;
</script>

<!-- You can bind `name` to input.value -->
<!-- Changing `name` will update input.value to be the value of `name` and -->
<!-- changing input.value will update `name` to be input.value -->
<input bind:value={name} />
<!-- You can bind input.checked for a checkbox input -->
<input type="checkbox" bind:checked={yes} />
```