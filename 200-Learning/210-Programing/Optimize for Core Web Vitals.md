---
title: "Optimize for Core Web Vitals"
description: ""
pubDate: "2023-08-13 17:43"
heroImage: "https://images.unsplash.com/photo-1548957318-e769d68f2ce7?crop=entropy&cs=srgb&fm=jpg&ixid=M3wzNjM5Nzd8MHwxfHJhbmRvbXx8fHx8fHx8fDE2OTE5MTk4NDN8&ixlib=rb-4.0.3&q=85"
date created: "2023-08-13 17:43"
date modified: "2023-08-13"
draft: true
tags:
    - writings
---

> Beauty is not in the face; beauty is a light in the heart.
> — <cite>Kahlil Gibran</cite>

- [Core Web Vitals](https://web.dev/learn-core-web-vitals/)
- [Optimize Largest Contentful Paint](https://web.dev/optimize-lcp/)
- [Optimize First Input Delay](https://web.dev/optimize-fid/)
- [Optimize Cumulative Layout Shift](https://web.dev/optimize-cls/)
- [Optimize for Core Web Vitals - YouTube](https://www.youtube.com/watch?v=AQqFZ5t8uNc&t=1073s)

## 优化LCP

最大内容绘制(LCP) 是核心 Web 指标中的一项指标，用于测量可视区域中最大内容元素变为可见的时间点。该项指标可用于确定页面主要内容在屏幕上完成渲染的时间点。

![image.png](https://raw.githubusercontent.com/fw6/assets/main/toy_docs/20230813204629.png)

导致 LCP 不佳的最常见原因是：

- 缓慢的服务器响应速度
- 阻塞渲染的 JavaScript 和 CSS
- 缓慢的资源加载速度
- 客户端渲染

### 缓慢的服务器响应速度

改进TTFB：
- 优化服务器基础设施
- CDN
- 缓存资产
  反向代理缓存、云服务缓存服务、CDN内容缓存
- 优先使用缓存提供HTML页面
  service worker
- 尽早建立第三方连接
  `rel="preconnect"`或`rel="dns-prefetch"`
- 使用签名交换技术（？[Signed Exchanges (SXGs)](https://web.dev/signed-exchanges/)）

### 阻塞渲染的JavaScript和CSS

优化CSS：
- 削减CSS
- 延迟加载非关键CSS
  [参考](https://www.filamentgroup.com/lab/load-css-simpler/)`<link rel="stylesheet" href="/path/to/my.css" media="print" onload="this.media='all'">`
- 内联关键CSS
  可减少请求

优化JavaScript：
- 削减压缩JavaScript
- 延迟加载未使用的JavaScript
  添加属性：`defer`、`async`
- 最大限度减少未使用的polyfill

### 缓慢的资源加载速度

优化和压缩图像：
- 首先考虑不使用图像
- 将图像转为更新的格式（JPEG2000、JPEG XR或WebP）
- 使用响应式图像
- 考虑使用CDN
- 添加`loading="lazy"`属性

预加载重要资源：
- `rel="preload"`

压缩文本文件：
- 检查服务器是否已自动压缩文件
- 优先使用Brotli算法，能提供更好的压缩率
- 构建中提前压缩资产，减少服务器开销

自适应服务：
- 基于网络质量的自适应服务，针对不同设备与连接使用不同资源

使用Service Worker缓存资产

## 优化FID

首次输入延迟 (FID) 是核心 Web 指标中的一项指标，可捕获用户对网站交互性和响应度的第一印象。该项指标测量从用户第一次与您的网站交互直到浏览器实际能够对交互作出响应的时间。FID 是一项实际指标，无法在实验室环境中进行模拟。该项指标需要**真实的用户交互**才能测量响应延迟。

![image.png](https://raw.githubusercontent.com/fw6/assets/main/toy_docs/20230813210924.png)


糟糕的 FID 主要是由**繁重的 JavaScript 执行**导致的。优化网页上 JavaScript 的解析、编译和执行方式将直接降低 FID。

1. 将长时间运行的代码拆解为更小的异步任务
2. 渐进式加载代码和功能
3. 减少级联数据获取的依赖
4. 减小客户端需要处理数据的大小
5. 延迟加载第三方代码（如ads、analysis等服务）
6. 将非用户界面操作的繁重工作，放到Web worker
7. 在关键渲染路径或首屏渲染不需要的JavaScript始终延迟加载
8. 代码分割 + 动态导入

## 优化CLS

累积布局偏移 (CLS)：核心 Web 指标中的一项指标，通过计算未在用户输入 500 毫秒内发生的布局偏移的偏移分数总和来测量内容的不稳定性。该项指标查看可视区域中可见内容的位移量以及受影响元素的位移距离。

![image.png](https://raw.githubusercontent.com/fw6/assets/main/toy_docs/20230813212345.png)

CLS 较差的最常见原因为：
- 无尺寸的图像
- 无尺寸的广告、嵌入和 iframe
- 动态注入的内容
- 导致不可见文本闪烁 (FOIT)/无样式文本闪烁 (FOUT) 的网络字体
- 在更新 DOM 之前等待网络响应的操作
