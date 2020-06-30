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

Flink程序执行时会被映射为Streaming dataflows，由流和转换算子组成。每一个数据流从一个或多个源开始，结束于一个或多个sinks。数据流类似于任意的DAG。

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

- 事件时间：事件创建的时间，通常使用事件时间戳描述。Flink通过时间赋值器访问事件时间。
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

## 分布式运行时

### 任务以及算子链

对于分布式执行，Flink链化算子子任务为任务。每个任务通过一个线程执行。

链化算子为任务是一个优化：减少了线程到线程的传输以及缓存的开销，增加了整体的吞吐同时减少了延迟性。

链化行为可以被配置。

![](C:\Users\Administrator\Desktop\flink\分布式运行时\tasks_chains.svg)

### Job管理器，任务管理器，客户端

Flink运行时由2种进程类型组成：

- Job管理器（又名master）协调分布式执行。负责调度任务，协调检查点，协调失败恢复等。
  至少有一个Job管理器。HA情况下有多个，一个为leader其余为standby。
- 任务管理器（又名worker）执行数据流的任务（子任务），缓冲并交换数据流。至少有一个任务管理器。

Job管理器与任务管理器有多种启动方式：

- standalone模式
- YARN模式
- Mesos模式
- K8s模式

任务管理器启动后连接到Job管理器，通过心跳保持存活，然后领取任务。

客户端不是运行时和程序执行的一部分，但用来准备和发送数据流到master。此后，客户端可以断开连接或等待接收进度报告。客户端可以用Java/Scala程序触发执行，或用CLI触发。

![](C:\Users\Administrator\Desktop\flink\分布式运行时\processes.svg)

### 任务槽与资源

每一个worker是一个JVM进程，可能在单独的线程中执行多个子任务。worker使用任务槽控制任务的接受数量（至少一个）。

每个任务槽描述了woker的固定资源子集。例如，三个槽的woker会平均分配内存到每个槽。

槽化资源意味着子任务间不会有资源竞争，会保留一定数量的托管内存。

槽只隔离了内存并没有隔离CPU，目前槽只拆分了任务的托管内存。

通过调整任务槽的数量，用户可以定义子任务间的隔离方式。

每个woker一个槽意味着每个任务组运行在单独的JVM中，多个槽意味着多个子任务共享同一个JVM。同一个JVM中的任务共享TCP连接（通过多路复用）和心跳信息。它们可能会共享数据集和数据结构，从而减少每个任务的开销。

![](C:\Users\Administrator\Desktop\flink\分布式运行时\tasks_slots.svg)

默认的，Flink允许子任务共享槽，即使它们是不同任务的子任务，只要它们来自同一个Job。这意味着一个槽可能会持有一个Job的整个流水线。允许槽共享有2个好处：

- Flink集群需要的任务槽与job中的最高并行度刚好相同。不需要计算一个程序包含多少个任务。
- 这样更容易获得更好的资源利用。没有槽共享的情况下非密集型source/map子任务会阻塞与密集型窗口任务一样多的资源。在槽共享的情况下，增加了基础并行度，同时确保了大量的子任务公平的分布在worker中。

![](C:\Users\Administrator\Desktop\flink\分布式运行时\slot_sharing.svg)

槽共享API同样包括了一个资源组机制，可以用来阻止意外的槽共享。经验表明，槽数等于CPU核数最好。

### 状态后台

kv索引存储的数据结构取决于所选的状态后台。

内存状态后台保存在hash map中，RocksDB保存在KV存储中。

除了定义状态的数据结构，状态后台也要实现快照机制以及保存在检查点中。

![](C:\Users\Administrator\Desktop\flink\分布式运行时\checkpoints.svg)

### 存档点

DataStream API中的程序能从存档点恢复。存档点允许更新程序和集群，并且不会丢失状态。

**存档点是手动触发的检查点，**拍下程序的快照并写入状态后台。存档点依赖于常规的检查点机制。程序执行期间周期性的产生快照并产生检查点。恢复只需最新的检查点，一旦新的检查点产生旧的就可以丢弃。

存档点类似于周期性的检查点，但是由用户触发，并且不会自动过期。存档点可以通过命令行创建或REST API创建。

## 组件栈

作为一个软件栈，Flink是一个层级系统。不同的栈层构建在彼此之上，并提高了它们所接受的程序描述抽象层：

- 捆绑在Flink中库和API生成DataSet或DataStream API程序。
  如，Table，CEP，Gelly
- DataStream以及DataSet API 通过单独编译生成JobGraph。DataSet API使用优化器来决定程序的最优计划，而DataStream API使用流建造器。
- 运行时以JobGraph的形式接受程序。JobGraph是泛化的并行数据流包含了任意数量的任务。
- 工作图根据Flink中多种可用的部署选项来执行。

![](C:\Users\Administrator\Desktop\flink\stack.png)

## 工作与调度

### 调度

Flink中的执行资源是通过任务槽定义的。

每个worker有多个槽，每个槽能运行一个并行任务的流水线。

每个流水线由多个连续的任务组成。

**Flink通常并发的执行连续的任务。**

![](C:\Users\Administrator\Desktop\flink\工作调度\slots.svg)

内部，Flink通过SlotSharingGroup以及CoLocationGroup分别定义哪些任务可能共享槽（允许的话），哪些任务必须严格放置在同一个槽。

### Master数据结构

在job执行期间，master持续跟踪分布式任务，确定何时调度下一个任务，以及响应任务的完成或失败。

master接受JobGraph，JobGraph是一个由算子（JobVertex）和中间结果（IntermediateDataSet）组成的数据流的描述。

每个算子有属性，如并行度，代码。此外JobGraph有一组附加的库，这些库是执行算子所必需的。

master转换JobGraph为ExecutionGraph。



执行图是工作图的并行版本：对于每个JobVertex，每个并行子任务包含了一个ExecutionVertex。

一个具有100并行度的算子只有一个JobVertex以及100个ExecutionVertex。

ExecutionVertex跟踪了一个特殊子任务的执行状态。

一个JobVertex的所有ExecutionVertex保存在一个ExecutionJobVertex中，它跟踪了算子的整体状态。

除了顶点之外，ExecutionGraph同样也包含了IntermediateResult以及IntermediateResultPartition。前者跟踪IntermediateDataSet，后者跟踪每个分区的状态。

![](C:\Users\Administrator\Desktop\flink\工作调度\job_and_execution_graph.svg)

每个ExecutionGraph关联了一个job status。job status指示了当前job执行的状态。

- Flink job起始状态是created
- 然后切换到running
- 所有工作完成后切换到finished
- 失败的情况下，首先切换到failing，所有的running任务会被取消
- 如果所有的job vertices抵达最终状态且job不可重启，job failed。
- 如果可以重启，进入restarting。
- 一旦完全重启，进入created。
- 在用户取消job的情况下，进入cancelling。
- 一旦所有的running 任务抵达最终状态，过渡到canceled。

不像finished，canceled和failed指示了一个全局的终止状态，因此触发器可以清除job。

suspended状态只是本地终止。本地终止意味着job终止在一个master，集群可以从另一个master中恢复它。因此suspended状态的job不会完全的清除。

![](C:\Users\Administrator\Desktop\flink\工作调度\job_status.svg)

在ExecutionGraph执行期间，每个并行任务会通过多个阶段，从created到finished或failed。一个任务可能会执行多次，在失败的情况下。出于这个原因，ExecutionVertex的执行被跟踪在Execution中，每个ExecutionVertex有一个当前的Execution和上一个Execution。

![](C:\Users\Administrator\Desktop\flink\工作调度\state_machine.svg)

## 任务生命周期

任务是Flink中的基本执行单元。它是每个算子并行实例被执行的地方。例如，并行度为5的算子，每个实例会执行在单独的任务中。

StreamTask是所有任务子类的基类。

### 算子生命周期

由于任务是执行算子并行实例的实体，所以它的生命周期与算子紧密集成。

1. 初始化阶段
2. 处理阶段
3. 检查点阶段
4. 终止阶段

```
    // initialization phase
    OPERATOR::setup
        UDF::setRuntimeContext
    OPERATOR::initializeState
    OPERATOR::open
        UDF::open
    
    // processing phase (called on every element/watermark)
    OPERATOR::processElement
        UDF::run
    OPERATOR::processWatermark
    
    // checkpointing phase (called asynchronously on every checkpoint)
    OPERATOR::snapshotState
            
    // termination phase
    OPERATOR::close
        UDF::close
    OPERATOR::dispose
```

setup方法用于初始化特定于算子的机制，

例如，setRuntimeContext，指标集合数据结构。

此后，调用initializeState方法初始化算子状态。

然后，调用open方法执行特定于算子的初始化，例如，调用udf函数的open方法。

> 注意：initializeState方法包含了首次执行初始化状态的逻辑，也包含了在失败后从检查点检索状态的逻辑。

设置完成后，开始处理数据。进入的元素可能是：正常的元素，水印，检查点屏障。

每种元素都有特定的方法进行处理。

- 元素调用processElement方法。
- 水印调用processWatermark方法。
- 检查点屏障调用snapshotState方法。

>processElement方法同样也是UDF逻辑被调用的地方，例如MapFunction的map方法

最后，在正常无错的算子终止情况下，close方法被调用，用于完成最终的记账动作。

> UDF的close方法在此之后被调用

随后dispose方法被调用。

在由于失败被终止或手动取消的情况下，直接调用dispose方法，跳过中间阶段。

**检查点：**snapshotState方法是被异步调用的。检查点是在处理阶段进行时完成的。此方法会将算子的状态写入状态后台。

### 任务生命周期

#### 正常执行

```
   TASK::setInitialState
    TASK::invoke
	    create basic utils (config, etc) and load the chain of operators
	    setup-operators
	    task-specific-init
	    initialize-operator-states
   	    open-operators
	    run
	    close-operators
	    dispose-operators
	    task-specific-cleanup
	    common-cleanup
```

如上所示，在恢复任务配置以及初始化某些重要的运行时参数后，第一步是检索任务级别的初始化状态。

这是调用setInitializeState方法完成的，有2个特别重要的情况:

1. 当任务从失败恢复以及从上一次成功的检查点重启时。
2. 当从一个存档点恢复时。

如果任务首次执行，初始化状态设置为空。

在恢复初始状态后，任务进入invoke方法。

首先，调用算子的setup方法来初始算子涉及到的本地计算。

然后调用init方法，完成特定于任务的初始化。此方法用于获取任务级别所需的资源。

在获取必要的任务级资源后，调用每个算子的initializeState方法获取算子级和UDF级的资源。此方法应当被每个状态算子重写，并包含状态初始化逻辑，包括首次执行和失败恢复或从存档点恢复。

初始化算子级状态后，调用每个算子的open方法。此方法完成所有操作的初始化，例如，注册已检索到的定时器。

open方法从算子链的底部开始调用直到头部，这是因为当第一个算子开始处理数据时，所有下游算子已经就绪接收数据。

当任务和算子准备就绪后，调用run方法。run方法会调用算子的processElement方法和processWatermark方法。

在运行到完成的情况下，即没有任何输入，退出run方法后，任务进入shutdown过程。

首先定时器服务停止注册新的定时器，清除所有未启动的定时器，等待执行中的定时器完成。

然后调用算子的close方法关闭计算，此后任意缓冲的输出数据将会被flush。最后调用dispose方法清除所有算子持有的资源。

> 算子链的dispose方法是从前往后调用。

最终，当所有的算子被关闭，资源被释放，任务停止定时器服务，完成任务级清理。

**检查点**：检查点是在不同的线程完成的。CheckpointBarriers元素周期性的注入数据流中，并且与真实数据一起从源流向sink。

源任务在running模式后注入这些屏障，并且假设检查点协调器同样也是running模式。无论何时一个任务接受到一个屏障后，通过检查点线程调用算子的snapshotState方法进行快照。此时数据仍然被接受，但会被缓冲，直到检查点成功完成，才会继续处理以及发送至下游算子。

#### 中断执行

当算子取消时，从取消时开始，只会执行定时器服务停止，任务级清理，算子销毁，通用任务清理。

## 数据流容错

### 介绍

Flink提供了容错机制用于一致性恢复数据流程序的状态。

容错机制确保了即使是在失败的情况下，程序状态最终只会恰好一次的反应数据流中的记录。

> 有一个开关可以切换到至少一次的保障。

容错机制持续的绘制分布式流式数据流的快照。具有小状态的流程序，快照会非常轻量级，能够频繁的绘制且不影响性能。流应用的状态被存储在一个可配置的地方（HDFS，master memory）。



在程序失败的情况下，Flink会停止分布式流式数据流。系统会重启算子并重置它们到最近一次成功的检查点。输入流重置到状态快照的位置点。重启后处理的记录保证不再是前一次检查点之前的记录。

> 默认检查点是禁用的。

> 要实现恰好一次的保障，数据源需要重播能力。

> Flink通过分布式快照实现检查点，检查点和快照是可以互换的。

### 检查点

Flink容错机制的核心部分是绘制分布式数据流和算子状态的一致性快照。这些快照充当一致性检查点，在失败的情况下系统能回退到这些点。

分布式快照：http://research.microsoft.com/en-us/um/people/lamport/pubs/chandy.pdf

#### 屏障

Flink的分布式快照的核心元素是流屏障。

这些屏障被注入到数据流中，与记录一起被当作数据流的一部分。

屏障永远不会超过记录，它们严格的按顺序流动。

屏障拆分数据流中的记录为当前快照记录集以及下一个快照集。

每个屏障携带了一个快照ID，ID之前的记录都处理完成了。

屏障不会中断流的流动，因此非常轻量级。来自不同快照的多个屏障能同时出现在流中，这意味着多个快照可以并发的进行。

![](C:\Users\Administrator\Desktop\flink\数据流容错\stream_barriers.svg)

流屏障在流式源处被注入到并行数据流中。

快照n（称之为S<sub>n</sub>）被注入的点是源流中快照所覆盖数据的位置。

例如，Kafka中，这个位置是最后一条记录的偏移量。

S<sub>n</sub>的位置被报告给检查点协调器（master）



随后屏障流向下游。当一个中间算子从它所有的输入流接收到快照n的屏障时，它会发送快照n的屏障到所有的输出流。

一旦sink算子从所有的输入流中接受到屏障n，它会向master ack快照n。在所有的sink ack完一个快照后，就可以认为sink已经完成了快照。

一旦快照n已经完成，job永远不会再次查询快照n之前的记录，因为快照n及以前的记录已经通过整个数据流拓扑。

![](C:\Users\Administrator\Desktop\flink\数据流容错\stream_aligning.svg)

接受多个输入流的算子需要对齐输入流的屏障，上图演示了对齐：

- 一旦算子从输入流接收到快照屏障n，就不能再继续处理该流的记录，直到接收到其他输入流的屏障n。否则，它将会混合属于快照n和快照n+1的记录。
- 报告屏障n的流暂时被搁置。从那个流接收的记录不会被处理，但会被放入输入缓冲区。
- 一旦最后一个流接收到屏障n，算子会发送所有挂起的输出记录，然后发送屏障n本身。
- 在那之后，算子继续处理所有输入流的记录，在处理流中记录之前处理输入缓冲区的记录。

#### 状态

当算子包含任意形式的状态时，状态必须是快照的一部分。算子状态以不同的形式出现:

- 用户定义状态：由转换函数创建并修改
- 系统状态：这个状态指的是作为算子计算部分的数据缓冲区。例如，窗口缓冲区。

算子在接受到快照屏障时会给自己的状态拍快照，并且是在发送屏障到输出流之前。

在这点上，所有屏障前的更新都已完成，屏障后的更新不会存在。

因为快照的状态可能过大，所以它被存储在可配置的状态后台中。

默认是master的内存中，但生产环境应该配置为分布式可靠存储。

在状态存储后，源算子会向master确认检查点，发送快照屏障至输出流，然后继续。

产生的快照现在包含:

- 每个并行流数据源的偏移量
- 每个算子的状态指针

![](C:\Users\Administrator\Desktop\flink\数据流容错\checkpointing.svg)

#### 恰好一次 VS 至少一次

对齐步骤可能会添加延迟性。通常会有几毫秒的延迟，但我们看到了一些延迟明显增加的情况。对于所有记录都要求超低延迟的应用，Flink有一个开关，可以在检查点时跳过对齐。

一旦算子从每个输入流中看到检查点屏障，快照依然绘制。

当对齐跳过时，算子继续处理输入，即使在某些检查点屏障之后到达。这种方式，在快照n被拍下之前，算子同样也会处理属于快照n+1的元素。

在恢复时，这些记录会重复出现，因为快照n中包括了这些记录，重播时会作为快照n的一部分。

> 对齐只会发生在有多个前任输入流以及由多个发送者的情况下。

#### 异步状态快照

每次同步快照时会引入延迟，算子会停止处理数据并缓冲。

Flink使用异步异步状态快照，使用copy-on-write的数据结构，例如RocksDB

在接收到检查点屏障后，算子开始异步快照，拷贝算子状态。

算子会立即发送屏障至下游并继续正常的流处理。

一旦后台拷贝过程完成，算子会向master确认。

在所有的sink接收到屏障以及所有的状态算子向master确认它们的状态备份后，整个检查点过程完成。

### 恢复

恢复在检查点的机制下变得很简单：失败时，Flink会选择最近一次完成的检查点k进行恢复。

然后系统重新部署整个分布式数据流，并且赋予每个算子检查点k的算子状态。

源算子设置从检查点k中的位置开始读取。例如，Kafka中，这个位置是消费者分区的偏移量。

如果状态是增量的，算子会从最新的完整快照状态开始，然后使用一系列的增量快照更新状态。

### 算子快照实现

当算子快照被拍下时，有2部分组成：同步与异步。

算子以及状态后台将这些快照作为一个java FutureTask提供。

这些任务包含状态同步完成的部分以及异步挂起的部分。然后异步的部分通过一个检查点后台线程执行。

检查点纯同步的算子作为一个已完成的FutureTask。如果异步操作需要完成，则执行run方法。

这些任务可以被取消，所以流以及其他资源可以被释放。

## 文件系统

Flink有自己的文件系统抽象： `org.apache.flink.core.fs.FileSystem`

这个抽象提供了一组通用的操作以及多个文件系统实现之间的最小保证。Flink提供的文件系统操作十分有限。

### 实现

Flink直接实现了文件系统，以file模式开头，描述了本地文件系统。

其他文件系统类型由桥接到Hadoop支持的文件系统套件进行访问。

如果在类路径下找到了Hadoop文件系统的类以及有效配置，则透明的加载Hadoop文件系统。默认查找类路径下的Hadoop配置，可以使用fs.dfs.hadoopconf配置进行更改。

### 持久化保证

FileSystem以及FsDataOutputStream实例用于持久化存储数据，即用于应用的结果也用于容错与恢复。因此，定义良好的流持久化语义是至关重要的。

#### 持久化保证定义

数据写入输出流被认为是持久化，有2个需求要满足：

1. 可见性需求：当给出一个绝对路径时，必须保证所有的进程，机器，虚拟机，容器等能够访问这个文件一致性的看见数据。
   这个需求类似于close-to-open语言，定义在POSIX，但受限于文件本身。
2. 持久性需求：文件系统的特殊持久性需求必须满足。
   这是特定于文件系统的。例如，本地文件系统在硬件以及操作系统宕机的情况下不保证持久性，HDFS则保证n个节点可以失败，文件任然可用。

不需要更新父目录就可以认为文件流中的数据是持久化的。

这种放松对于文件系统是重要的，更新目录内容只会最终保持一致。

FSDataOutputStream在调用close方法后，必须保证数据持久化。

#### 案例

- 对于容错分布式文件系统，一旦数据被接收并被确认，就认为是持久化的，通常是看到法定机器数量复制了副本。此外，文件绝对路径必须对所有有权访问的机器可见。
  数据是否命中非易失性存储，取决于特定的文件系统。
  父目录元数据的更新不要求抵达一致性状态。只要可以通过文件的绝对路径进行访问，对于某些机器可以在监听父目录时可以看见一些文件，而其他机器不能是允许的。
- 本地文件系统必须支持POSIX close-to-open语义。因为本地文件系统没有任何容错保证，没有更多的需求存在。
  这暗示了，从本地文件系统的角度来看，当数据被认为是持久化时，可能仍然在OS缓存中。宕机引起的缓存数据丢失被认为是本地机器失败，并不是Flink定义的本地文件系统保证所覆盖的情况。
  这意味着，计算结果，检查点，存档点，写入本地文件系统，不被保证可恢复，在本地机器失败的情况下，这使得本地文件系统不适用于生产设置。

### 更新文件内容

大多数文件系统要么不支持覆写已存在的文件内容，要么不支持覆写文件时，更新内容的一致性可见。出于此因，Flink的文件系统不支持追加到已存在的文件或者查看输入流中上次已写入的数据。

### 覆写文件

覆写文件是可能的。通过删除文件然后重新创建进行覆写。但是，某些文件系统无法使该更改对所有有权访问该文件的人可见。某些机器可能看见旧文件，某些机器可能看见新文件。

为了避免这些一致性问题，Flink中的失败/恢复机制的实现严格的避免写入这些文件多于一次。

### 线程安全

文件系统的实现必须是线程安全的：同一个文件系统实例经常跨线程共享，必须有能力并发的创建输入/输出流以及列出文件元数据。

FSDataOutputStream以及实现严格的非线程安全：流实例不应该在读或写操作的线程间传递，因为没有跨线程操作可见性的保证。（许多操作不能创建内存屏蔽）

## 基础API概念

### 数据集与数据流

Flink使用DataSet以及DataStream描述数据。

可以认为这些API是不可变的数据集合。

DataSet对应有限数据集。DataStream对应无限数据集。

集合一旦创建不可被修改。可以通过Source创建集合，新集合可以使用转换在旧集合上衍生。

### Flink程序剖析

程序组成:

1. 获取执行环境
2. 加载数据集
3. 使用转换函数
4. 输出结果
5. 触发执行

- DataSet API：org.apache.flink.api.scala
- DataStream API：org.apache.flink.streaming.api.scala

```scala
getExecutionEnvironment()

createLocalEnvironment()

createRemoteEnvironment(host: String, port: Int, jarFiles: String*)
```



### 惰性求值

### 指定Key

### 指定转换函数

### 数据类型

### 累加器与计数器

## 事件时间概述

## 时间戳与水印

## 时间戳提取器与水印发射器

## 状态概述

## 使用状态

## 广播状态模式

## 检查点

## 可查询状态

## 状态后台

## 状态模式演化

## 自定义状态序列化

## 算子概述

## 窗口

## 连接

## 处理函数

## 异步IO

## 副输出

## Table API 与SQL概述

## Table API与SQL常见概念与API

## 流概念概述

## 动态表

## 时间属性

## 持续查询中的连接

## 时间表

## 检测模式

## 数据类型

## 函数概述

## 内置系统函数

## UDF

## Hive集成

## 性能调优

## 自定义源与Sink

## 数据类型与序列化