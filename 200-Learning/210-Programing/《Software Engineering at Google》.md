---
title: "《Software Engineering at Google》"
description: ""
pubDate: "2023-08-30 19:26"
heroImage: "https://images.unsplash.com/photo-1581091226033-d5c48150dbaa?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D&auto=format&fit=crop&w=1200&q=80"
date created: 2023-08-30
date modified: 2023-08-31
tags: 
    - notes
    - Programming
---

> Mountains cannot be surmounted except by winding paths.
> — <cite>Johann Wolfgang von Goethe</cite>

- 在线版本：[abseil / Software Engineering at Google](https://abseil.io/resources/swe-book)。
- 中文翻译：[Software Engineering at Google](https://qiangmzsx.github.io/Software-Engineering-at-Google/#/)


## TOC



## What is software engineering?

编程与软件工程的3个关键区别：
1. Time 时间
2. Scale 规模
3. Trade-offs 权衡取舍

在一个软件工程中，工程师需要更多的关注时间成本和需求变更；组织更多关注规模和效率；作为软件工程师，需要做出更复杂、风险更大的决策（往往基于时间和规模增长的不确定性的评估）。


了解时间对程序的影响的一种方法是思考“代码的预期生命周期是多少？“。对于一个只需要存活一个小时的程序，你不太可能考虑其底层库、操作系统（OS）、硬件或语言的新版本。这些短期系统实际上“只是”一个编程问题，就像在一个维度中压缩得足够扁的立方体是正方形一样。随着我们扩大时间维度，允许更长的生命周期，改变显得更加重要。在十年或更长的时间里，大多数程序依赖关系，无论是隐式的还是显式的，都可能发生变化。这一认识是我们区分软件工程和编程的根本原因。

:::note {title="技术债务"}
这也许是一个合理且简单的技术债务定义：那些”应该“做却还未完成的事（代码现状和理想代码之间的差距）
:::

软件工程任务是团队的工作。早期定义软件工程的尝试为这一观点提供了一个很好的定义：“多人开发的多版本程序”。这表明软件工程和程序设计之间的区别是时间和人的区别。团队协作带来了新的问题，但也提供了比任何单个程序员更多的潜力来产生有价值的系统。

团队组织、项目组成以及软件项目的策略和实践都支配着软件工程复杂性。我们还可以说，软件工程与编程的不同之处在于需要做出的决策的复杂性及其风险。

:::note

[Nothing is Certain Except Death, Taxes and a Short Mobile App Lifespan](https://blog.axway.com/learning-center/software-development/api-development/nothing-is-certain-except-death-taxes-and-a-short-mobile-app-lifespan-2)

:::

:::note
Hyrum’s Law: *With a sufficient number of users of an API, it does not matter what you promise in the contract: all observable behaviors of your system will be depended on by somebody.*

>有了足够数量的 API 用户，在约定中承诺什么并不重要：你的系统的所有可观察行为都将取决于某人。

:::

海勒姆定律代表了一种实践知识，即使有最好的规划、最好的工程师和可靠的代码评审实践，我们也不能假设完全遵守已发布的契约或最佳实践。作为API所有者，通过明确地接口约定，你将获得一定的灵活性和自由度，但在实践中，给定更改的复杂性和难度还取决于用户对你的API的一些可观察行为的有用程度。如果用户不能依赖这些东西，那么你的API将很容易更改。如果有足够的时间和足够的用户，即使是最无害的变更也会破坏某些东西；你对变更价值的分析必须包含调查、识别和解决这些缺陷的难度。


