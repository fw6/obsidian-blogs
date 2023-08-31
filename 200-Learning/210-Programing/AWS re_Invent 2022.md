
---
title: "Dr. Werner in AWS re:Invent 2022"
description: "re:Invent 2022中，Dr. Werner的主题演讲"
pubDate: "2023-08-31 09:00"
heroImage: "https://raw.githubusercontent.com/fw6/assets/main/toy_docs/20230831090633.png"
date created: "2023-08-31 08:59"
date modified: "2023-08-31"
tags:
    - notes
---

- [AWS re:Invent 2022 - Keynote with Dr. Werner Vogels - YouTube](https://www.youtube.com/watch?v=RfvL_423a-I)

同步是在异步世界创造的幻觉

:::note
 The universe is incredibly agile, fault tolerant, resilient and robust. It is also asynchronous.
:::

操作系统的本质是管理所有设备，并在此基础上添加一些抽象。那些设备如何工作的呢？有一个设备驱动程序，来确保所有磁盘看起来相同。实际情况是设备与操作系统之间的交互是基于时钟中断、事件驱动的系统。如果需要向磁盘中放入数据，可以将其放入共享内存缓冲区，将一些内容写入寄存器中，也可以在写入完成将其输入回来。因此，操作系统中的底层进程，是纯粹异步的。

>时钟是生成中断的最重要设备，它驱动调度程序，并驱动实际执行的进程。

Windows NT可能是世界上第一个将异步通信或与设备交互作为内核第一原则的操作系统。Linux直到2000年代初才实现异步（实际知道2019年，io_uring才真正提供了异步交互）。

为什么需要松散耦合的系统？
1. 依赖很少
2. 故障隔离
3. 真正的可进化架构
可以轻松的改造您的架构。

:::note
Amazon's Distributed Computing Manifesto, 1998.
Read at [All Things Distributed](https://www.allthingsdistributed.com/)
:::

工作流程要解决的问题，在松散耦合的组件中构建应用程序，如何创造、组合、进化？

如何构建工作流程？
- 需要拥有如下平台：
  1. Sequence
  2. Retry
  3. Error handling
  4. Parallel
  5. Routing based on data
  6. Concurrent/iterative
- AWS提供的两项服务：
  1. Step function
  2. Event bridge

世界是事件驱动的、异步的。

事件驱动的系统天然松散耦合。  

![image.png](https://raw.githubusercontent.com/fw6/assets/main/toy_docs/20230831095924.png)

:::note
Gall‘s low: All complex systems that work evolved from simpler systems that worked.
:::

构建可进化系统的最佳方法是关注事件驱动的架构。

事件总线是网络🕸️中的🕷️，负责路由、协调和调度。

事件是可组合的。使用管道将不同服务组合起来。