
---
title: Kafka - - 快速入门指南
date: 2019-02-01 18:41:49 +0800
tags: [珠峰翻译计划,Kafka]
categories: 
---
<a name="e05dce83"></a>
## 简介

Kafka起源于2011年LikedIn的开源项目，并不断发展。如今它是一个完整的平台，允许您冗余地存储巨大的数据量，拥有一个具有巨大吞吐量（数百万/秒）的消息总线，同时可实现实时流处理。

Kafka具备的特性有：**分布式、水平扩展、高容错和日志管理**。接下来将逐个介绍特性。

<a name="00ea564d"></a>
### 分布式

分布式系统的显著特点是将任务分配到多个机器处理，即组成的集群对外暴露的形式仍为单一节点。Kafka的分布特性在于在不同节点（代理）存储、接收和发送消息。

> 分布式系统的介绍可以参阅：[A Thorough Introduction to Distributed Systems](https://medium.freecodecamp.org/a-thorough-introduction-to-distributed-systems-3b91562c9b3c)


分布式系统的优势主要体现在：高可扩展性、容错性。

<a name="32a24e98"></a>
### 水平扩展

首先解释、定义垂直扩展。例如，数据库服务器过载，最直接的解决办法是添加资源（CPU、RAM和SSD）。垂直扩展的两大缺点是：

* 现有的硬件限制了扩展能力。例如不能无线提高CPU核数。
* 通常需要停机时间。

**水平扩展**通过投入更多的机器来解决上述问题。添加新机器不需要停机，也没有机器数量的限制。它的缺点是，并非所有系统支持水平弹性伸缩，因为在设计之初没有考虑工作在集群环境下，同时设计也更为复杂。<br />![](https://cdn.nlark.com/yuque/0/2019/jpeg/250680/1550673729515-587ccc54-647c-4f27-a36f-08848fd77da0.jpeg#align=left&display=inline&height=389&linkTarget=_blank&originHeight=389&originWidth=400&size=0&status=done&width=400)
<a name="327fa254"></a>
### 高容错性
非分布式系统的一大安全隐患是单点故障（single point of failure, SPOF）。例如单实例数据库宕机，会产生无法估量的损失。<br />分布式系统被设计为可配置的方式，来处理故障。例如，在5节点的Kafka集群中，即使其中两个节点关闭，系统仍然继续工作。需要注意的地方是，系统容错率与系统性能成反比，即容错率越高，性能越低。
<a name="62b5133f"></a>
### 日志采集
日志采集（事务日志）仅支持附加的持久有序数据结构，即无法修改或删除已有的记录。读取顺序从左向右如图所示。<br />![](https://cdn.nlark.com/yuque/0/2019/jpeg/250680/1550675101681-613e5d8c-b7f7-4800-8c6e-f0c7a41dae65.jpeg#align=left&display=inline&height=187&linkTarget=_blank&originHeight=187&originWidth=396&size=0&status=done&width=396)<br />在某种程度来讲，Kafka的数据结构就是如此简单。这也是Kafka的核心，因为它提供了有序数据，而有序的数据结构提供了确定性的处理方式。<br />Kafka实际上将所有消息存储在磁盘上，并且对他们进行排序，最大化利用硬盘顺序读取速度快的优势。
* 读取和写入是整数级别O(1)（知道记录ID），与磁盘上的其他数据结构的操作复杂度为O(log N)相比，有巨大优势。
* 读取和写入是分离的。写入时不会锁定读取，反之亦然（与平衡树相反）。

这两点使得Kafka的性能有巨大优势，因为系统性能与数据量是分离的。Kafka无论处理100KB的数据量还是100TB的数据量，性能都是一样的。
<a name="b3fbd195"></a>
## 工作原理
向Kafka节点（broker）发送消息（记录）的称为程序（生产者），发送给其他程序处理消息的称为消费者。产生的消息存储在一个**topic**内，消费者订阅向topic订阅后，可以接收到对应topic的新消息。<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/250680/1550761835264-486e6737-07e1-4bbb-aae7-4e09b7888f77.png#align=left&display=inline&height=178&linkTarget=_blank&name=image.png&originHeight=180&originWidth=258&size=8295&status=done&width=255)<br />随着topic变得越来越大，为保持性能和可伸缩性将topic拆分成更小的分区。（例如，需要存储用户的登录名，可以将用户名按照首字母进行拆分存储在不同的分区）<br />Kafka保证分区内的所有消息都按照它们进入的顺序排序。区分特定消息的方式是通过其**偏移量** ，可将其视为普通数组索引，在一个分区的序列号随着新消息进入而递增。<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/250680/1550762596583-9e4702c2-cd98-410f-8587-2f2a17cbfaea.png#align=left&display=inline&height=182&linkTarget=_blank&name=image.png&originHeight=267&originWidth=416&size=19550&status=done&width=283)<br />Kafka遵循愚蠢的broker和聪明的消费者原则。意思是，Kafka不会跟踪哪一条记录已经被消费者读取并删除，而是将他们按照一定的时间（例如一天）或约定某个阈值大小来存储。消费者自身从Kafka的broker**主动拉取**新信息，并告知他们想要读取的片段。这意味着broker允许消费者按照自身的需要进行增加或减小偏移量，因此具有重新读取和重新处理信息的能力。<br />需要注意的是，消费者其实是消费者群组，包含一个或多个消费进程。为避免两个进程重复读取相同的信息，每个分区仅与每个组的一个消费者进程相关联。<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/250680/1550763797220-d9a12e85-f1df-477c-adf7-ce92ddbeb9ac.png#align=left&display=inline&height=331&linkTarget=_blank&name=image.png&originHeight=661&originWidth=2000&size=238955&status=done&width=1000)
<a name="efbcc1fb"></a>
## 数据持久化
Kafka将所有记录存储在磁盘，并且不会在RAM中保存任何记录。你可能会惊讶于以怎样高效的方式来做明智的选择。这背后有很多值得学习的优化知识：
1. Kafka有将成组的消息合并的协议。允许网络请求将消息组合在一起减少网络开销，因此服务器一次性保留大量消息，消费者一次获取巨大的线性消息块。
1. 磁盘顺序I/O读取速度很快。现代磁盘速度慢的原因是由于大量的磁盘寻道，但是在大型线性读写操作时不是问题。
1. 线性操作由OS进一步优化，通过**预读取**后写入**（将小的逻辑写入操作合并为组后再进行物理写入操作）技术。
1. 现代OS利用RAM模拟磁盘缓存，这种技术成为pagecache。
1. 由于kafka在整个流程（生产者--->代理--->消费者）中以未经修改的标准化二进制格式存储消息，因此它可以使用**零拷贝**（zero-copy）优化。操作系统将数据从pagecache直接复制到套接字，有效地完全绕过了Kafka代理应用程序。

所有这些优化都使Kafka能够从接近网络的速度传输消息。
<a name="8b75be39"></a>
## 数据分发和复制
在这个章节，我们讨论Kafka如何实现容错以及它如何在节点之间分配数据。
<a name="82a6c7de"></a>
### 数据复制
分区数据在多个代理（broker）中复制，以便在其中一个代理挂掉时候，依然能够保存数据。<br />在任何时候，一个代理（broker）“拥有”其中一个分区，节点通过这个分区进行读写操作。因此，此分区被称为**分区领导者**（partition leader）。将接收到的数据复制给其他_**N**_个其他代理，称为**跟随者**（followers）。跟随者同样存储数据，并且随时准备着当leader挂掉时，取而代之。<br />这样的配置有助于保证任何成功发布的消息都不会丢失。通过更改复制因子，可以根据数据的重要性来交换性能以获得更强的持久性保证。<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/250680/1551192978691-57bdf14e-90a0-47df-8aeb-60224ba6254c.png#align=left&display=inline&height=765&linkTarget=_blank&name=image.png&originHeight=1531&originWidth=2600&size=422840&status=done&width=1300)<br />在其中一个leader挂掉时，其他follower会竞选上岗，具体算法可以参考：<br />[_How does a producer/consumer know who the leader of a partition is?_](https://community.hortonworks.com/questions/149532/how-producer-and-consumer-identify-the-leader-in-k.html)<br />作为生产者/消费者，对一个分区进行读取时，首先需要知道对应分区的leader。这个信息需要存储在可以被访问到的地方，Kafka使用Zookeeper进行存储这些元数据。
<a name="f52343ab"></a>
### 何为Zookeeper
Zookeeper是一个分布式键值存储结构。它针对读取进行了高度优化，但写入速度较慢。常应用于存储元数据和处理集群的核心机制（心跳包、分发更新配置等）。


