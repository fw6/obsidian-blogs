---
title: "有意思的Web API"
description: ""
pubDate: "2023-06-27 16:23"
heroImage: "https://images.unsplash.com/photo-1621839673705-6617adf9e890?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D&auto=format&fit=crop&w=640&q=80"
date created: 2023-06-27
date modified: 2023-06-27
tags:
    - notes
    - HTML
    - WebAPI
    - W3C
    - Programming
---

> The function of wisdom is to discriminate between good and evil.
> — <cite>Cicero</cite>

## TOC


## IME

- [IME | W3C](https://w3c.github.io/uievents/#ime)
- [text composition system | W3C](https://w3c.github.io/uievents/#text-composition-system)
- [Issue #180 · w3c/uievents · GitHub](https://github.com/w3c/uievents/issues/180)
- [Input method editor | MDN](https://developer.mozilla.org/en-US/docs/Glossary/Input_method_editor)
- [Understanding Composition Browser Events | Square Corner Blog](https://developer.squareup.com/blog/understanding-composition-browser-events/)

输入法编辑器是一种操作系统程序，用于将一种语言或字符集转换为另一种语言或字符集。例如输入*nihao*，键盘上没这种字符，但如果启用IME，开始打字时会出现下滑线表明IME处于活跃状态，在此过程中可以选择哪组字符与要输入的含义相匹配。按下Enter键将确认选择，将所选字符放到输入中，IME关闭。通过IME进行字符转换可输入来自任意国家/地区的母语。

## Edit Context

- [EditContext API](https://w3c.github.io/edit-context/)
![image.png](https://raw.githubusercontent.com/fw6/assets/main/toy_docs/20230626145625.png)

![image.png](https://raw.githubusercontent.com/fw6/assets/main/toy_docs/20230626145600.png)


## Background Tasks API

[Background Synchronization API - Web APIs | MDN](https://developer.mozilla.org/en-US/docs/Web/API/Background_Synchronization_API)

该API提供了一种新的方式在浏览器下载资源，如电影、音视频、软件。

当网友端要求客户下载大文件时，这通常需要用户保持与页面连接才能完成下载，如果失去连接、关闭选项卡、离开下载页面都会导致下载停止🤚。

[Background Synchronization API](https://developer.mozilla.org/en-US/docs/Web/API/Background_Synchronization_API) 使Web应用能延迟任务，一旦用户网络稳定即可在`Service Worker`中运行。但是它不能用于需要长时间运行的任务，例如
使用 Service Worker 提供了一种推迟处理直到用户连接的方法；但是它不能用于长时间运行的任务，例如下载大文件。后台同步要求服务工作人员保持活动状态直到获取完成，并且为了节省电池寿命并防止在后台发生不需要的任务，浏览器将在某个时刻终止该任务。




