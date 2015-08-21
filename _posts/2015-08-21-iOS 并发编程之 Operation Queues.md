---
layout: post
title: iOS 并发编程之 Operation Queues（上)
comments: true
category: 技术

---

现如今移动设备也早已经进入了多核心 CPU 时代，并且随着时间的推移，CPU 的核心数只会增加不会减少。而作为软件开发者，我们需要做的就是尽可能地提高应用的并发性，来充分利用这些多核心 CPU 的性能。在 iOS 开发中，我们主要可以通过 Operation Queues、Dispatch Queues 和 Dispatch Sources 来提高应用的并发性。本文将主要介绍 Operation Queues 的相关知识，另外两个属于 Grand Central Dispatch（以下正文简称 GCD ）的范畴，将会在后续的文章中进行介绍。

由于本文涉及的内容较多，所以建议读者先提前了解一下本文的目录结构，以便对本文有一个宏观的认识：

- 基本概念
	* 术语
	* 串行 vs. 并发
	* 同步 vs. 异步
	* 队列 vs. 线程
- iOS 的并发编程模型

- Operation Queues vs. Grand Central Dispatch (GCD)

- 关于 Operation 对象
	* 并发 vs. 非并发 Operation
	* 创建 NSInvocationOperation 对象
	* 创建 NSBlockOperation 对象
- 自定义 Operation 对象
	* 执行主任务
	* 响应取消事件
	* 配置并发执行的 Operation
	* 维护 KVO 通知
- 定制 Operation 对象的执行行为
	* 配置依赖关系
	* 修改 Operation 在队列中的优先级
	* 修改 Operation 执行任务线程的优先级
	* 设置 Completion Block
- 执行 Operation 对象
	* 添加 Operation 到 Operation Queue 中
	* 手动执行 Operation
	* 取消 Operation
	* 等待 Operation 执行完成
	* 暂停和恢复 Operation Queue