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
```js
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