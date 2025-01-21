---
title: HTMX HDA
description: 
pubDate: 2025-01-20 19:53
heroImage: https://images.unsplash.com/photo-1731329295698-26faaf820c87?crop=entropy&cs=srgb&fm=jpg&ixid=M3w2Mjc5MjV8MHwxfHJhbmRvbXx8fHx8fHx8fDE3MzczNzQwMDd8&ixlib=rb-4.0.3&q=85&w=1200h=400
date created: 2025-01-20 19:53
date modified: 2025-01-20
draft: true
tags:
  - writings
---

# HTMX HDA

> [!quote] You can observe a lot just by watching.
> — Yogi Berra

Hypermedia Driven Application（HDA）架构，结合了传统多页应用程序的简单性和灵活性以及SPA应用更好的用户体验。

HDA架构通过扩展现有的Web HTML基础架构来实现。由两个约束表征：
- 使用声明式HTML嵌入语法而不是命令时脚本实现更好的前端交互性
- 使用HTML而不是非超媒体格式（如JSON）与服务进行通信

## AJAX

- hx-get
- hx-post
- hx-put
- hx-patch
- hx-delete

```html
<button hx-put="/messages">
    Put To Messages
</button>
```

用户点击button，发起PUT请求并将响应加载到按钮

### Trigger

默认情况下，AJAX请求通过事件触发：
- `input`、`textarea` & `select`通过`change`事件
- `form`通过`submit`事件
- 其他通过`click`事件

可通过`hx-trigger`属性事件类型：
```html
<div hx-post="/mouse_entered" hx-trigger="mouseenter">
    [Here Mouse, Mouse!]
</div>
```

#### Trigger Modifiers

`trigger`可通过一些`modifier`改变行为。

例如，请求只发一次：
```html
<div hx-post="/mouse_entered" hx-trigger="mouseenter once">
    [Here Mouse, Mouse!]
</div>
```
其他`modifier`还包括：
- `changed`：只当元素值发生变化时才发出请求
- `delay:<time interval>`：在发起请求前等待一段时间，事件重新触发则定时器刷新
- `throttle:<time interval>`：与延迟不同的是，到达时限前发生新事件，事件还是会被执行。
- `from:<CSS Selector>`：监听不同元素上的事件

```html
<input type="text" name="q"
    hx-get="/trigger_delay"
    hx-trigger="keyup changed delay:500ms"
    hx-target="#search-results"
    placeholder="Search...">
<div id="search-results"></div>
```

通过`keyup`触发事件且input的值发生改变，发起请求前等待500ms

#### Trigger Filters

可以在事件名后使用方括号应用触发筛选器，方括号内可包含可被执行的JS表达式

```html
<div hx-get="/clicked" hx-trigger="click[ctrlKey]">
    Control Click Me
</div>
```

像`ctrlKey`