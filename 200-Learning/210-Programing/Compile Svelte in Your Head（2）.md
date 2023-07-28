---
title: "Compile Svelte in Your Head（2）"
description: ""
pubDate: "2023-07-28 19:06"
heroImage: "https://images.unsplash.com/photo-1536319040287-757e83a8198e?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D&auto=format&fit=crop&w=1200&q=80"
date created: "2023-07-28 19:03"
date modified: "2023-07-28"
draft: true
tags:
    - writings
---

`$$invalidate`是Svelte响应式背后的秘密㊙️。每当一个变量是：
- reassigned（foo = 1）
- mutated（foo.bar = 1）
svelte将用`$$invalidate`函数包装赋值或更新：

```js
name = 'Svelte';
count++;
foo.a = 1;
bar = baz = 3;
// compiled into
$$invalidate('name', (name = 'Svelte'));
$$invalidate('count', count++, count);
$$invalidate('foo', (foo.a = 1), foo);
$$invalidate('bar', (bar = $$invalidate('baz', (baz = 3))));
```



## `$$invalidate`

`$$invalidate`在概念上的工作原理如下：
```js
// conceptually...
const ctx = instance(/*...*/);
const fragment = create_fragment(ctx);
// to track which variable has changed
const dirty = new Set();
const $$invalidate = (variable, newValue) => {
    // update ctx
    ctx[variable] = newValue;
    // mark variable as dirty
    dirty.add(variable);
    // schedules update for the component
    scheduleUpdate(component);
};

// gets called when update is scheduled
function flushUpdate() {
    // update the fragment
    fragment.p(ctx, dirty);
    // clear the dirty
    dirty.clear();
}
```
但这并非其确切实现，在本文中，我们将了解`$$invalidate`在svelte中是如何实现的。

