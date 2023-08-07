---
title: "Debugging by Thinking"
description: ""
pubDate: "2023-08-06 11:27"
heroImage: "https://images.unsplash.com/photo-1690947332692-070bd9f2ad2e?crop=entropy&cs=srgb&fm=jpg&ixid=M3wzNjM5Nzd8MHwxfHJhbmRvbXx8fHx8fHx8fDE2OTEyOTI0NTl8&ixlib=rb-4.0.3&q=85"
date created: "2023-08-06 11:26"
date modified: "2023-08-06"
tags:
    - notes
    - Programming
---

对该分享的记录：[Debugging Software, June 2023](https://blog.isquaredsoftware.com/presentations/2023-06-debugging-js/?slideIndex=0&stepIndex=0)

## WHAT 什么是Debug？

:::tldr
debugging is the process of finding and resolving _bugs_ (defects or problems that prevent correct operation) within computer programs, software, or systems.
—— Wikipedia
:::

**Why is it broken, and how do we fix it?**

Debug是关于「为啥出问题，以及我们如何修复」的过程。

## WHY 为何Debug能力如此重要？

编程不只是写代码，还包括：
1. Planning/design
2. Communication
3. Review
4. Figuring out why something isn't working

开发者花费大量时间Debug和修复代码。

Debug如此重要，但很多开发者却对此不熟悉，为啥？
1. 学校不教
2. 大量知识来自经验
3. 边走边学的艰难的路

## HOW 如何Debug？

Debug的一些核心原则:
1. 每个问题都有根源（但可能并不容易，比如会遇到环境复杂、难以复现等问题）
2. 理解系统的预期行为很重要（系统没按照预期执行就是bug）
3. 能复现是关键（用于确认问题、以及修复后的验证）
4. 有计划的debug（不要漫无目的，一次只改一处，逐渐缩小范围）
5. 错误信息非常有用（包含where、what、和when等信息）

常用的Debug步骤：
1. 理解问题描述
2. 复现问题
   使用最小化的产生问题的示例场景收窄复现的问题（降低复杂度！）
3. 如果可能的话，找到根本问题（假设、测试、实验）
4. 确定解决问题的最佳方案
   - 尽量修复根本问题，而不是现象
   - 确定约束条件（严重性、代码要求）
5. 修复问题（添加测试或错误处理，确保不会再发生）
6. 完善文档，尽可能多地记录下来

## 一些Tips

1. 🔧善用工具
2. 📦深入底层实现（必要时翻源码，如三方库）
3. 😨别怕（逐步分解问题）
4. 🧘注意劳逸结合（坚持、专注、适当休息！）

