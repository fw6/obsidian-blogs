---
title: "Compile Svelte in Your Head"
description: ""
pubDate: "2023-07-21 19:06"
heroImage: "https://images.unsplash.com/photo-1536319040287-757e83a8198e?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D&auto=format&fit=crop&w=1200&q=80"
date created: "2023-07-21 19:05"
date modified: "2023-07-21"
draft: true
tags:
    - writings
---

> The winner ain't the one with the fastest car it's the one who refuses to lose.
> — <cite>Dale Earnhardt</cite>

## Svelte syntax

下方展示svelte的基础语法

:::note
希望了解更多？强烈推荐[svelte interactive tutorial](https://svelte.dev/tutorial/basics)
:::

一个基础的Svelte 单文件组件：
```svelte
<h1>Hello world</h1>
```
[Svelte REPL](https://svelte.dev/repl/99aeea705b1e48fe8610b3ccee948280?version=4.1.1)

增加点样式：
```svelte
<h1>Hello world</h1>

<style>
    h1 {
        color: purple;
    }
</style>
```

给组件增加点数据：
```svelte
<script>
let name = "world"
</script>

<h1>Hello {name}</h1>
```

绑定事件：
```svelte
<script>
let count = 0;

function onClickButton(event) {
    count++;
}
</script>

<button>Clicked {count}</button>
```

接下来看下，上面的Svelte语法是怎么编译为JavaScript的

## Compile Svelte in your Head

svelte编译器分析编写的代码并生成优化后输出为JavaScript

为研究Svelte如何编译，我们从简单的例子出发，逐步构建代码。通过这个过程，将可看到svelte根据更改逐步添加到输出代码中。

第一个例子如下：
```svelte
<h1>Hello world</h1>
```
[Svelte REPL](https://svelte.dev/repl/99aeea705b1e48fe8610b3ccee948280?version=4.1.1)

JavaScript Output
```js
function create_fragment(ctx) {
	let h1;

	return {
		c() {
			h1 = element("h1");
			h1.textContent = "Hello World";
		},
		m(target, anchor) {
			insert(target, h1, anchor);
		},
		p: noop,
		i: noop,
		o: noop,
		d(detaching) {
			if (detaching) {
				detach(h1);
			}
		}
	};
}

class App extends SvelteComponent {
	constructor(options) {
		super();
		init(this, options, null, create_fragment, safe_not_equal, {});
	}
}
```
输出的代码分为两部分：
- `create_fragment`
- `class App extends SvelteComponent`

### create_fragment

Svelte组件是Svelte应用的一个个构建块。每一个组件只关注于构建最终DOM的一部分或片段。
create_fragment函数为svelte组件提供了如何构建DOM片段的规范。

create_fragment函数返回一个有如下方法的对象：
- c()。create的缩写。包含创建元素的所有指令
- m()。mount的缩写。包含如何将创建的元素与DOM树连接
- d()。destroy的缩写。将当前节点从DOM树中移除

### class App extends SvelteComponent

每个svelte组件都是一个class，你可以参考[这个文档](https://svelte.dev/docs/client-side-component-api)导入和实例化它。
在构造函数中，我们使用组成组件的相关信息初始化组件，例如create_fragment。svelte只会传递需要的信息，并在不必要的时候删除他们。

例如注释掉`<h1>`标签，得到JavaScript为：
```svelte
<!-- empty -->
```
```js
class App extends SvelteComponent {
	constructor(options) {
		super();
		init(this, options, null, null, safe_not_equal, {});
	}
}
```
svele将传入null而不是create_fragment。

init函数是初始化大部分内置功能的地方，例如：
- 组件的props，ctx和context
- 组件的生命周期事件
- 组件的更新机制
最后，svelte调用create_fragment来创建元素和与DOM树进行连接

所有的内部状态和方法都附加到`this.$$`上了。因此如果你直接使用`this.$$`访问组件内部属性，需要尤其注意⚠️👮

### Adding Data

现在让我们给组件增加点数据：
```svelte
<script>
let name = 'world'
</script>

<h1>Hello {name}</h1>
```

JavaScript Output：
```js
function create_fragment(ctx) {
	let h1;

	return {
		c() {
			h1 = element("h1");
			h1.textContent = `Hello ${name}`;
		},
		m(target, anchor) {
			insert(target, h1, anchor);
		},
		p: noop,
		i: noop,
		o: noop,
		d(detaching) {
			if (detaching) detach(h1);
		}
	};
}

let name = "world";

class App extends SvelteComponent {
	constructor(options) {
		super();
		init(this, options, null, create_fragment, safe_not_equal, {});
	}
}
```

与第一个例子相比，生成的代码中类似如下结构：
```js
let name = 'world'
{
    h1.textContent = `Hello ${name}`;
}
```

- 在script中写的代码被放到了顶级作用域
- h1的内容变为了模板字面量

### Updating Data
让我们添加一个函数用于更新name：
```svelte
<script>
let name = 'world'
function update() {
    name = 'Svelte'
}
</script>

<h1>Hello {name}</h1>
```
[Svelte REPL](https://svelte.dev/repl/3237a87f7da947cb845aa2cbc9d8ea2e?version=4.1.1)

JavaScript Output:
```js {2-4,10,17-19,28-36, 41}
function create_fragment(ctx) {
	let h1;
	let t0;
	let t1;

	return {
		c() {
			h1 = element("h1");
			t0 = text("Hello ");
			t1 = text(/*name*/ ctx[0]);
		},
		m(target, anchor) {
			insert(target, h1, anchor);
			append(h1, t0);
			append(h1, t1);
		},
		p(ctx, [dirty]) {
			if (dirty & /*name*/ 1) set_data(t1, /*name*/ ctx[0]);
		},
		d(detaching) {
			if (detaching) {
				detach(h1);
			}
		}
	};
}

function instance($$self, $$props, $$invalidate) {
	let name = 'World';

	function update() {
		$$invalidate(0, name = 'Svelte');
	}

	return [name];
}

class App extends SvelteComponent {
	constructor(options) {
		super();
		init(this, options, instance, create_fragment, safe_not_equal, {});
	}
}
```

多出的地方：
- `h1`的内容被拆分为两个文本元素
- `create_fragment`返回的对象增加了一个新的方法`p`
- 新的函数`instance`被创建
- 在svelte组件的script内写的内容被放到了`instance`
- 在`create_fragment`中被用到的数据`name`现在被替换为`ctx[0]`

svelte编译器跟踪所有在script中定义的变量，它跟踪变量是否：
1. 可以被改变，如：`count++`
2. 可被重新赋值，如：`name = ‘svelte’`
3. 是否在模板中被引用？如：`<h1>Hello {name}</h1>`
4. 是否可写？例如：`const i = 1` vs `let i = 1`
5. ...more

当svelte编译器意识到变量名可被重新分配时，它会将h1的文本内容分解为多个部分，这样他就能动态更新部分文本。

你可以看到新方法来更新文本节点：

#### `p(ctx, dirty)`

update的简写。它根据状态（dirty）和组件状态（ctx）的变化来更新元素

#### instance variable

变量名不能在不同组件实例上共享，这就是为什么它将变量名移动到一个名为instance的函数中。

instance函数返回组件实例的变量列表，这些变量是：
1. 在模板中被使用
2. 被改变或重新赋值（在组件的实例内被更改）
在组件中，这个列表被称为`ctx`
在init函数中，svelte执行instance函数创建ctx，然后使用它创建fragment：
```js
const ctx = instance(/*...*/)
const fragment = create_fragment(ctx)
// create the fragment
fragment.c()
// mount the fragment onto the DOM
fragment.m(target)
```

现在不访问组件实例的变量名，而是通过ctx拿到name的值：
```js
t1 = text(/*name*/ ctx[0])
```
ctx是数组而非对象或Map，与一项位掩码的优化技术有关，参考[相关讨论](https://github.com/sveltejs/svelte/issues/1922) [stackoverflow](https://stackoverflow.com/questions/59541070/how-is-svelte-making-a-component-dirty)
``
#### `$$invalidate`

svelte响应式系统背后的秘密是`$$invalidate`函数，每一个变量如果可被：
- `reassigned/mutated`
- 被模板引用
都会有`$$invalidate`函数被插入到`reassigned/mutated`所在位置的右侧:
```js
name = 'svelte';
count++;
foo.a = 1;

// compiled
name='svelte';
$$invalidate(/* name */, name);

count++;
$$invalidate(/* count */, count);

foo.a = 1;
$$invalidate(/* foo */, foo);
```

`$$invalidate`函数将变量标记为`dirty`并安排组件的更新：
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

### Adding event listeners

在svelte组件中增加一个事件监听:
```svelte {7}
<script>
	let name = 'world';
	function update() {
		name = 'Svelte';
	}
</script>
<h1 on:click={update}>Hello {name}</h1>
```
[Svelte REPL](https://svelte.dev/repl/a9893863475049149ca72e1416eef7e5?version=4.1.1)

JavaScript Output:
```js {20,33,45}
function create_fragment(ctx) {
	let h1;
	let t0;
	let t1;
	let mounted;
	let dispose;

	return {
		c() {
			h1 = element("h1");
			t0 = text("Hello ");
			t1 = text(/*name*/ ctx[0]);
		},
		m(target, anchor) {
			insert(target, h1, anchor);
			append(h1, t0);
			append(h1, t1);

			if (!mounted) {
				dispose = listen(h1, "click", /*update*/ ctx[1]);
				mounted = true;
			}
		},
		p(ctx, [dirty]) {
			if (dirty & /*name*/ 1) set_data(t1, /*name*/ ctx[0]);
		},
		d(detaching) {
			if (detaching) {
				detach(h1);
			}

			mounted = false;
			dispose();
		}
	};
}

function instance($$self, $$props, $$invalidate) {
	let name = 'world';

	function update() {
		$$invalidate(0, name = 'Svelte');
	}

	return [name, update];
}

class App extends SvelteComponent {
	constructor(options) {
		super();
		init(this, options, instance, create_fragment, safe_not_equal, {});
	}
}
```

多出的地方：
1. instance函数返回了内部的update函数
2. 在mount时添加了事件监听，destroy时移除事件

如上所述，instance函数返回模板中引用的变量，并且这些变量会被修改或重分配。
因在template中引用了update函数，所以update也作为了ctx的一部分被返回

svelte编译器尝试生成紧凑的JavaScript代码，如果非必要则不会返回额外的变量。

#### listen & dispose

每当在svelte组件中增加事件监听器时，svelte都会注入代码添加事件，并在DOM片段被移除时将事件移除。

当有多个事件被绑定时，svelte编译器会压缩多个事件：
```js {1-5,7}
dispose = [
  listen(h1, 'click', /*update*/ ctx[1]),
  listen(h1, 'mousedown', /*update*/ ctx[1]),
  listen(h1, 'touchstart', /*update*/ ctx[1], { passive: true }),
];
// ...
run_all(dispose);
```


## 总结

svelte语法是HTML的超集。

一个svelte组件会通过svelte编译器分析并生成优化后的JavaScript代码。

输出的JavaScript代码可分成3部分：
### 1. create_fragment
返回一个对象，包含了通过组件创建一个元素片段所需的方法。

### 2. instance

1. 写在script标签内的大多数代码作为instance的内容
2. 返回一个数组表示组件实例的若干状态（可变动、且在template中被引用的变量）
3. `$$invalidate`被插入到每个改变变量值的后面。

### 3. class App extends SvelteComponent
