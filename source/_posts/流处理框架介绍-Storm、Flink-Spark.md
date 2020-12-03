---
title: '流处理框架介绍: Storm、Spark & Flink'
tags: ['BigData', 'Stream Progressing']
categories: ['Concepts']
date: 2020-11-23 17:03:00
---
## 前言

与传统的批处理（Batch processing）相比，流处理（Streaming processing）处理的是实时的持续数据流，也被称为无界数据集（unbounded datasets），亦即能够持续增长的、不可预测的无限数据集。而批处理处理的是有界数据集（bounded datasets），有界数据集是有限的不变的，存在开始和结束，也被称之为历史数据集（historic datasets）。

通常，为了应对高速流动的无界数据流，流处理对于处理效率要求更高，内存占用要求更低，与之相对的，相较批处理对于错误的容忍度要更高。

本文介绍了 Apache Storm、Apache Flink 和 Spark Streaming 三种常用流处理框架，主要包含它们各自的拓扑结构和运行时架构，最后还对流处理框架的演进和流批一体化的趋势做了简要介绍。

## Apache Storm

[Apache Storm](https://storm.apache.org/index.html) 是一个分布式实时计算框架，主要使用 Clojure 和 Java 语言编写，目前最新版本 2.2.0。在 Storm 中，数据流被抽象为 tuples，由数据和 ID 标识符组成。Storm 的拓扑结构（Topology）是一个有向无环图，由**输入节点Spouts**、**处理节点Bolts**和**代表数据流的边**三部分组成，如下图所示：

![7f120eb6c1e92b4775e84d196100e3f5.png](1.png)

Spouts 是整个拓扑结构的入口点，负责将输入数据流转换为 tuples，送至 Bolts 进行处理；Bolts 负责处理输入流并转换为输出流，它维护了处理逻辑，能够对 tuples 执行过滤、映射、聚合等函数式操作，还能与数据库进行交互。

Storm 的运行时架构与 Hadoop 类似，也是经典的主从模式（master-slave）。Storm 中的主节点（master node）运行一个叫做 **Nimbus** 的程序，由其负责资源分配和任务调度，用户定义的 Topology 会被提交到 Nimbus 上；从节点（worker node）运行 **Supervisor** 程序，负责执行 Nimbus 分配的任务，其上可以运行一个或多个工作进程（worker process），每个工作进程执行 Topology 的一个子集。Nimbus 不能直接与 Supervisor 进行交互，两者需要通过 **ZooKeeper** 进行协作，ZooKeeper 保存调度信息、心跳信息、集群状态和配置信息。

![aa1882c89b70544081866ade75563064.png](2.png)

类似于 Hadoop 中的 MapReduce 架构，每个 Spouts 和 Bolts 也可以设置**并行度**（parallelism）。通过设置并行度可指定一个 Worker 运行多个 Executor，所以实际上 Executor 才是运行 Spouts 或者 Bolts 组件的最小单元。

**数据一致性**方面，Storm 基于 ACK 确认机制，可以确保每个 Tuple 至少被执行一次（**at-least-once**）。但现如今 Storm 官方提供了一个在 Storm 之上的更高层级的抽象：[Trident](http://storm.apache.org/releases/current/Trident-tutorial.html)，可以确保每个 Tuple 有且只有一次被执行（exactly-once），代价是增大了数据处理的延迟。Trident 的示例 API 如下所示：

```java
TridentTopology topology = new TridentTopology();        
TridentState wordCounts =
     topology.newStream("spout1", spout)
       .each(new Fields("sentence"), new Split(), new Fields("word"))
       .groupBy(new Fields("word"))
       .persistentAggregate(new MemoryMapState.Factory(), new Count(), new Fields("count"))                
       .parallelismHint(6);
```

## Spark Streaming

[Apache Spark](https://spark.apache.org/) 于 2009 年诞生于加州大学伯克利分校，2013 年被捐献给 Apache 基金会。Spark 的初衷是改良 Hadoop 的 MapReduce 编程模型和执行速度，它提供了更加方便易用的接口，提供 Java、Scala、Python 和 R 四种语言的 API，支持 SQL、机器学习和图计算，覆盖了绝大多数大数据计算的场景。Spark 由 Java 和 Scala 编写，目前最新版本 3.0.1。

Spark Streaming 是 Spark 框架中的核心组件之一，提供流处理功能。但 Spark Streaming 并不支持严格意义的实时流处理，它按照预设的时间间隔将流数据累积，对这个时间间隔上的数据做批处理。所以实际上 Spark Streaming 是一种**微批处理（micro-batch）**框架。

Spark 提出了弹性分布式数据集 **RDD**（resilient distributed dataset）的概念，它是 Spark 中最基本的数据抽象，它代表一个不可变、只读的，被分区的数据集，你可以像操作本地集合一样操作 RDD。而在 RDD 之上，Spark Streaming 包装了称为离散流 **DStream**（discretized stream）的高级抽象，底层由一系列连续的 RDD 组成。

![557ca0d5fa2c990b57e8d1a8a625df02.png](3.png)

Spark 架构也是经典的 master-slave 架构。主节点上运行着 Driver，用户从客户端提交应用 Jar 包，首先会构建一个 Spark Application，并初始化程序入口 **SparkContext**（要运行 Spark Streaming 程序则是 StreamingContext），由 SparkContext 负责和资源管理器 Cluster Manager（可以是 Standalone，Mesos，YARN）进行通信以及资源的申请、任务的分配和监控等。

SparkContext 根据 RDD 之间的依赖关系构建 DAG 图，提交给 DAG 调度器 DAGScheduler 进行解析，DAGScheduler 将 DAG 图分解成多个阶段（stage），也就是任务子集（Taskset），再提交给底层的任务调度器 TaskScheduler ，最后由 Task Scheduler 将 Task 发送给 Executor 运行。

![f987224f95342713f69ee1a0bd428474.png](4.png)

数据一致性方面，Spark 采用了 Checkpoint 机制保证了数据处理的 **exactly-once** 语义。

另外值得一提的是，Spark 提供了 MLlib 机器学习库，并且支持**流式机器学习算法**（streaming machine learning algorithms），包括 Streaming Linear Regression，Streaming KMeans 等等。这意味着可以一边使用流数据训练模型，一边将模型应用于流数据。此外，你也可以通过历史数据离线训练模型，再将模型在线地应用于实时流数据。

## Apache Flink

[Apache Flink](https://flink.apache.org/) 是由德国几所大学发起的的学术项目，后来不断发展壮大，并于 2014 年末成为 Apache 顶级项目。Flink 主要面向流处理，如果说 Spark 是批处理界的王者，那么 Flink 就是流处理领域的冉冉升起的新星。Flink 由 Java 和 Scala 编写，目前最新版本 1.11。

Flink 提供了负责流处理的  **DataStream API** 和负责批处理的 **DataSet API**，在其之上，又封装了 **Table API** 和 **SQL** 两种关系型 API。这两个 API 都是批处理和流处理统一的 API，这意味着在无边界的实时数据流和有边界的历史记录数据集上，关系型 API 会以相同的语义执行查询，并产生相同的结果。它们可以与 DataStream 和 DataSet API 无缝集成，并支持用户自定义的标量函数，聚合函数以及表值函数。

在 DataStream API 设计中，一个 Streaming Dataflow 被定义为由一系列 **Operator**（算子）组成，Operator 分为三类：Source Operator 定义入口；Sink Operator 定义出口；Transformation Operator 定义数据的中间转换操作。下图是一个使用 DataStream API 的示例：

![7489b8c785545f3d80c397cfc32bdca3.png](5.png)

在一个完整的 Dataflow DAG 中，可能包含多个 Source 和 Sink，一个 Transformation 也可以包含多个算子。在执行过程中，一个流会有一个或多个流分片（stream partitions），一个算子包含一个或多个算子子任务（operator subtasks），算子子任务的个数就是该算子的并行度（parallelism）。

Flink 运行时架构如下图所示，主要由一个 JobManager 进程和若干个 TaskManager 进程组成。其中，客户端 Client 并不是程序运行的组成部分，而是负责将用户的 Jar 包构建成 Dataflow Graph，提交到 JobManager 上。JobManager 和 TaskManager 既可以直接以 standalone 模式启动，也可以通过 YARN 或者 Mesos 等资源管理框架进行协调工作。

![e870b01bdb5da98f44fbc6729f928b14.png](6.png)

**JobManager** 类似于 Storm 中的 Nimbus，是协调 Flink 应用分布式执行的主进程。一个 Flink 应用中至少有一个 JobManager，在高可用（High Availability）模式下可能会存在多个 JobManager，它们中的一个作为 leader，其余作为 standby。JobManager 由三个组件组成：

- **ResourceManager**，负责 Flink 集群资源的分配。它管理着资源调度的最小单元 Task Slots，同时支持 YARN，Mesos，Kubernetes 等多种部署管理方式；
- **Dispatcher**，为用户提供了一个可以提交 Flink 应用的 REST 接口。同时  Dispatcher 也会启动一个Web UI，方便展示和监控作业执行的信息；
- **JobMaster**，负责管理作业图（JobGraph）的执行。多个 Job 可以同时在 Flink 集群上运行，每个 Job 会有自己独立的 JobMaster。

**TaskManager**，又称作 Worker，负责执行 Task，以及数据流的缓存和交换。Flink 很形象的将任务执行资源称为 Task Slot（插槽），每个插槽是 TaskManager 资源的一个固定子集，比如拥有 3 个插槽的 TaskManager 每个插槽能够使用 1/3 的内存。TaskManager 携有资源，而调度则是通过 JobManager。

数据一致性方面，Flink 通过 Checkpoint（检查点）机制，保证了数据处理的“精确一次”（**exactly-once**）语义。在应用程序运行期间，Flink 会定期检查状态的一致检查点。如果发生故障，Flink 会将程序状态置为最近的检查点时的状态，并重新启动处理流程，消费并处理检查点和发生故障之间的所有数据。尽管这意味着 Flink 会对一些数据处理两次（在故障之前和之后），我们仍然可以说这个机制实现了精确一次的一致性语义，因为所有算子的状态都已被重置，而重置后的状态下还不曾看到这些数据。

## 流处理框架演进

流处理框架的演进，要从 MapReduce 编程模型开始讲起。为了解决分布式计算学习和使用成本高的问题，Google 在 2004 年提出一种编程范式，它要求程序员将分布式数据操作拆分为两大步：`map` 和 `reduce`，也就是所谓的 MapReduce 编程模型。

在同一年，Hadoop 的创始人受 MapReduce 编程模型等一系列论文的启发，对论文中提出的模型进行了编程实现。时至今日，Hadoop 不仅仅是整个大数据领域的先行者和领导者，更形成了一套围绕 Hadoop 的生态系统，成为企业首选的大数据解决方案。不论是 Storm、Spark 还是 Flink 都是敞开怀抱拥抱 Hadoop 生态并融入成为了生态圈的一部分。

![e2dc809939ef9af086d3197d3e648b0e.png](7.png)

Hadoop 虽然已被公认为大数据分析领域无可争辩的王者，但它更加专注于批处理，并不适合做实时计算。随着 Hadoop 生态的繁荣发展，诞生了一批流处理框架，本质上它们的核心处理流程也不偏离 MapReduce 思想：

- **第一代**被广泛采用的流处理框架是 Storm，但由于 Storm 只支持 "at least once" 语义，对于很多对数据准确性要求较高的应用，Storm 有一定劣势。
- **第二代**非常流行的流处理框架是 Spark Streaming。Spark Streaming 使用微批处理的思想，每次处理一小批数据，以接近实时处理的效果。也正是由于时间间隔的存在，导致 Spark Streaming 的“实时处理”延迟较大，一般适用于延迟是秒级别的实时计算应用。但 Spark Streaming 的优势是拥有 Spark 这个靠山，用户从 Spark 迁移到 Spark Streaming 的成本较低，因此能给用户提供一个批量和流式于一体的计算框架。
- **第三代**流处理框架 Flink 是一个支持在有界和无界数据流上做有状态计算的大数据引擎。它以事件为单位，并且支持 SQL、State、WaterMark 等特性。比起 Storm，它的吞吐量更高，延迟更低，准确性能得到保障；比起 Spark Streaming，它以事件为单位，达到真正意义上的实时计算，且所需计算资源相对更少。

![67fc13f9d829042f7b800522e416d280.jpeg](8.jpg)

Spark 和 Flink 各有所长，也在相互竞争、相互借鉴。可以说 Spark 是以批处理起家的，通过使用内存计算比传统 Hadoop MapReduce 具有显著性能优势，Spark 已经成为行业内大数据批处理的首选处理引擎。但在处理稍微复杂点的实时流处理场景 (比如各种窗口、状态等) ，Flink 要比 Spark Streaming 更具有显著优势。事实证明，阿里最终在流处理框架选型中选择了 Flink，并在其之上开发了自己的流处理框架 Blink，并对 Flink 社区提供了贡献，包括促进 Flink 流处理、批处理一体化等。

## 流批一体

现如今，流批一体已经越来越成为一种趋势，它旨在将流处理和批处理通过一套相同的处理逻辑来实现。流批一体意味着计算引擎同时具备流计算的低延迟和批计算的高吞吐高稳定性，提供统一编程接口开发两种场景的应用并保证它们的底层执行逻辑是一致的。

2015 年，Google 提出了 [Dataflow](https://www.vldb.org/pvldb/vol8/p1792-Akidau.pdf) 模型，旨在提供一种统一批处理和流处理的解决方案。作为 Dataflow 模型的最早采用者之一，Apache Flink 在流批一体特性的完成度上在开源项目中是十分领先的。Flink 遵循 Dataflow 模型的理念: **批处理是流处理的特例**，亦即批处理处理的是无界数据流上的一小段有界数据流。Flink 设计之初流处理应用和批处理应用底层都是流处理，但在编程 API 上是分开的。

![dbcd10468cea3d61ff1f88d9c640643a.png](9.png)

在 Flink 架构上，负责物理执行环境的 Runtime 层是统一的流处理，上面分别有独立的 DataStream 和 DataSet 两个 API，两者基于不同的任务类型（Stream Task/Batch Task）和 UDF 接口（Transformation/Operator）。而更上层基于关系代数的 Table API 和 SQL API 表面上是统一的，但实际上编程入口（Environment）是分开的，且内部将流批作业分别翻译到 DataStream API 和 DataSet API 的逻辑也是不一致的。

基于批处理是流处理的特例的理念，用流处理表达批处理在语义上是完全可行的，而流批一体的难点在于批处理场景作为特殊场景的优化。对 Flink 而言，难点主要体现批处理作业在 Task 线程模型、调度策略和计算模型及算法的差异性上。因此，要实现真正的流批一体，Flink 需完成 Table/SQL API 的和 DataStream/DataSet API 两层的改造，将批处理完全移植到流处理之上，并且需要兼顾作为批处理立身之本的效率和稳定性。目前流批一体也是 Flink 长期目标中很重要一点，流批一体的完成将标志着 Flink 进入 2.0 新版本的时代。

## 参考

- [Apache Storm Tutorial](https://www.tutorialandexample.com/apache-storm-tutorial/)
- [Spark Streaming Programming Guide 官方文档](https://spark.apache.org/docs/latest/streaming-programming-guide.html)
- [Apache Flink 1.11 官方文档](https://ci.apache.org/projects/flink/flink-docs-release-1.11/learn-flink/)
- [尚硅谷Flink教程](https://confucianzuoyuan.github.io/flink-tutorial/book/chapter03-05-02-%E4%BB%8E%E4%B8%80%E8%87%B4%E6%A3%80%E6%9F%A5%E7%82%B9%E4%B8%AD%E6%81%A2%E5%A4%8D%E7%8A%B6%E6%80%81.html)
- [从Hadoop到Spark、Flink，大数据处理框架十年激荡发展史](https://zhuanlan.zhihu.com/p/94302767)
- [Flink 流批一体的实践与探索 —— 阿里云开发者社区](https://developer.aliyun.com/article/754468)
- [The Dataflow Model: A Practical Approach to Balancing Correctness, Latency, and Cost in Massive-Scale, Unbounded, Out-of-Order Data Processing](https://www.vldb.org/pvldb/vol8/p1792-Akidau.pdf)