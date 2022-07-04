# mapreduce-spark-flink
## Spark 与 Hadoop [原文链接](http://c.biancheng.net/view/3642.html)
根据 Hadoop MapReduce 的工作流程，可以分析出 Hadoop MapRedcue 的一些缺点。<br />
1）Hadoop MapRedue 的表达能力有限。<br />
所有计算都需要转换成 Map 和 Reduce 两个操作，不能适用于所有场景，对于复杂的数据处理过程难以描述。<br />
2）磁盘 I/O 开销大。<br />
Hadoop MapReduce 要求每个步骤间的数据序列化到磁盘，所以 I/O 成本很高，导致交互分析和迭代算法开销很大，而几乎所有的最优化和机器学习都是迭代的。所以，Hadoop MapReduce 不适合于交互分析和机器学习。<br />
3）计算延迟高。<br />
如果想要完成比较复杂的工作，就必须将一系列的 MapReduce 作业串联起来然后顺序执行这些作业。每一个作业都是高时延的，而且只有在前一个作业完成之后下一个作业才能开始启动。因此，Hadoop MapReduce 不能胜任比较复杂的、多阶段的计算服务。<br />
<br />
Spark 是借鉴了 Hadoop MapReduce 技术发展而来的，继承了其分布式并行计算的优点并改进了 MapReduce 明显的缺陷。<br />
Spark 使用 Scala 语言进行实现，它是一种面向对象的函数式编程语言，能够像操作本地集合对象一样轻松地操作分布式数据集。它具有运行速度快、易用性好、通用性强和随处运行等特点，具体优势如下。<br />
1）Spark 提供了内存计算，把中间结果放到内存中，带来了更高的迭代运算效率。通过支持有向无环图（DAG）的分布式并行计算的编程框架，Spark 减少了迭代过程中数据需要写入磁盘的需求，提高了处理效率。<br />
2）Spark 为我们提供了一个全面、统一的框架，用于管理各种有着不同性质（文本数据、图表数据等）的数据集和数据源（批量数据或实时的流数据）的大数据处理的需求。
Spark 使用函数式编程范式扩展了 MapReduce 模型以支持更多计算类型，可以涵盖广泛的工作流，这些工 作流之前被实现为 Hadoop 之上的特殊系统。
Spark 使用内存缓存来提升性能，因此进行交互式分析也足够快速，缓存同时提升了迭代算法的性能，这使得 Spark 非常适合数据理论任务，特别是机器学习。<br />
3）Spark 比 Hadoop 更加通用。Hadoop 只提供了 Map 和 Reduce 两种处理操作，而 Spark 提供的数据集操作类型更加丰富，从而可以支持更多类型的应用。
Spark 的计算模式也属于 MapReduce 类型，但提供的操作不仅包括 Map 和 Reduce，还提供了包括 Map、Filter、FlatMap、Sample、GroupByKey、ReduceByKey、Union、Join、Cogroup、MapValues、Sort、PartionBy 等多种转换操作，以及 Count、Collect、Reduce、Lookup、Save 等行为操作。<br />
4）Spark 基于 DAG 的任务调度执行机制比 Hadoop MapReduce 的迭代执行机制更优越。
Spark 各个处理结点之间的通信模型不再像 Hadoop 一样只有 Shuffle 一种模式，程序开发者可以使用 DAG 开发复杂的多步数据管道，控制中间结果的存储、分区等。<br /><br />

对于多维度随机查询也是一样。在对 HDFS 同一批数据做成百或上千维度查询时，Hadoop 每做一个独立的查询，都要从磁盘中读取这个数据，而 Spark 只需要从磁盘中读取一次后，就可以针对保留在内存中的中间结果进行反复查询。<br /><br />

Spark 在 2014 年打破了 Hadoop 保持的基准排序（SortBenchmark）记录，使用 206 个结点在 23 分钟的时间里完成了 100TB 数据的排序，而 Hadoop 则是使用了 2000 个结点在 72 分钟才完成相同数据的排序。也就是说，Spark 只使用了百分之十的计算资源，就获得了 Hadoop 3 倍的速度。<br />

尽管与 Hadoop 相比，Spark 有较大优势，但是并不能够取代 Hadoop。<br /><br />

因为 Spark 是基于内存进行数据处理的，所以不适合于数据量特别大、对实时性要求不高的场合。另外，Hadoop 可以使用廉价的通用服务器来搭建集群，而 Spark 对硬件要求比较高，特别是对内存和 CPU 有更高的要求。<br />
