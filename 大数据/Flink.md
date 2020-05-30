# Flink

官网翻译，版本:1.10

## 编程模型

### 抽象级别

提供了四层抽象：

![](C:\Users\Administrator\Desktop\flink\levels_of_abstraction.svg)

- 最底层抽象只提供了状态化流处理。内嵌ProcessFunction到DataStreamAPI。允许用户处理一个或多个事件流并使用一致性容错状态。允许用户注册事件时间与处理时间回调（也就是定时器），允许用户实现复杂计算。
- 实际上大多数应用不需要低层抽象，而是根据Core API进行编程，（DataStream和DataSet，前者对应无边界流，也就是流处理，后者对应边界流，也就是批处理）。
  这些fluent API提供了常用的数据处理构建块，如用户指定的转换，聚合，连接，窗口，状态。这些API中的数据类型处理在各自的编程语言中被描述为类。
  低层API与DataStream集成，使得某些操作可以使用低层抽象。DataSet API在有限数据集上提供了额外的primitives，例如loops和iterations.
- Table API是以表为中心的声明式DSL，这些表是可以动态改变的（当使用流描述时）。
  Table API遵循扩展的关系模型：Table附加了一个模式，API提供了可比较的操作，如，select，project，join，group-by，aggreate等。
  Table API程序声明式的定了逻辑操作应当如何完成，而不是精确的定义操作代码的样子。
  Table API的表现力不如Core API，但使用简洁。Table API程序在执行之前经过了优化器的优化。
  Table API可以无缝的在DataSet以及DataStream之间转换。
- SQL的语义与表现力类似于Table API，但使用SQL查询表达。
  SQL紧密的与Table API交互，SQL可以执行在Table API定义的表上。

### 程序与数据流

Flink程序的基础构建块是流和转换。概念上，流是一个永远不会终止的数据流，转换是使用一个或多个流作为输入，然后产生一个或多个流做为结果。

Flink执行时会被映射为Streaming dataflows，由流和转换算子组成。每一个数据流从一个或多个源开始结束于一个或多个sinks。数据流类似于任意的DAG。

![](C:\Users\Administrator\Desktop\flink\program_dataflow.svg)

通常转换与算子是一一对应的，某些时候一个转换可能由多个算子组成。

### 并行数据流

Flink程序本质上是并行和分布式的。在执行时，流有多个流分区，算子有多个子任务，子任务间相互独立，执行在不同的线程以及可能在不同的机器或容器中。

算子子任务是算子的并行度。流的并行度等于产生它的算子的并行度。相同程序的不同算子可能有不同的并行度。

![](C:\Users\Administrator\Desktop\flink\parallel_dataflow.svg)

流可以在算子间以一对一的模式或再分发模式传输数据：

- 一对一流保留元素的分区和顺序。意味着map子任务1将会以相同的顺序看到source子任务1相同的元素
- 再分发流改变了流的分区。每个算子子任务发送数据到不同的目标子任务，取决于所选的转换操作。再分发中，元素的顺序只在任务对中保留，例如map子任务一与keyBy子任务2。

### 窗口

流处理中的聚合事件完全不同于批处理。流处理的聚合通过窗口进行限制。

窗口可以是时间驱动（例如，每30秒统计依次）也可以是数据驱动（例如，每20个元素）。

窗口分类：

- 滚动窗口
- 滑动窗口
- 会话窗口
- 全局窗口

![](C:\Users\Administrator\Desktop\flink\windows.svg)

### 时间

流处理中的时间概念:

- 事件时间：时间创建的时间，通常使用事件时间戳描述。Flink通过时间赋值器访问事件时间。
- 摄入时间：事件进入源算子的时间。
- 处理时间：算子的本地时间。

![](C:\Users\Administrator\Desktop\flink\event_ingestion_processing_time.svg)

### 状态操作

记录了多个事件信息的操作称为状态化操作。

状态维护在一个内嵌的kv存储中。

状态与算子一起严格的分区和分布式化。

kv状态只能在keyed流上访问，也就是keyBy函数之后，一个key对应一个状态，访问时只能访问当前key的状态。

key与kv状态是一一对应的，确保了状态操作都在本地，保证了一致性，省去了事务开销。对齐同样允许Flink再分发状态以及透明的调整流分区。

![](C:\Users\Administrator\Desktop\flink\state_partitioning.svg)

### 容错检查点

Flink使用流回放以及检查点来实现容错。

一个检查点与每个输入流中的特定点以及每个算子的相应状态相关。

一个流式数据流可以从检查点恢复，同时通过恢复算子的状态保持一致性，并且从检查点中的特定点回放事件。

检查点间隔是用恢复时间交换容错开销的手段。

### 流处理之上的批处理

Flink执行批处理为一种流处理，这里的流是边界化，元素有限。

一个Dataset内部处理为一个数据流。上述的概念同样适用于批处理，有几个例外：

- 批处理容错不使用检查点。通过完全回放进行恢复。这增加了恢复的开销，但处理变得更廉价。
- 批处理的状态操作使用了简化的内存/核心外的数据结构而不是kv索引。
- DataSet API引入了特殊的同步性迭代，这些迭代只能用装批处理上。

## Maven

```shell
mvn archetype:generate                               \
      -DarchetypeGroupId=org.apache.flink              \
      -DarchetypeArtifactId=flink-quickstart-scala     \
      -DarchetypeVersion=1.10.0   \
      -DgroupId=com.hiscat \
      -DartifactId=wordcount \
      -Dversion=0.1 \
      -Dpackage=wordcount \
      -DinteractiveMode=false
      
 mvn archetype:generate \
    -DarchetypeGroupId=org.apache.flink \
    -DarchetypeArtifactId=flink-walkthrough-datastream-scala \
    -DarchetypeVersion=1.10.0 \
    -DgroupId=frauddetection \
    -DartifactId=frauddetection \
    -Dversion=0.1 \
    -Dpackage=spendreport \
    -DinteractiveMode=false
```

```xml
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-scala_2.11</artifactId>
  <version>1.10.0</version>
  <scope>provided</scope>
</dependency>
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-streaming-scala_2.11</artifactId>
  <version>1.10.0</version>
  <scope>provided</scope>
</dependency>
```

