> 英文原文：http://peter.bourgon.org/blog/2017/02/21/metrics-tracing-and-logging.html

译者：吴晟

原作发表时间： 2017年2月21日

这是在OpenTracing和分布式追踪领域内广受欢迎的一篇博客文章。在构建监控系统时，大家往往在这几个名词和方式之间纠结。 通过这篇文章，作者很好的阐述了分布式追踪、统计指标与日志之间的区别和关系。

## 正文

今天，我很荣幸的参加了2017分布式追踪峰会（2017 Distributed Tracing Summit）， 并和来自AWS/X-Ray, OpenZipkin, OpenTracing, Instana, Datadog, Librato，以及其他更多组织的同仁进行了愉快的沟通和讨论。 其中一个重要的论点，是针对监控项目的范围和定义的。作为一个分布式追踪系统，应该管理日志么？从不同角度看来，到底什么是日志？如何通过一张图形象的定位这些形形色色的系统？

总体说来，我觉得我们是在一些通用的名词间纠结。我想我们可以通过图表来定义监控的作用域，使各名词的作用范围更明确。 我们使用维恩图（Venn diagram）来描述Metrics, tracing, logging三个概念的定义。他们三者在某些情况下是重叠的，但是我尽量尝试定义他们的不同。如下图所示：

![640?wx_fmt=png&wxfrom=5&wx_lazy=1](https://ss.csdn.net/p?http://mmbiz.qpic.cn/mmbiz_png/gak2lhVxV6KBxvJXcYALaE5g3QIfS7tmnZn9fIE0GygBT9M2EhiaQF9coBrCV8Vpeic4NaoUIFdWSNfuVhDuMaIw/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

Metric的特点是，它是可累加的：他们具有原子性，每个都是一个逻辑计量单元，或者一个时间段内的柱状图。 例如：队列的当前深度可以被定义为一个计量单元，在写入或读取时被更新统计； 输入HTTP请求的数量可以被定义为一个计数器，用于简单累加； 请求的执行时间可以被定义为一个柱状图，在指定时间片上更新和统计汇总。

logging的特点是，它描述一些离散的（不连续的）事件。 例如：应用通过一个滚动的文件输出debug或error信息，并通过日志收集系统，存储到Elasticsearch中； 审批明细信息通过Kafka，存储到数据库（BigTable）中； 又或者，特定请求的元数据信息，从服务请求中剥离出来，发送给一个异常收集服务，如NewRelic。

tracing的最大特点就是，它在单次请求的范围内，处理信息。 任何的数据、元数据信息都被绑定到系统中的单个事务上。 例如：一次调用远程服务的RPC执行过程；一次实际的SQL查询语句；一次HTTP请求的业务性ID。



根据上述的定义，我们可以标记上图的重叠部分。

![640?wx_fmt=png](https://ss.csdn.net/p?https://mmbiz.qpic.cn/mmbiz_png/gak2lhVxV6KBxvJXcYALaE5g3QIfS7tmGOjHcicXLnEjPiaaGSKo0qNaBrMXdKiaxa2SekrbPicBLhj8rgY6u0y7GQ/640?wx_fmt=png)

当然，大量的被监控的应用是具有分布式能力(Cloud-native)的应用，逻辑处理在单次请求的范围内完成。因此，讨论追踪的上下文是有意义的。 但是，我们注意到，并不是所有的监控系统都绑定在请求的生命周期上的。他们可能是逻辑组件诊断信息、处理过程的生命周期明细信息，这些信息和任何离散的请求时正交关系。 所以，不是所有的metric和log都可以被塞进追踪系统的概念中，至少在不经过数据加工处理是不行的。又或者，我们可能发觉使用metric统计数据，对应用监控有很大帮助，例如prometheus生态，可以量化的实时展现应用视图；相应的，如果我们将metric统计数据强行使用针对log的管道来处理，将使我们丢失很多特性。

那么，在这里，我们可以开始对已知的系统进行分类。如：Prometheus， 专一的metric统计系统，随着时间推移，也许会进化为追踪系统，进而进行请求内的指标统计，但不太可能深入到log处理领域。ELK生态提供log的记录，滚动和聚合，并在其他领域不停的积累更多的特性，并集成进来。

另外，我发现通过维恩图的方式展现三者关系时，会正巧展现出一个附加效应。在这三个功能域中，metric倾向于更节省资源，因为他会“天然的”压缩数据。相反，日志倾向于无限增加的，会频繁的超出预期的容量。（有另一篇我写的关于这方面的文章，查看，译者注：未翻译）。所以，我们可以在图上，绘制出容量的需求趋势，metrics低到logging高， 而trace可能处于他们两的中间位置

![640?wx_fmt=png](https://ss.csdn.net/p?https://mmbiz.qpic.cn/mmbiz_png/gak2lhVxV6KBxvJXcYALaE5g3QIfS7tm41k5Zc0ehPAYKcic1Toic5FbiadXxXv8mibltZSp3qNhIEyE2LySNv5pfw/640?wx_fmt=png)

也许，这不是最完美的方式描述这三者的管理，但我从会议现场收到的反馈来看，这个分类还是相当不错的：随着三者的关系越清晰，我们越容易建设性的讨论其他问题。如果你尝试对产品的功能进行定位，你可能也需要这张图，在讨论中，澄清产品的位置。

