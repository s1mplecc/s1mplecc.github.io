---
title: 图流算法之 Triangle Counting
tags: [Graph Theory, Stream Progressing]
categories: [Algorithms]
date: 2020-07-21 09:15:32
---
## 前言

图中的三角形是复杂网络分析的重要角色，不论是来自社会交互、计算机交流、金融交易、蛋白质还是生态学网络，图中三角形的数量都是巨大的，它在这些领域中有着非常广泛的应用。例如：通过三角形的分布可以区分哪些是垃圾邮件的主人；角色行为识别中，通过使用者参与的三角形的数量可以判断这个使用者的地位。

图三角数量的计算（Triangle Counting）是计算网络聚集系数和传递性的重要步骤，广泛应用于重要角色识别、垃圾邮件检测、社区发现、生物检测等。目前图中三角形的计算主要分为**准确计算**（exact counting）和**近似计算**（approximate counting）两种类型。 

## Gelly Streaming

Gelly Streaming 框架给出的 Triangle Counting 示例算法的 Java Class：
- `WindowTriangles`：Counts exact number of triangles in a graph slice.
- `ExactTriangleCount`：Single-pass, insertion-only exact Triangle Local and Global Count algorithm.
- `BroadcastTriangleCount`：The broadcast triangle count example estimates the number of triangles in a streamed graph.

## Exact Counting

准确计算可以准确地计算图中三角形的数量，对于规模很大的大图来说，计算的时空消耗很大。所以**在流处理情形下，我们可以将图的边流按照时间来切片，计算一个图切
片（graph slice）中的三角形准确数量**。比如 Gelly 示例类 `WindowTriangles` 中的处理流程：

```
edges.slice(windowTime, EdgeDirection.ALL)
        .applyOnNeighbors(new GenerateCandidateEdges())
        .keyBy(0, 1).timeWindow(windowTime)
        .apply(new CountTriangles())
        .timeWindowAll(windowTime).sum(0);
```

需要注意的是，在这种情况下，一个图分片的所有信息我们都已获得，**相当于我们是在批处理情形下计算一个图的三角数量**。况且图的规模不大，在顶点和边已知的情况下，图三角的计算就变的简单了。算法的核心步骤如下：
1. 计算每一个顶点（Vertex）的领域（Neighborhood），即一个顶点的所有相邻顶点集合；
2. 领域中的每两个顶点构成一条候选边（Candidate Edge），为避免重复计算需要满足 `neighborIds[i] > vertexID && neighborIds[j] > vertexID`；
3. 如果候选边存在于图中，则 Count + 1。

## Approximate Counting

当图的规模很大时，准确计算的时空复杂度将会很大，而在很多应用中只需要近似得到图三角的数量，所以近年来近似算法得到了很多关注。 

## 参考

- [Gelly Streaming：An experiemental API for single-pass graph streaming analytics on Apache Flink.](https://github.com/vasia/gelly-streaming)
- [图论概念梳理](https://yhx-12243.github.io/OI-transit/memos/14.html)
- [Research progress of triangle counting in big data](http://www.infocomm-journal.com/dxkx/CN/10.11959/j.issn.1000-0801.2016169)
- [Counting Triangles in Real-World Graph Streams:Dealing with Repeated Edges and Time Windows](https://ieeexplore.ieee.org/document/7421397)