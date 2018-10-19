---
layout: post
title: 分布式系统中的通讯模型
categories: [distributed-programming]
description: distributed system
keywords: distributed system
---

人们在谈及分布式系统的理论时，首先要明确其前置的`通讯模型`（有的地方也称为`时间模型`）的约定。比较常见的有`同步网络模型`、`异步网络模型`和`部分同步网络模型`。

# 同步网络模型

在`同步网络模型`中：
1. 进程间的消息通讯、传输延时有界。认为通讯耗时是有确定范围的；
2. 每个进程的处理速度是确定的。我们可以确切知道进程中每步算法的耗时。

因此我们可以推导出：
* 每个进程间的时钟是同步的，因为前面的定义**1**、**2**，所以我们可以使用通讯来同步各个机器上的时钟，使得各个机器的时钟误差在`ΔT`内。
* 如果一个请求超过的应答超过$$RTT + T_{请求处理耗时}$$，则可以判定对端异常。

`同步网络模型`使用起来最简单，是理想的网络模型，但是它就跟物理实验中的"光滑平面"一样，在实际环境中并不存在。

# 异步网络模型

在`异步网络模型`中，我们引用`FLP 不可能性`[^2]中的定义：
1. 进程间的消息通讯、传输延时没有上界；
2. 每个进程的处理速度是不确定的；
3. 每个机器上没有同步时钟（即不存在`原子钟`这种东西），时间流逝速度可能也不同。

因此我们可以推导出：
* 各个机器上的时钟没有可参考性，因为**3**每个机器不能自发保持时间的一致性，并且因为**1**、**2**，机器时间也无法同步时钟在一个有界的误差之内。所以依赖超时机制的算法并不可用。
* 因为**1**、**2**，当一个请求在本地时钟上超时后，我们无法判断这个请求是否是因为对端异常造成的。这是一个经典的`两军问题`[^3]场景，故，我们无法对其他实例进行故障探测。

`异步网络模型`是一个最理想的"最差"网络模型，但是其复杂度又远超我们实际情形。

# 部分同步网络模型

我们现实中遇见的网络模型通常介于`同步网络模型`和`异步网络模型`两者之间。因为在我们所知的大部分系统，**在大部分时间内**：
* 进程间的消息通讯、传输延时是有上界的；只有在网络过载、网络分区故障时，才没有上界；
* 每个进程的处理速度是确定的；只有在发生[`GC`](https://en.wikipedia.org/wiki/Garbage_collection_(computer_science))、磁盘IO阻塞等异常情况时，每个进程的处理速度才不可确定。
* 每个机器上的时钟我们可以认为是基本同步的，比如我们可以使用[`NTP`](https://zh.wikipedia.org/zh-hans/網路時間協定)来同步机器时间，而且多数机器上有独立的[时钟芯片](https://zh.wikipedia.org/wiki/實時時鐘)，我们也可以粗略认为各个机器上时间流逝的速度是相同的。但是严格要求时序的系统除外。

# 结论
在`FLP 不可能性`[^2]中已经明确给我们指明了分布式系统中的矛盾点，我们需要结合`部分同步网络模型`与`异步网络模型`的约束条件变化，以及我们的业务需求，去做出具体的取舍。比如在[分布式编程中的故障探测](https://lrita.github.io/2017/10/30/failure-detect-in-distributed-programming/)就给出了`故障探测`需要做出的取舍。

# 参考
[^1]: [WHAT_WE_TALK_ABOUT_WHEN_WE_TALK_ABOUT_DISTRIBUTED_SYSTEMS](http://alvaro-videla.com/2015/12/learning-about-distributed-systems.html)
[^2]: [FLP_不可能性](/images/posts/distribution/impossibility-of-distributed-consensus-with-one-faulty-process.pdf)
[^3]: [两军问题](https://baike.baidu.com/item/两军问题/20124353)