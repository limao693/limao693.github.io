
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

- 现有的硬件限制了扩展能力。例如不能无线提高CPU核数。
- 通常需要停机时间。

**水平扩展**通过投入更多的机器来解决上述问题。添加新机器不需要停机，也没有机器数量的限制。它的缺点是，并非所有系统支持水平弹性伸缩，因为在设计之初没有考虑工作在集群环境下，同时设计也更为复杂。<br />![](https://cdn.nlark.com/yuque/0/2019/jpeg/250680/1550673729515-587ccc54-647c-4f27-a36f-08848fd77da0.jpeg#align=left&display=inline&height=389&originHeight=389&originWidth=400&size=0&status=done&width=400)
<a name="327fa254"></a>
### 高容错性
非分布式系统的一大安全隐患是单点故障（single point of failure, SPOF）。例如单实例数据库宕机，会产生无法估量的损失。<br />分布式系统被设计为可配置的方式，来处理故障。例如，在5节点的Kafka集群中，即使其中两个节点关闭，系统仍然继续工作。需要注意的地方是，系统容错率与系统性能成反比，即容错率越高，性能越低。
<a name="62b5133f"></a>
### 日志采集
日志采集（事务日志）仅支持附加的持久有序数据结构，即无法修改或删除已有的记录。读取顺序从左向右如图所示。<br />![](https://cdn.nlark.com/yuque/0/2019/jpeg/250680/1550675101681-613e5d8c-b7f7-4800-8c6e-f0c7a41dae65.jpeg#align=left&display=inline&height=187&originHeight=187&originWidth=396&size=0&status=done&width=396)<br />在某种程度来讲，Kafka的数据结构就是如此简单。这也是Kafka的核心，因为它提供了有序数据，而有序的数据结构提供了确定性的处理方式。<br />Kafka实际上将所有消息存储在磁盘上，并且对他们进行排序，最大化利用硬盘顺序读取速度快的优势。

- 读取和写入是整数级别O(1)（知道记录ID），与磁盘上的其他数据结构的操作复杂度为O(log N)相比，有巨大优势。
- 读取和写入是分离的。写入时不会锁定读取，反之亦然（与平衡树相反）。

这两点使得Kafka的性能有巨大优势，因为系统性能与数据量是分离的。Kafka无论处理100KB的数据量还是100TB的数据量，性能都是一样的。
<a name="b3fbd195"></a>
## 工作原理
向Kafka节点（broker）发送消息（记录）的称为程序（生产者），发送给其他程序处理消息的称为消费者。产生的消息存储在一个**topic**内，消费者订阅向topic订阅后，可以接收到对应topic的新消息。<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/250680/1550761835264-486e6737-07e1-4bbb-aae7-4e09b7888f77.png#align=left&display=inline&height=178&name=image.png&originHeight=180&originWidth=258&size=8295&status=done&width=255)<br />随着topic变得越来越大，为保持性能和可伸缩性将topic拆分成更小的分区。（例如，需要存储用户的登录名，可以将用户名按照首字母进行拆分存储在不同的分区）<br />Kafka保证分区内的所有消息都按照它们进入的顺序排序。区分特定消息的方式是通过其**偏移量** ，可将其视为普通数组索引，在一个分区的序列号随着新消息进入而递增。<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/250680/1550762596583-9e4702c2-cd98-410f-8587-2f2a17cbfaea.png#align=left&display=inline&height=182&name=image.png&originHeight=267&originWidth=416&size=19550&status=done&width=283)<br />Kafka遵循愚蠢的broker和聪明的消费者原则。意思是，Kafka不会跟踪哪一条记录已经被消费者读取并删除，而是将他们按照一定的时间（例如一天）或约定某个阈值大小来存储。消费者自身从Kafka的broker**主动拉取**新信息，并告知他们想要读取的片段。这意味着broker允许消费者按照自身的需要进行增加或减小偏移量，因此具有重新读取和重新处理信息的能力。<br />需要注意的是，消费者其实是消费者群组，包含一个或多个消费进程。为避免两个进程重复读取相同的信息，每个分区仅与每个组的一个消费者进程相关联。<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/250680/1550763797220-d9a12e85-f1df-477c-adf7-ce92ddbeb9ac.png#align=left&display=inline&height=331&name=image.png&originHeight=661&originWidth=2000&size=238955&status=done&width=1000)
<a name="efbcc1fb"></a>
## 数据持久化
Kafka将所有记录存储在磁盘，并且不会在RAM中保存任何记录。你可能会惊讶于以怎样高效的方式来做明智的选择。这背后有很多值得学习的优化知识：

1. Kafka有将成组的消息合并的协议。允许网络请求将消息组合在一起减少网络开销，因此服务器一次性保留大量消息，消费者一次获取巨大的线性消息块。
1. 磁盘顺序I/O读取速度很快。现代磁盘速度慢的原因是由于大量的磁盘寻道，但是在大型线性读写操作时不是问题。
1. 线性操作由OS进一步优化，通过**预读取**（预读取多倍数据块）和**后写入**（将小的逻辑写入操作合并为组后再进行物理写入操作）技术。
1. 现代OS利用RAM模拟磁盘缓存，这种技术成为pagecache。
1. 由于kafka在整个流程（生产者--->代理--->消费者）中以未经修改的标准化二进制格式存储消息，因此它可以使用**零拷贝**（zero-copy）优化。操作系统将数据从pagecache直接复制到套接字，有效地完全绕过了Kafka代理应用程序。

所有这些优化都使Kafka能够从接近网络的速度传输消息。
<a name="8b75be39"></a>
## 数据分发和复制
在这个章节，我们讨论Kafka如何实现容错以及它如何在节点之间分配数据。
<a name="82a6c7de"></a>
### 数据复制
分区数据在多个代理（broker）中复制，以便在其中一个代理挂掉时候，依然能够保存数据。<br />在任何时候，一个代理（broker）“拥有”其中一个分区，节点通过这个分区进行读写操作。因此，此分区被称为**分区领导者**（partition leader）。将接收到的数据复制给其他_**N**_个其他代理，称为**跟随者**（followers）。跟随者同样存储数据，并且随时准备着当leader挂掉时，取而代之。<br />这样的配置有助于保证任何成功发布的消息都不会丢失。通过更改复制因子，可以根据数据的重要性来交换性能以获得更强的持久性保证。<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/250680/1551192978691-57bdf14e-90a0-47df-8aeb-60224ba6254c.png#align=left&display=inline&height=765&name=image.png&originHeight=1531&originWidth=2600&size=422840&status=done&width=1300)<br />在其中一个leader挂掉时，其他follower会竞选上岗，具体算法可以参考：<br />[_How does a producer/consumer know who the leader of a partition is?_](https://community.hortonworks.com/questions/149532/how-producer-and-consumer-identify-the-leader-in-k.html)<br />作为生产者/消费者，对一个分区进行读取时，首先需要知道对应分区的leader。这个信息需要存储在可以被访问到的地方，Kafka使用Zookeeper进行存储这些元数据。
<a name="f52343ab"></a>
### 何为Zookeeper
Zookeeper是一个分布式键值存储结构。它针对读取进行了高度优化，但写入速度较慢。常应用于存储元数据和处理集群的核心机制（心跳包、分发更新配置等）。<br />它允许服务（Kafka的broker）的客户订阅通知，并且能在Zookeeper发生变动的时候发送给客户消息。这也是为什么brokers能够感知分区的leader发生变动。Zookeeper同时也具有成熟的容错性，或者说，Kafka很大程度上依赖Zookeeper的高容错性。<br />Zookeeper用于存储所有类型的元数据，包括但不限于：

- 消费者群组中每个分区的偏移量（尽管现在的客户端在单独的Kafka主题Topic内存储偏移量）
- ACL（访问控制列表），用于限制访问/授权
- 生产者和消费者配额，包括每秒最大信息量
- 存储分区leader和健康状态
<a name="c577e2f9"></a>
### 如何区分分区的领导者
在以往版本中，生产者和消费者经常直接连接并与Zookeeper交谈以获取此（和其他）信息。 目前Kafka已经弃用这种耦合，从0.8和0.9版本开始，客户端直接从Kafka的brokers那里获取元数据信息，他们自己与Zookeeper交谈。

![image.png](https://cdn.nlark.com/yuque/0/2019/png/250680/1551274509585-90480e7a-f43d-4959-9703-a908c3983f5f.png#align=left&display=inline&height=1076&name=image.png&originHeight=2152&originWidth=2000&size=462646&status=done&width=1000)
<a name="3f4a00cf"></a>
## 流
在Kafka中，流处理器是从输入的Topic中连续读取数据流，并对数据进行一些处理生成数据流以生成主题的任何（或外部服务、数据库、垃圾箱等）内容。<br />对于一些简单的消息，可能使用消费者或生产者的API接口直接处理即可，但是涉及到复杂的消息流（例如，多条数据流联合）处理的情况，Kafka提供一个集成的[Stream API](https://kafka.apache.org/documentation/streams/)库。<br />此API应用于自己的代码块中，而不是直接在代理（broker）上运行。它与消费者API类似，可以帮助你在多个应用（类似多个消费者）上扩展流处理工作。
<a name="d34be05f"></a>
### 无状态处理
流的无状态处理是确定性的，不需要依赖任何外部的处理方式。对于任何给定的数据，将始终生成与其他内容无关的相同输入。<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/250680/1551280756166-0643b4f0-57aa-483b-b51f-ff4d42fc174e.png#align=left&display=inline&height=381&name=image.png&originHeight=762&originWidth=2600&size=233348&status=done&width=1300)
<a name="6c8a2395"></a>
### 流式表的双重性
首先要认识到流和表是相同的含义。流，可以解释为表，反之亦然。
<a name="e3e24155"></a>
### 流作为表
流可以看做对数据进行一系列的更新，因此最终结果作为表进行聚合。这种技术成为事件采集（Event sourcing）。<br />如果你了解如何实现同步数据库的复制，你会知道它是通过所谓的流复制（**Streaming replication**），每次表格中的变动都会发送到副本服务器。事件采集的另外一个例子是，区块链分类账，它也是进行一系列变化。<br />Kafka的数据流可以用相同的方式解释，即可以认为是积累到最终的状态的事件。此类流聚合保存在本地RocksDB中，称为**KTable**。<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/250680/1551281577256-6c7efae0-3e86-4379-9b28-3a6870bb826a.png#align=left&display=inline&height=343&name=image.png&originHeight=686&originWidth=2000&size=219233&status=done&width=1000)
<a name="d10a4cc4"></a>
### 表作为流
可以将表视为流中每个键的最新值的快照。 以相同的方式，流记录可以生成表，表更新可以生成更改日志流。

![image.png](https://cdn.nlark.com/yuque/0/2019/png/250680/1551281769435-7f51d3f6-30f3-4ae8-9018-19d0ebfb11a9.png#align=left&display=inline&height=646&name=image.png&originHeight=1291&originWidth=1600&size=350243&status=done&width=800)
<a name="096aebe0"></a>
### 有状态处理
一些简单的操作，例如`map()`或者`filter()`都是无状态的，不需要额外保存有关处理的任何数据。但是，在现实生活中，大部分操作都是有状态的（例如`count()`），因此需要保存当前积累的值。<br />假如在流处理器上维护这些状态，流处理器可能会宕机，导致状态丢失。那么应当在哪里保存状态值才能容错呢？<br />一种最简单的方式是简单地将所有状态存储在远程数据库中，并通过网络连接到该数据库。这样做的问题是，没有数据的位置和产生大量的网络交互损耗，这两者都会显着减慢您的应用程序。 一个更微妙但重要的问题是您的流处理作业的正常运行时间将紧密耦合到远程数据库，并且作业将不会自包含_（数据库中的数据库与另一个团队的更改可能会破坏您的处理）_ 。<br />回忆下表和流的二元性。运行我们将流转化为与我们处理位于同一位置的表。它还能提供一种处理容错的机制，即在Kafka的Broker中存储流。<br />数据流处理器能够在本地表（即，RocksDB）存储状态，该表将从输入流（可能实在某些任意变换之后）更新。当进程失败时，可以通过重新请求流来恢复其数据。<br />你也可以使用一个远程数据库作为流的生产者，用于在本地重建表进行高效的广播更改日志。<br />
![image.png](https://cdn.nlark.com/yuque/0/2019/png/250680/1551282823033-1e7dcc9e-c2a2-4130-b972-b1c1776bd0f1.png#align=left&display=inline&height=729&name=image.png&originHeight=1458&originWidth=2000&size=234952&status=done&width=1000)
<a name="KSQL"></a>
## KSQL
通常，使用Kafka只能使用JVM语言编写刘处理，因为这是Kafka唯一的官方Streams API客户端。<br />
![image.png](https://cdn.nlark.com/yuque/0/2019/png/250680/1551799835544-909daa5c-d2ad-4f30-9060-496ba185feaf.png#align=left&display=inline&height=772&name=image.png&originHeight=1543&originWidth=1600&size=165449&status=done&width=800)<br />2018.04的Kafka发布**KSQL**，一种可以使用类SQL语言来编写简单流媒体工作的工具。<br />通过设置KSQL服务器，并且通过CLI方式进行交互以此来管理处理。它使用相同的抽象（KStream和KTable），保证了StreamS API的相同有点（可伸缩性、容错性）和更加简便的方式处理工作流。<br />这个特性虽然不被人经常提起，但经过实践对于测试更有用，甚至运行开发之外的人（例如，产品所有者）使用流处理。
<a name="f525672f"></a>
### 流的可选择性
Kafka的流兼具了力量和简约的完美结合。可是说是市场上处理流工作的最佳工具，与其他流处理工具（Storm、Samza、Spark和Wallaroo）相比，Kafka更容易与其他工具结合。<br />大多数其他流处理的框架的问题在于它们运行和部署的复杂性。例如Spark这样的处理框架需要以下几点：

1. 在一组计算机上控制大量的作业，并在集群上有效的分配。
1. 为此，必须动态打包你的程序并将其部署在它需要执行的节点（以及配置、库等）。

为此，要处理以上问题，使得框架尤为复杂。它们需要控制很多方面：部署、配置、监控和打包。<br />Kafka流能够允许你在你需要时，提出自己的部署策略，例如Kubernetes、Mesos、Nomad、Docker Swarm或者其他方式。<br />Kafka Streams的基本目的是使所有应用程序能够进行流处理，而无需运行和维护另一个操作复杂的集群。 唯一潜在的缺点是它与卡夫卡紧密结合，但在现代世界中，大多数（如果不是全部）实时处理由卡夫卡提供动力可能不是一个很大的劣势。
<a name="c5329d3d"></a>
## 何时启用Kafka
正如我们已经介绍的，Kafka允许通过集中式介质获取大量消息并且存储他们，并不担心性能或数据丢失等问题。<br />这意味着非常适合用在系统框架的核心，充当连接不同程序的中间媒介。Kafka能够成为事件驱动架构的中心部分，是您能够真正的将应用程序间解耦。

![image.png](https://cdn.nlark.com/yuque/0/2019/png/250680/1551888348371-8ade2e7b-bab2-4c43-9172-231a08b074c0.png#align=left&display=inline&height=337&name=image.png&originHeight=674&originWidth=900&size=432210&status=done&width=450)<br />Kafka能够非常轻松的分离不同（微）服务之间的通信。使用Streams API，现在可以更容易的编写业务逻辑，从而丰富Kafka主题数据以便提供服务。
<a name="25f9c7fa"></a>
## 总结
Apache Kafka作为分布式流平台，每天可以处理数以万亿计的事件。Kafka提供低延迟、高吞吐量、高容错和订阅式流水线，同时能够流式处理事件。<br />我们回顾了它的基本语义（生产者、代理、消费者和Topic），了解了它的一些优化（page cache），通过复制数据了解了它的容错能力，并且介绍了它不断增长的强大流功能。


