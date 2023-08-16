---
title: "Compile Svelte in Your Head（3）"
description: ""
pubDate: "2023-07-31 13:46"
heroImage: "https://images.unsplash.com/photo-1593720213428-28a5b9e94613?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D&auto=format&fit=crop&w=1200&q=80"
date created: "2023-07-31 13:46"
date modified: "2023-07-31"
tags:
    - writings
    - svelte
---

本文将介绍3个DOM相关的指令：
- `on:` 事件绑定
- `bind:`属性的双向绑定
- `use:`绑定`action`（元素创建时调用的函数）

## TOC


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

## Vanilla JavaScript

下面展示在不使用任何框架的情况下实现事件处理程序、绑定和`action`

### Event handler

使用[事件监听器](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener)
```js
element.addEventListener('click', handleClick);
```
事件监听器接受一个可选的第三个参数，用于指定事件处理程序的特征：
```js
element.addEventListener('click', handleClick, {
    capture: true, // triggered before any child element
    once: true, // triggered at most once
    passive: true, // indicates that will never call `preventDefault` to improve performance
});
```

要删除事件处理器，需要使用相同的事件、监听器、`capture`/`useCapture`

### Bindings

绑定是在变量值和属性值之间进行同步。

要将变量同步到属性，需要观察变量的值，当变量变化时将其应用到元素的属性上。
另一方面，为了将元素的属性同步到变量，需要根据属性坚挺元素的事件，并在事件发生时更新变量的值。

```js
// binding variable `checked` with the checkbox `checked` property
let checked;
let input = document.querySelector('#checkbox');

// synchronise variable `checked` to checkbox `checked` property
observe(checked, newValue => {
    input.checked = newValue;
});

// synchronise checkbox `checked` property to variable `checked`
// listen to `change` event for `checked` property
input.addEventListener('change', event => {
    checked = input.checked;
});
```

事件的名称和元素的属性名称可能各不相同，例如：`<input type="text">`可监听`input`事件中value属性变化，但复选框`<input type="checkbox">`监听`checked`属性的`change`事件

:::warning
如果元素没触发任何事件来指示属性已更改，则几乎不可能绑定元素的属性，例如[HTMLDialogElement](https://developer.mozilla.org/en-US/docs/Web/API/HTMLDialogElement)有close事件但没有open事件！
另一种选择是使用[`MutationObserver`](https://developer.mozilla.org/zh-CN/docs/Web/API/MutationObserver)。
:::

### Action

action是一个在DOM元素创建时调用的函数，返回一个对象，有如下两个方法：
- `update`当参数改变时调用
- `destroy`当元素从文档中断开时被调用

```js
function actionFn(element, parameter) {
    return {
        update(newParameter) {},
        destroy() {},
    };
}

// When element is mounted onto the DOM
let parameter = 1;
const actionObj = actionFn(element, parameter);

// When parameter changes
parameter = 2;
actionObj.update(parameter);

// When element is removed from the DOM
actionObj.destroy();
```

## The Compiled JavaScript

现在看经过svelte编译器编译后输出的代码：

### `on:`

```svelte
<script>
    function onChange() {}
</script>
<input on:change={onChange} />
```

output:
```js {11-12,16,26}
function create_fragment(ctx) {
    let input;
    let dispose;

    return {
        c() {
            input = element('input');
        },
        m(target, anchor, remount) {
            insert(target, input, anchor);
            if (remount) dispose();
            dispose = listen(input, 'change', /*onChange*/ ctx[0]);
        },
        d(detaching) {
            if (detaching) detach(input);
            dispose();
        },
    };
}

function instance($$self) {
    let i = 0;
    function onChange() {
        i++;
    }
    return [onChange];
}
```

- svelte在`mount`时添加事件处理程序`listen(...)`
- svelte在`destroy`时删除了事件处理程序`dispose()`

### 事件修饰符

在绑定事件时可以传入事件修饰符，如：
```svelte
<script>
let i=0;
function onClick() {
    i++;
}
</script>

<button on:click|preventDefault={onClick} />
<button on:change|stopPropagation={onClick} />
<button on:change|once={onClick} />
<button on:change|capture={onClick} />

<!-- Chain multiple modifiers -->
<button on:click|preventDefault|stopPropagation|once|capture={onClick} />
```

编译后的代码：
```js {7-18}
function create_fragment(ctx) {
  // ...
    return {
        c() { /* ... */ },
        m(target, anchor, remount) {
          // ...
            dispose = [
                listen(button0, "click", prevent_default(/*onClick*/ ctx[0])),
                listen(button1, "change", stop_propagation(/*onClick*/ ctx[0])),
                listen(button2, "change", /*onClick*/ ctx[0], { once: true }),
                listen(button3, "change", /*onClick*/ ctx[0], true),
                listen(
                    button4,
                    "click",
                    stop_propagation(prevent_default(/*onClick*/ ctx[0])),
                    { once: true, capture: true }
                ),
            ];
        },
    // ...
    };
}
```

- svelte以不同的方式处理事件修饰符
- 对于`capture`、`once`, `passive`将他们作为选项传递到`listen`
- 对于`stopPropagation`、`preventDefault` 和`self`，事件处理器使用各自的装饰器函数嵌套


`preventDefault`装饰器函数的示例实现：
```js
function prevent_default(fn) {
    return function(event) {
        event.preventDefault();
        return fn.call(this, event);
    };
}
```

### `bind:`

```svelte
<script>
    let checked = false;
    function updateChecked() {
        checked = true;
    }
</script>
<input type="checkbox" bind:checked />
```

Output：
```js {10-11,14-16,20,29,32-35}
function create_fragment(ctx) {
    let input;
    let dispose;
    
    return {
        c() { /* ... */ },
        m(target, anchor, remount) {
            insert(target, input, anchor);
            input.checked = /*checked*/ ctx[0];
            if (remount) dispose();
            dispose = listen(input, 'change', /*input_change_handler*/ ctx[1]);
        },
        p(ctx, [dirty]) {
            if (dirty & /*checked*/ 1) {
                input.checked = /*checked*/ ctx[0];
            }
        },
        d(detaching) {
            if (detaching) detach(input);
            dispose();
        },
    };
}

function instance($$self, $$props, $$invalidate) {
    let checked = false;
    
    function updateChecked() {
        $$invalidate(0, (checked = true));
    }
    
    function input_change_handler() {
        checked = this.checked;
        $$invalidate(0, checked);
    }
    
    return [checked, input_change_handler];
}
```

- 将变量值同步到元素属性
    - 用`$$invalidate(...)`包装变量`checked`的更新
    - 在`update`方法中如果变量`checked`更新，则将`input.checked`设置为变量`checked`的值
- 将元素属性的更改同步到变量
    - svelte创建一个`input`事件的`handler`，读取`this.checked`属性值并调用`$$invalidate`更新它
    - svelte在`mount`时设置了`listen(...)`，在`destroy`时`dispose`

### `use:`

```svelte
<script>
let i = '';
function action() {}
function updateI() {
    i++;
}
</script>
<div use:action={i} />
```

Output:
```js {9-12,15-16,20}
function create_fragment(ctx) {
  // ...
    let action_action;
    
    return {
        c() { /* ... */ },
        m(target, anchor, remount) {
            insert(target, div, anchor);
            if (remount) dispose();
            dispose = action_destroyer(
                (action_action = action.call(null, div, /*i*/ ctx[0]))
            );
        },
        p(ctx, [dirty]) {
            if (action_action && is_function(action_action.update) && dirty & /*i*/ 1)
                action_action.update.call(null, /*i*/ ctx[0]);
        },
        d(detaching) {
            if (detaching) detach(div);
            dispose();
        },
    };
}
```

- 在`mount`时通过调用`action`函数创建`action_action`并使用`action_destroyer`包装为`dispose`
- 当参数改变时，使用update方法中更新的参数调用`action_action.update`方法
- 在destroy时调用第一步中由`action_destroyer`返回的`dispose`函数
