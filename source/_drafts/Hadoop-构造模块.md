---
title: Hadoop 构造模块
tags:
- Hadoop
---

在简单了解 Hadoop 概念后，需要对 Hadoop 有更为深入的理解。

[img]

此文将会对 Hadoop 构造做详细的介绍。

## NameNode 名字节点
Hadoop 在分布式计算与分布式存储中都采用了主/从(master/slave)结构。分布式存储系统称为 HDFS。NameNode 位于 HDFS 的 master 端，指导 slave 端的 DataNode 执行底层的 I/O 任务。

**NameNode 跟踪文件如何被分割成文件块，而这些块又被那些节点存储，以及分布式文件系统的整体运行状态是否正常。**

运行 NameNode 消耗大量的内存和I/O资源。因此，为了减轻机器的负载，驻留 NameNode 的服务器通常不会存储用户数据或者执行MapReduce程序的计算任务。这意味着 NameNode 服务器不会同时是 DataNode 或者 TaskTracker。

不过 NameNode 的重要性也导致一个负面影响—— Hadoop 集群的单点失效问题。对于任何其他守护进程，如果所驻留的节点发生软件硬件失效， Hadoop 集群仍能平稳运行，但此不适用于 NameNode。

## DataNode 数据节点
**每个集群上的从节点都会驻留一个DataNode守护进程，来执行分布式文件系统的繁重工作——将HDFS数据块读取或者写入到本地文件系统的实际文件中。**当希望对HDFS文件进行读写时，文件被分割为多个块，由NameNode告知客户端每个数据驻留在那个DataNode。客户端直接与DataNode守护进程通信，来处理与数据块相对应的本地文件。而后，DataNode会与其他DataNode进行通信，复制这些数据块以实现冗余。

NameNode/DataNode 在 HDFS 中的交互。NameNode跟踪文件的元数据——描述系统中所包含的文件以及每个文件如何被分割为数据块。DataNode提供数据块的备份存储，并持续不断的向NameNode报告，以保持元数据为最新状态NameNode/DataNode 在 HDFS 中的交互。NameNode跟踪文件的元数据——描述系统中所包含的文件以及每个文件如何被分割为数据块。DataNode提供数据块的备份存储，并持续不断的向NameNode报告，以保持元数据为最新状态。

[img]

上图可看出，data1与data2分别被部署在多个DataNode节点上，每个数据块都有多个副本。如果一个DataNode崩溃或无法通过网络访问时，仍能通过其他节点读取数据。

DataNode不断向NameNode发送报告，初始化时，每个DataNode将当前存储数据块告知NameNode。初始化映射完成后，DataNode仍会不断更新NameNode，提供本地修改信息，同时接收指令创建、移动或删除本地磁盘上的数据块。

## Secondary NameNode 次名字节点
**Secondary NameNode（SNN）是一个用于监测HDFS集群状态的辅助守护进程。**像NameNode一样每个集群有一个SNN，它通常独占一台服务器，该服务器不会运行其他的DataNode或TaskTracker守护进程。SNN与NameNode的不同在于它不接收或记录任何HDFS的实时变化。相反它与NameNode通信，根据集群所配置的时间间隔获取HDFS元数据的快照。

NameNode是Hadoop集群的单一故障点，而SNN的快照可以有助于减少停机的时间，并降低数据丢失的风险。当NameNode失效时，需要人工干预，即手动的重新配置集群，将SNN用作主要的NameNode。

## JobTracker 作业跟踪节点
JobTracker守护进程是应用程和Hadoop之间的纽带。一旦提交代码到集群上，JobTracker就会确定执行计划，包括确定处理哪些文件，为不同的任务分配节点以及监控所有任务的运行。如果任务执行失败，JobTracker将自动重启任务，但所分配的节点可能会不同，同时受预定义的重试次数限制。

**每个Hadoop只有一个JobTracker守护进程。它通常运行在集群的主节点上。**

## TaskTracker 任务跟踪节点
与存储进程一样，计算的守护进程也遵从主/从架构：**JobTracker作为主节点，监测MapReduce作业的整个执行过程，同时，TaskTracker管理各个任务在每个从节点的执行情况。**如下图：

[img]

Jobtracker和TaskTracker的交互。当客户端调用JobTracker来启动一个数据处理数据时，JobTracker会将工作切分，并分配不同的map和reduce任务到集群中的每个TaskTracker上Jobtracker和TaskTracker的交互。当客户端调用JobTracker来启动一个数据处理数据时，JobTracker会将工作切分，并分配不同的map和reduce任务到集群中的每个TaskTracker上。

**每个TaskTracker负责执行有JobTracker分配的单项任务。**虽然每个从节点只有一个TaskTracker，但每个TaskTracker可以生成多个JVM来并行地处理许多map或reduce任务。

**TaskTracker的一个职责是持续不间断地与JobTracker通信。**如果JobTracker在指定时间内没有收到来自TaskTracker的“心跳”，它会假定TaskTracker已经崩溃，进而重新提交相应的任务到集群的其他节点中。

[img]

一个典型的Hadoop集群拓扑图。这是一个主/从架构，其中NameNode和JobTracker为Master端，DataNode和TaskTracker为Slave端