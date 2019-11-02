# Trepang
Trepang is an implementation of Raft Algorithm in Go


## 资料
总结学习和实现Raft过程中用到的一些资料。

**[Raft论文](https://raft.github.io/raft.pdf)**
> 广为流传的Raft精简版论文。这篇论文主要介绍了Raft的工作原理，简单提及但没有详细介绍Cluster Membership changes和Log Compaction高级主题。

**[Raft算法作者ongardie的博士论文](https://github.com/ongardie/dissertation/blob/master/book.pdf?raw=true)**
> Raft作者的博士论文，250多页，精髓在前70页，包含了精简版论文的内容，同时又详细介绍了Cluster Membership changes和Log Compaction高级主题。从70页开始，主要跟Raft实践有关：Raft的易学习性、选举超时时间评估、性能等。

**[Raft官网](https://raft.github.io/)**
>Raft官网，汇总了一些资料，大部分都是一些其他人介绍Raft的talk，内容重复又不系统，没有太多借鉴的意义，网站下面介绍了Raft的各种实现。两个可视化工具比较有用，[工具一](http://thesecretlivesofdata.com/raft/)，介绍了Raft大概工具原理，比较简单，主要了解概念。[工具二](https://raft.github.io)，也就是Raft官网的Raft Visualization部分，模拟5个节点Raft集群的情况。可以手动操作让节点宕机、复制数据，让请求超时、丢失等，这里对于了解Raft的选举、复制机制比较有用。

**[Fault-Tolerant Virtual Machines](https://pdos.csail.mit.edu/6.824/papers/vm-ft.pdf)**
> 与Raft没有太大的关系，可以加深了解复制状态机模型(RSM,Replicated State Machine)模型。Raft是基于应用层的RSM，这篇文介绍了VM层面的RSM实现，通过机器指令级别的复制，一个虚拟机的状态被完整地复制到另外一台虚拟机上。最终，两台虚拟机执行一样的指令，每条指令又产生相同的结果，即使是随机函数。

**[MIT6.824课程](https://pdos.csail.mit.edu/6.824/schedule.html)**
> MIT的分布式课程，比较经典。读论文+实践的方式授课。先介绍了MapReduce，然后实现Raft，接着在Raft基础上实现一个分布式KV系统，实现是用go。这个课程助教写了一篇文章对于Raft实现常见的坑做了一个整理：[Students' Guide to Raft ](https://thesquareplanet.com/blog/students-guide-to-raft/)。

**[Tidb的博客](https://pingcap.com/blog-cn/#Raft)**
> Tidb是基于Raft实现，看过几篇博客，有工程上的借鉴意义，但还没有仔细研究。


