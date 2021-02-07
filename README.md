# 分布式系统原理 & 架构设计

该项目专注于分布式系统原理、分布式系统架构设计、业界支撑性分布式软件的介绍、学习。

包括作者在内的很多人，都有这样的困惑，“我想学习分布式系统的方方面面，我该从何处下手呢？”。我们很多开发人员在从业多年，也基于多年的开发经验建立起了对分布式系统的一些认识，但是也依然会对事情的分布式系统的原理、论证基础等感到好奇。

前几年，我主要专注于微服务框架的研究、go及go调试器的研究，现在希望能花些时间在分布式系统领域也有进一步的锻炼和成长。该项目收集、整理技术论坛上一些靠谱的分布式系统文章，总结提炼相关的原理，并结合业界相关软件的实现，来加深对原理的认识。

希望我们最后都能成为“独当一面”的技术帝，而不是一群只会“纸上谈兵”的家伙。

[TOC]

## 1 分布式系统相关的学习路线图

下列文章是[slack dis-sys](dis-sys@slack.com)推荐的学习路线图：

[distributed-systems-theory-for-the-distributed-systems-engineer](https://www.the-paper-trail.org/post/2014-08-09-distributed-systems-theory-for-the-distributed-systems-engineer/)


## 2 为什么分布式系统具有挑战性

下列文章对分布式系统中的一些挑战性难题进行了较为全面的介绍:

- [Distributed Systems for Fun and Profit](http://book.mixu.net/distsys/)
- [Notes on distributed systems for young bloods](http://www.somethingsimilar.com/2013/01/14/notes-on-distributed-systems-for-young-bloods/)
- [A Note on Distributed Systems](http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.41.7628)
- [The fallacies of distributed computing](http://en.wikipedia.org/wiki/Fallacies_of_Distributed_Computing)

### 2.1 Distributed Systems for Fun and Profit

该系列文章[Distributed Systems for Fun and Profit](http://book.mixu.net/distsys/)主要是介绍了分布式系统编程以及分布式系统的**核心概念**，方便读者后续针对性的深入了解。如果要包含分布式系统的方方面面，那是一件非常疯狂、工作量巨大的事情。

#### 2.1.1 [Basics](http://book.mixu.net/distsys/intro.html)

##### 2.1.1.1 水平扩展 vs 垂直扩展

每一个计算机系统都需要关注两方面的事情：**“计算” + “存储”**。
    
假如我们有足够的资金，当计算机硬件能力跟不上时，我们可以考虑升级不停地升级硬件（**垂直扩展，vertical scaling**），甚至是雇佣专门的团队来帮我们设计计算机，就不存在分布式系统的问题了。但我们的资源是有限的，技术发展是有瓶颈的，不可能一直对计算机做升级。

分布式系统，通过构建集群，通过添加新节点的方式来提升整体的处理能力（**水平扩展，horizontal scaling**）。水平扩展才是真正比较可行的解决方案。而且，集群中节点数量增多之后，节点之间的通信也会增多，这里的通信开销也会影响计算效率。所以需要研究高效的分布式算法来优化。

##### 2.1.1.2 分布式系统追求什么

- Scalability
    
    > is the ability of a system, network, or process, to handle a growing amount of work in a capable manner or its ability to be enlarged to accommodate that growth.
    
    困难大小源于问题规模。如果要处理的问题规模很小，那困难是微不足道的，比如让“举起”一块巧克力，但是让举起一座山，那就很困难了。当问题规模超过了一定的尺寸、容量或者物理条件限制时，解决问题就会变得非常困难。

    所以说，**everything starts with size - scalability**。在一个设计良好的分布式系统中，系统处理不会因为问题规模从小渐渐变大就变得很糟糕。

    **问题规模增长，指的都是哪些维度的增长**？有3个令人感兴趣的维度供参考。
    - Size scalability：系统集群中添加新节点会线性提升集群计算能力，待处理数据集、请求量增大后并不会明显增加系统处理时延；
    - Geographic scalability： 地理上会使用多地多机房的部署，如在天津、上海、深圳都有部署数据中心，来减少不同地域用户的请求时延。某些时延要求敏感场景下，也需要优化流量调度来减少跨城时延；
    - Administrative scalability：系统集群中添加更多节点不会明显引入额外的管理运维代价； 

    当然在真实的系统中，随着问题规模扩大，系统的“**growth**”是多维度的，不止上面这些。
    
- Performance (and latency)

    > is characterized by the amount of useful work accomplished by a computer system compared to the time and resources used.

     一个可扩展的系统随着规模扩大，依然能够满足用户需求。其中有两个比较重要的指标：performance 和 availability，这两个指标有各种各样的度量方式。关于performance，结合具体上下文，它可能代表了下面几个含义：

     - 针对特定的任务处理，有较短的响应时间（response time）或处理时延（low latency）；
     - 针对特定的任务类型，有较高的吞吐量（high throughput）或较快的处理速率（rate of processing work）；
     - 较低的资源占用率（low utilization of computing resources），可能隐含了更优的算法、逻辑；
     
     系统对这些指标的优化作优化时，涉及到一系列的权衡、折中（tradeoffs）。比如，系统可以通过处理较大型的作业也提升系统吞吐量，作业较小可能意味着人为的干预、会中断系统处理，但是较大型的作业的处理结束的响应时间也会变长，这里在响应时间和吞吐量之间做了折中。

     处理时延latency的含义，与响应时间差不太多：

     > The state of being latent; delay, a period between the initiation of something and the occurrence.

     在分布式系统中，latency不可能为0，如信息需要通过电信号传输，速度可以认为是光速，看上去很快，latency可以忽略不计？No，硬件有自己的物理特性，硬件操作都是有时延的，尽管看起来很小，但是这些时延会叠加起来……所以latency不可能为0或忽略不计。最终latency的大小，取决于任务处理的工作量大小以及信息传输的距离。
   

- Availability (and fault tolerance)

    > the proportion of time a system is in a functioning condition. If a user cannot access the system, it is said to be unavailable.

    分布式系统使得我们可以获得单机系统没有的特性，如容错。单机部署的服务如果挂掉了，是没法上线容错的，只能事后重启来应对。
    
    分布式系统中，其实应用了很多不可靠的组件，但是将这些组件结合起来却可以构建一个可靠的分布式系统。系统通过冗余（redundancy）可以实现部分容错，可以获得更好的可用性（availability）。

    冗余（redundancy），结合上下文，也可以是指代的不同的东西，如组件的冗余、server的冗余、数据中心的冗余、网络的冗余，等等。

    可用性的计算公式，很简单，`availability = uptime / (uptime + downtime)`，业界通常用“几个9”来表示可用性级别，通过不同可用性级别对应的每年停机时间，会获得一个非常直观的感受：

    |可用性级别     |每年停机/故障时间|
    |---------------|-----------------|
    |90%，1个9      |超过1个月        |
    |99%，2个9      |不足4天          |
    |99.9%，3个9    |不超过9个小时    |
    |99.99%，4个9   |不超过1个小时    |
    |99.999%，5个9  |不超过5分钟      |
    |99.9999%，6个9 |不超过31秒       |

    可用性的概念，不能仅仅停留于系统上线时间（uptime），更精准地描述，应该是系统能否正常工作、满足用户请求。这样的话要求就更高了，因为涉及到了响应时间等具体的硬指标。

    一个系统的可用性要达标，需要考虑方方面面的事情，系统走的网络、数据中心、应用的组件、本身部署的机房、依赖的外部API，等等，因为不能假定这些things都是可靠、不可能出问题的，所以在设计的时候最好就要考虑到冗余设计，来完成部分容错，不然不可能获得一个高可用的分布式系统。

    那么，容错（fault tolerance）指的是什么呢：

    > ability of a system to behave in a well-defined manner once faults occur

    要实现容错，前提是要知道错误或者故障的类型，然后才能设计对应的系统或者提出一种算法来在遇到该错误时进行处理、补偿。不可能对未考虑过的错误进行自动容错。

##### 2.1.1.3 上述目标为何难以实现

分布式系统，受限于两个物理上的因素：

- 系统中节点的数量（节点数量增加，提高了系统的存储容量和计算能力）；
- 节点之间的物理距离（信息传输需要时间，最好情况下是光速传输，但是硬件自己的电器特性还是有开销的）；

在这些约束基础上，我们再来看下：

- 系统中节点的数量增加，也意味着节点出现故障的概率更大了（任一节点正常工作概率为p (p<0)，则系统中出现故障节点概率为1-p^n），会降低可用性、增加运维成本；
- 系统中节点的数量增加，也增加了节点之间的通信开销（节点要与其他节点通信，通信请求数量多了），随着集群规模扩大，通信引入的开销会降低节点的计算性能；
- 系统中节点的数量增加，如果节点间地理位置增大了，节点间最小通信时延会增加，会降低系统处理系统或特定操作的响应时间；

    ![cluster size (number of cores)](http://book.mixu.net/distsys/images/barroso_holzle.png)

分布式系统设计，需要定实现的设计目标，设计实现时需要为了设计目标来做各种现实的权衡、折中。通常这里的设计目标，需要以各种保证条款的形式沉淀下来，如SLA（Service Level Agreement）。如，如果一次数据写操作成功，隔多久可以从其他地方访问最新数据？或者数据写入之后，数据持久性能保持多久？或者针对一次计算，隔多久可以计算完成返回结果？再或者说，如果一个组件失败，或者某个操作出问题，对系统会造成多大范围的影响、事故？

系统是代替人干活的，所以还有另一个层面的问题，系统的可理解性（intelligibility）。系统处理的结果、给出的信息、告警信息、错误信息等等是不是易于理解？有多么容易让人理解？

我们见过设计的很糟糕的系统，出点问题，经常需要拉起很多人、用很多时间、经过多番讨论、监控数据对比才能定位到是哪个server发生了问题、问题又是如何传播引起了整个系统的故障。对可理解性方面，没有明确的一些metrics可供度量、列出，但是它依旧是一个非常重要的维度。还有就是，一个系统的运行状态是否正常，是否能够做出判断，是否一定需要有经验的人才能做出诊断。这也是一些可以使用的metrics。

##### 2.1.1.5 抽象 & 建模

前面提到的这些问题，就需要通过一定的抽象和建模来解决。

- 抽象（abstractions），指的是对要解决的一些具体的问题，做一些褪去无关细节、提升思考维度做些通用性设计，以解决一类问题的方法；
- 建模（models），以更精确的方式来描述了一个分布式系统应该具备的各种关键属性；

    - System model (asynchronous / synchronous)
    - Failure model (crash-fail, partitions, Byzantine)
    - Consistency model (strong, eventual)

    ps: 关于建模的部分，请见本文下面的描述。

良好的抽象使使用系统更容易理解，同时能够捕获与特定问题相关的各种影响因素。

分布式系统中包含了很多节点，我们希望分布式系统能像一个单机系统那样易于理解，在这之间就存在一点冲突。比如，我们希望在一个分布式系统上实现一个共享内存的抽象设计，共享内存虽好理解，但是实现起来却很复杂。这样看来，常常我们单机系统中熟悉的模型，在分布式系统中实现起来会很复杂。

一个系统，如果对各种保证方面有更弱的保证，通常会获得更大的自由度，也会因此而获得更好的系统性能，但另一方面，也会变得更难难于理解。我们很容易理解一个单机系统中发生的事情，但是对于包含很多节点的分布式系统，要说清楚其中发生的事件则相对困难，尤其是一些异常、错误。

人们通常可以通过公开有关系统内部的更多细节来获得性能。 例如，在列式存储中，用户可以（在某种程度上）推断键值对在系统中的位置，从而做出影响典型查询性能的决策。 隐藏这些细节的系统更易于理解（因为它们的行为更像是一个单元，更少的细节需要考虑），而暴露更多真实细节的系统则可能更具性能（因为它们与现实的关系更紧密） 。

个别类型的故障使实现像单机系统一样工作的分布式系统变得非常困难。网络时延（network latency）和网络分区（network partition，total network failures between some nodes）意味着当系统中出现这类故障时，系统有时需要做出艰难的选择，以便更好地保持可用性，但失去了一些无法强制执行的关键保证，或者为了其安全性就拒绝了客户端请求。

下文中会提及CAP定理，来进一步解释这里遇到的一些问题。最后，理想的系统既可以满足程序员的需求（清晰的语义）又可以满足业务的需求（可用性、一致性、延迟等）。

##### 2.1.1.6 设计技巧：partition and replicate

分布式系统中有很多节点，节点要做计算就需要先知道待计算的数据是如何存储的、存储在哪里，数据集是如何存储的对分布式系统而言是非常重要的。

对于数据集存储，通常有两种方式：

- 分区（partition），数据集可以根据一定的规则做适当的分割（split）然后存储到不同的节点上；

    > is dividing the dataset into smaller distinct independent sets; this is used to reduce the impact of dataset growth since each partition is a subset of the data.

    分区把数据集按照特定split规则进行分割，如按照用户uid将不同用户信息切分开，然后存储到不同的节点上。相比完整数据集来讲，单一分区上的数据量大幅减小，有助于实现数据的最终存储，以及后续数据的高效查询。

    即便后续有的节点出现了失败，也知会影响该节点上存储的分区，并不会影响到整个的数据集，其他节点上存储的分区依然是可用的。并且这里出问题的节点，我们也可以进一步通过复制来增强可用性。    
    分区，和具体的数据集分割的规则有关系，是和应用场景有关系的，这里就先不多说了，我们将重点关注下面的复制。
     
- 复制（replication），数据集也可以被复制（copied）或缓存（cached）到不同的节点上，以减少为了访问其他节点存储的数据引入的通信时延，也可以实现更好的容错，避免节点故障引入的数据丢失；
    
    > copying or reproducing something - is the primary way in which we can fight latency.

    复制，为数据生成了额外的副本，提高了容错能力，提高了系统的可用性。

    复制，为数据生成了额外的副本，存储这些副本的节点也可以用来对外提供服务，这样有提升了系统的整体性能。

    其实，复制，也是一种非常常见的提升系统处理性能、降低处理时延（latency）的方式。

    复制，提供了额外的带宽（bandwidth），复制在延时敏感领域可以通过缓存降低数据访问时延，复制和维护数据一致性也息息相关（如WAL，Write Ahead Log）。

    复制，允许我们获得更好的扩展性、性能、容错能力，有了复制，就不用那么担心可用性受损或者数据访问性能下降的问题，可以通过复制数据到其他节点来降低系统访问瓶颈和数据单点故障。复制的概念可以进一步延伸，如复制计算，担心计算性能问题，将计算复制到多个系统，采用分布式计算的方式。担心慢速IO，将数据复制到本地cache来降低延时，或者复制到多机来提升数据访问的吞吐量。

    另外，也要意识到，复制也是造成一些难解问题的原因，比如数据一致性问题。数据复制到多个节点，如何维护这些数据副本的一致性，在分布式领域是一个非常重要的问题，**数据一致性模型**的选择对维护数据一致性至关重要。好的一致性模型即具有清晰的语义（如方便理解）也能满足业务要求（如数据强一致、高可用等）。
    
    对于复制，只有强一致模型才能够像对待没有数据复制一样轻松地编码实现。其他一致性模型，都或多或少暴露了一些数据复制的细节信息给开发人员，开发人员需要理解这样的复制存在的问题，如数据不总是一致的，需要借助一些其他手段来识别数据是否ok，编码上就没那么简单。但是，弱一致性模型，因为约束少、自由度大，往往可以获得更好的系统性能、更高的可用性，当然也并不是那么难以理解。
    
    业界存在一些经过验证的一致性算法如Paxos、Raft、ZAB等等，这些一致性算法既需要考虑如何维护数据一致性，也需要考虑降低维护一致性的开销、保证分布式系统的处理性能。


下图中展示了分区、复制二者之间的一个区别，图中数据集A和B是以分区的形式存储到了不同节点Node1和Node2上，数据集C则是以复制的方式在Node1、Node2上进行了存储。

![parition and replicated](http://book.mixu.net/distsys/images/part-repl.png)

不同的分布式系统可能会选择不同的数据存储策略，可能是分区，也可能是复制，更常见的是两者的集合，数据集会分区存储到不同节点，同时也会在其他节点生成数据的副本。现在需要了解的是，分区、复制这两种方式，它们各有优缺点，分区、复制也有不同的算法来支持，需要结合具体的设计目标来选择和评估。


参考资料：
- [The Datacenter as a Computer - An Introduction to the Design of Warehouse-Scale Machines](http://www.morganclaypool.com/doi/pdf/10.2200/s00193ed1v01y200905cac006) - Barroso & Hölzle, 2008
- [Fallacies of Distributed Computing](http://en.wikipedia.org/wiki/Fallacies_of_Distributed_Computing)
- [Notes on Distributed Systems for Young Bloods](http://www.somethingsimilar.com/2013/01/14/notes-on-distributed-systems-for-young-bloods/) - Hodges, 2013

#### 2.1.2 [Up and down the level of abstraction](http://book.mixu.net/distsys/abstractions.html)

#### 2.1.3 [Time and order](http://book.mixu.net/distsys/time.html)

#### 2.1.4 [Replication: preventing divergence](http://book.mixu.net/distsys/replication.html)

#### 2.1.5 [Replication: accepting divergence](http://book.mixu.net/distsys/eventual.html)

#### 2.1.6 [Appendix](http://book.mixu.net/distsys/appendix.html)
