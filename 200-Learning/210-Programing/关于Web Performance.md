---
title: "关于Web Performance"
description: "This artical will show you some core web vitals"
pubDate: "2023-07-19 12:12"
heroImage: "https://web-dev.imgix.net/image/ZDZVuXt6QqfXtxkpXcPGfnygYjd2/m80oUd2zASInyKJJ7QNc.png?auto=format&w=1600"
date created: 2023-07-19
date modified: 2023-07-19
tags:
    - writings
    - Programming
    - FE
---

>昔日所云我，而今却是伊。不知今日我，又属后来谁？  —《菜根谭》

[Core Web Vitals](https://web.dev/learn-core-web-vitals/)

:::info
等待资源加载时间和大部分情况下的浏览器单线程执行是影响 Web 性能的两大主要原因。
:::

Web 性能是客观的衡量标准，是用户对加载时间和运行时的直观体验。Web 性能指页面加载到可交互和可响应所消耗的时间，以及页面在交互时的流畅度——滚动是否顺滑？按钮能否点击？弹窗能否快速打开，动画是否平滑？Web 性能既包括客观的度量如加载时间，每秒帧数和到页面可交互的时间；也包括用户的对页面内容加载时间的主观感觉。

页面响应时间越长，越多的用户就会放弃该网站。重要的是，通过使体验尽可能早地变得可用和交互，同时异步地加载长尾体验部分，来最大程度地减少加载和响应时间，并添加其他功能以降低延迟。

有很多工具，API 和最佳实践帮助我们测量和改进网页性能。本章对此有所讲解。

## 健康网站的关键指标（Web Vitas）

Web 指标是 Google 开创的一项新计划，旨在为网络质量信号提供统一指导，这些信号对于提供出色的网络用户体验至关重要。

网站所有者要想了解他们提供给用户的体验质量，并非需要成为性能专家。 Web 指标计划旨在简化场景，帮助网站专注于最重要的指标，即**核心 Web 指标** 。

> [!info]
>核心性能指标：核心 Web 指标是适用于所有网页的 Web 指标子集，每位网站所有者都应该测量这些指标，并且这些指标还将显示在所有 Google 工具中。每项核心 Web 指标代表用户体验的一个不同方面，能够进行[实际](https://web.dev/user-centric-performance-metrics/#how-metrics-are-measured)测量，并且反映出[以用户为中心](https://web.dev/user-centric-performance-metrics/#how-metrics-are-measured)的关键结果的真实体验。

核心Web指标的构成会随着时间的推移而发展。当前针对2020年的指标构成侧重于用户体验的三个方面——加载性能、交互性、和视觉稳定性——并包括一下指标（及各指标的阀值）：

![https://web-dev.imgix.net/image/tcFciHGuF3MxnTr1y5ue01OGLBn2/ZZU8Z7TMKXmzZT2mCjJU.svg](https://web-dev.imgix.net/image/tcFciHGuF3MxnTr1y5ue01OGLBn2/ZZU8Z7TMKXmzZT2mCjJU.svg)

![https://web-dev.imgix.net/image/tcFciHGuF3MxnTr1y5ue01OGLBn2/iHYrrXKe4QRcb2uu8eV8.svg](https://web-dev.imgix.net/image/tcFciHGuF3MxnTr1y5ue01OGLBn2/iHYrrXKe4QRcb2uu8eV8.svg)

![https://web-dev.imgix.net/image/tcFciHGuF3MxnTr1y5ue01OGLBn2/dgpDFckbHwwOKdIGDa3N.svg](https://web-dev.imgix.net/image/tcFciHGuF3MxnTr1y5ue01OGLBn2/dgpDFckbHwwOKdIGDa3N.svg)

- **[Largest Contentful Paint (LCP)](https://web.dev/lcp/)** ：最大内容绘制，测量_加载_性能。为了提供良好的用户体验，LCP 应在页面首次开始加载后的**2.5 秒**内发生。
- **[First Input Delay (FID)](https://web.dev/fid/)** ：首次输入延迟，测量_交互性_。为了提供良好的用户体验，页面的 FID 应为**100 毫秒**或更短。
- **[Cumulative Layout Shift (CLS)](https://web.dev/cls/)** ：累积布局偏移，测量_视觉稳定性_。为了提供良好的用户体验，页面的 CLS 应保持在 **0.1.** 或更少。

对于上述每项指标，一个良好的测量阈值为75%，且该阈值同时适用于移动和桌面设备。如果一个页面满足上述全部三项建议的75%，那么评估核心Web指标合规性的工具应评判该页面通过。

### 测量工具

1. **Chrome UX Report:** [Chrome 用户体验报告](https://developer.chrome.com/docs/crux/)为每项核心 Web 指标收集匿名的真实用户测量数据。这些数据既能使网站所有者快速进行性能评估，而无需在页面上进行手动检测分析
2. 通过使用[web-vitals](https://github.com/GoogleChrome/web-vitals)库，测量每项指标就像调用单个函数一样简单

    ```jsx

import {getCLS, getFID, getLCP} from 'web-vitals';

function sendToAnalytics(metric) {
  const body = JSON.stringify(metric);
  // Use `navigator.sendBeacon()` if available, falling back to `fetch()`.
  (navigator.sendBeacon && navigator.sendBeacon('/analytics', body)) ||
      fetch('/analytics', {body, method: 'POST', keepalive: true});
}

getCLS(sendToAnalytics);
getFID(sendToAnalytics);
getLCP(sendToAnalytics);
    ```
3. 可以使用[Web 指标 Chrome 扩展程序](https://github.com/GoogleChrome/web-vitals-extension)来报告每项核心 Web 指标，且无需编写任何代码。该扩展程序使用[web-vitals](https://github.com/GoogleChrome/web-vitals)库来测量每一项指标，并在用户浏览网页时呈现给用户。
4. Chrome开发者工具、Lighthouse

### 提高分数的建议

[优化LCP](https://www.notion.so/LCP-1a764bffa6e54c9a99d7a6a4a2a71be0?pvs=21)

- 优化FID
- 优化CLS

### 其他Web指标

其他 Web 指标通常用作核心 Web 指标的代理或补充指标，有助于获取范围更广的体验或帮助诊断特定的问题。

例如，[Time to First Byte 首字节时间 (TTFB)](https://web.dev/ttfb/)和[First Contentful Paint 首次内容绘制 (FCP)](https://web.dev/fcp/)指标都是_加载_体验的重要方面，并且在诊断 LCP 问题方面（分别为[服务器响应时间](https://web.dev/overloaded-server/)过长或[阻塞渲染资源](https://web.dev/render-blocking-resources/)）都十分有用。

同样，[总阻塞时间 (TBT)](https://web.dev/tbt/)和[Time to Interactive 可交互时间 (TTI)](https://web.dev/tti/)等指标是实验室指标，对于捕获和诊断会对 FID 产生影响的潜在_交互性_问题至关重要。然而，这些指标不是核心 Web 指标的一部分，因为它们无法进行实际测量，也不反映以[用户为中心](https://web.dev/user-centric-performance-metrics/#how-metrics-are-measured)的结果。

## 关键性能指南

### CSS & JS 动画性能

对众多应用程序而言，动画对提供友好的用户体验有着关键的作用，我们有很多方式生成 web 动画，比如 CSS `transition`/`animation` 或者 JavaScript 动画 (使用 `Window.requestAnimationFrame`).

### DNS 预获取

**`DNS-prefetch`** (**DNS 预获取**) 是尝试在请求资源之前解析域名。这可能是后面要加载的文件，也可能是用户尝试打开的链接目标。

### 懒加载

**延迟加载 (懒加载)** 是一种将资源标识为非阻塞（非关键）资源并仅在需要时加载它们的策略。 这是一种缩短[关键渲染路径](https://developer.mozilla.org/zh-CN/docs/Web/Performance/Critical_rendering_path)长度的方法，可以缩短页面加载时间。

<aside> 🌀 关键渲染路径：是浏览器将 HTML，CSS 和 JavaScript 转换为屏幕上的像素所经历的步骤序列。优化关键渲染路径可提高渲染性能。关键渲染路径包含了 [文档对象模型](https://developer.mozilla.org/zh-CN/docs/Web/API/Document_Object_Model) (DOM)，[CSS 对象模型](https://developer.mozilla.org/zh-CN/docs/Web/API/CSS_Object_Model) (CSSOM)，渲染树和布局。

</aside>

### 性能监控

**合成监控**和**真实用户监控** (RUM) 是两种监视和提供 Web 性能见解的方法。

### 优化启动性能

一个在软件应用开发中经常被忽视的方面—即便在那些专注于性能优化的软件中—就是启动时的表现。你的应用需要花费多长时间启动？当应用加载时是否会卡住用户的设备或浏览器？这会让用户担心你的应用崩溃了，或者哪儿出错了。花时间确保你的应用能够更好地启动总是一个好主意。

## Performance 使用指南

1. 打开新的隐私窗口
    隐私窗口运行在一个纯净的浏览器环境，不会加载多余的浏览器扩展。
2. 在隐私窗口打开如下地址：用于测试Profile的demo，包含了一系列运动的小球
    [Janky Animation](https://googlechrome.github.io/devtools-samples/jank/)
3. 通过`Command+Option+I` 组合键（Mac）或`Control+Shift+I`(Windows)打开开发者工具

在`性能` 面板，可点击设置图标，选择`CPU` 速度


