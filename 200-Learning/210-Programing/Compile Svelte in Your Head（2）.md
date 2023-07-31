---
title: "Compile Svelte in Your Head（2）"
description: ""
pubDate: "2023-07-28 19:06"
heroImage: "https://images.unsplash.com/photo-1617791160536-598cf32026fb?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D&auto=format&fit=crop&w=1200&q=80"
date created: "2023-07-28 19:03"
date modified: "2023-07-28"
tags:
    - writings
    - svelte
---

## TOC

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

`$$invalidate`函数将：
1. 更新`$$ctx`的变量
2. 在`$$.dirty`中标记变量
3. schedule update
4. 返回赋值或更新表达式的值

```js
//https://github.com/sveltejs/svelte/blob/99a3cc93b66bf2c6be551e23101bfdbdb2c6bf72/packages/svelte/src/runtime/internal/Component.js#L124C2-L133C8
// 1. update the variable in $$.ctx
$$.ctx = instance
		? instance(component, options.props || {}, (i, ret, ...rest) => {
				const value = rest.length ? rest[0] : ret;
				if ($$.ctx && not_equal($$.ctx[i], ($$.ctx[i] = value))) {
					if (!$$.skip_bound && $$.bound[i]) $$.bound[i](value);
					// 2a. mark the variable in $$.dirty
					if (ready) make_dirty(component, i);
				}
    			// 4. return the value of the assignment or update expression
				return ret;
		  })
		: [];


//https://github.com/sveltejs/svelte/blob/99a3cc93b66bf2c6be551e23101bfdbdb2c6bf72/packages/svelte/src/runtime/internal/Component.js#L78
/** @returns {void} */
function make_dirty(component, i) {
	if (component.$$.dirty[0] === -1) {
		dirty_components.push(component);
		// 3. schedule an update
		schedule_update();
		// initialise $$.dirty
		component.$$.dirty.fill(0);
	}
	// 2b. mark the variable in $$.dirty
	component.$$.dirty[(i / 31) | 0] |= 1 << i % 31;
}

```

在3.16.0以前，svelte源码使用对象将变量标记为脏：
```js
$$.dirty = { givenName: true, familyName: false };
```

在之后使用位掩码的技术跟踪更改。
svelte为每个变量分配索引:
```
givenName -> 0
familyName -> 1
```
并使用位掩码存储脏信息：
```js
$$.dirty = [0b0000_0011]
// 0和1位标记为true
```

### Bitmask

最紧凑的的方式表示一组true或false的方式，是使用位。如果该位为1，则为true，为0则为false。

数字可以使用二进制表示，5就是二进制的`0b0101`

如果5用4位二进制表示，那么可以存储4个布尔值，第0和第2位为true，第1位和第3位为false（从右到左，从最低有效位到最高有效位）。

在JavaScript中，数字可以用64位表示。但是，当对数字使用按位运算时，JavaScript会将数字视为32位。
要坚持和修改存储在数字中的布尔值，可以使用按位运算：
```js
// set 1st boolean to true
0b0101 | 0b0010 = 0b0111;

// set 2nd boolean to false
0b0101 & 0b1011 = 0b0001;

// is 2nd boolean true?
((0b0101 & 0b0100) > 0) === true;

// NOTE: You can test multiple boolean values at once
// is 2nd and 3rd boolean true?
((0b0101 & 0b1100) > 0) === true;
```
在位运算中使用的第二个操作数，就像掩码，允许我们定位第一个数字中存储布尔值的特定位。
在这里称掩码为位掩码。

### Bitmask in Svelte

如上所述，为每个变量分配一个索引：
```
givenName -> 0
familyName -> 1
```
在instance函数中，返回的ctx作为数组：
```js {3}
function instance($$self, $$props, $$invalidate) {
  // ...
  return [givenName, familyName];
}
```

此后访问变量可通过索引，而非变量名：
```js
$$.ctx[0] + $$.ctx[1];
```

`$$invalidate`函数的工作原理相同：
```js
$$invalidate(0, (givenName = 'Li Hau'));
```

`$$.dirty`现在也存储数组，数组每一项携带31个布尔值，每个布尔值指示该索引的变量是否为`dirty`.

要将数据设置为脏变量，可直接使用位运算：
```js
$$.dirty[0] |= 1 << 0;
```
要验证是否位脏，也使用位运算：
```js
if ($dirty[0] & 1) { /* ... */ }
if ($dirty[0] & 3) { /* ... */ }
```

使用位掩码后，`$$.dirty`重置为`[-1]`而不是`null`

:::tip
`-1`是二进制的`0b1111_1111`，所有位都为`1`
:::

## 响应式声明

svelte允许使用label语句申明响应值：
```svelte {3-6}
<script>
  export let count = 0;
  // `doubled`, `tripled`, `quadrupled` are reactive
  $: doubled = count * 2;
  $: tripled = count * 3;
  $: quadrupled = doubled * 2;
</script>
{doubled} {tripled} {quadrupled}
```

如果查看编译后的输出，会发现声明语句出现在instance函数中：
```js {3-13}
function instance($$self, $$props, $$invalidate) {
    // ...
    $$self.$$.update = () => {
        if ($$self.$$.dirty & /*count*/ 8) {
            $: $$invalidate(0, doubled = count * 2);
        }
        if ($$self.$$.dirty & /*count*/ 8) {
            $: $$invalidate(1, tripled = count * 3);
        }
        if ($$self.$$.dirty & /*doubled*/ 1) {
            $: $$invalidate(2, quadrupled = doubled * 2);
        }
    };
    
	return [doubled, tripled, quadrupled, count];
}
```

- 当存在响应式声明时，svelte定义`$$.update`方法。
    - `$$.update`函数默认是无操作函数
- svelte也使用`$$invalidate`更新响应式变量的值
- svelte根据申明与语句之间的依赖关系对响应式声明和语句排序

由于所有响应式声明和语句都分组到`$$.update`方法中，而且svelte会根据它们的依赖关系对声明和语法排序，因此最终输出结果与源码中声明的位置和顺序无关。

以下组件是有效的：
```svelte
<script>
// NOTE: use `count` in a reactive declaration before `count` is declared
$: doubled = count * 2;
let count = 1;
</script>
{count} * 2 = {doubled}
```

在`flush`函数中调用了一个`update`函数，其结构如下：
```js {4}
//https://github.com/sveltejs/svelte/blob/99a3cc93b66bf2c6be551e23101bfdbdb2c6bf72/packages/svelte/src/runtime/internal/scheduler.js#L113
function update($$) {
	if ($$.fragment !== null) {
		$$.update();
		run_all($$.before_update);
		const dirty = $$.dirty;
		$$.dirty = [-1];
		$$.fragment && $$.fragment.p($$.ctx, dirty);
		$$.after_update.forEach(add_render_callback);
	}
}
```

`$$.update`函数在DOM更新的同一个微任务中被调用，就是调用`$$.fragment.p()`更新DOM之前。

从上方update函数可得出：

### 所有响应式声明和语句都是批处理的

正如DOM更新的批处理方式一样：
```svelte
<script>
let givenName = '', familyName = '';
function update() {
    givenName = 'Li Hau';
    familyName = 'Tan';
}
$: name = givenName + " " + familyName;
$: console.log('name', name);
</script>
```
当update被调用时：
1. 与上述流程类似，为givenName和familyName执行`$$invalidate`，并安排更新
2. 任务结束
3. 微任务开始
4. `flush()`执行时为每个标记为脏的组件调用`update`函数
5. 允许`$$.update`
    1. 由于`givenName`和`familyName`被更改，执行`name`的`$$invalidate`
    2. 由于`name`被更改，执行`console.log('name', name);`
6. 调用`$$.fragment.p(...)`更新DOM

如⬆上，即使更新了`givenName`和`familyName`，也只更新`name`和执行`console.log('name', name);`一次而不是两次

### 响应式声明或语句外的响应式变量可能不是最新的

由于反应式声明和语句是在下一个微任务中批量执行的，因此您不能期望值会同步更新。
```svelte {6}
<script>
let givenName = '', familyName = '';
function update() {
    givenName = 'Li Hau';
    familyName = 'Tan';
    console.log('name', name); // Logs ''
}
$: name = givenName + " " + familyName;
</script>
```

可在另一个响应式语句中引用响应式变量：
```svelte {8}
<script>
let givenName = '', familyName = '';
function update() {
    givenName = 'Li Hau';
    familyName = 'Tan';
}
$: name = givenName + " " + familyName;
$: console.log('name', name); // Logs 'Li Hau Tan'
</script>
```

### 响应式声明和语句的排序

Svelte 尝试尽可能保留反应式声明和语句的声明顺序。
但是，如果一个反应式声明或语句引用了另一个反应式声明定义的变量，那么它将被插入到后一个反应式声明之后：

```svelte {4,6,12-13}
<script>
let count = 0;
// NOTE: refers to `doubled`
$: quadrupled = doubled * 2;
// NOTE: defined `doubled`
$: doubled = count * 2;

// compiles into:

$$self.$$.update = () => {
    // ...
    $: $$invalidate(/* doubled */, doubled = count * 2);
    $: $$invalidate(/* quadrupled */, quadrupled = doubled * 2);
    // ...
}
</script>
```

### 非响应式变量

svelte编译器会跟踪`<script>`脚本中所有变量。如果响应式声明或语句中变量使用了但未发生`mutated`或`reassigned`，则该响应式声明或语句将不会被添加到`$$.update`中。

```svelte
<script>
  let count = 0;
  $: doubled = count * 2;
</script>
{ count } x 2 = {doubled}
```
[Svelte REPL](https://svelte.dev/repl/23b2476accac46608807c73539b03953?version=3.20.1)

JavaScript output
```js
function instance($$self, $$props, $$invalidate) {
	let doubled;
	$: $$invalidate(0, doubled = count * 2);
	return [doubled];
}
```

由于`count`未被`mutated`或`reassigned`，svelte通过不定义`$$self.$$.update`来优化输出。


## 总结

1. Svelte在编译阶段追踪被`dirty`的变量并在DOM更新时批量更新
2. Svelte使用位掩码技术生成更加紧凑的运行时JavaScript代码
3. 响应式声明和语句和DOM更新类似，也是批量执行


