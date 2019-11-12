# Raft理解

看Raft论文的过程中，有些困惑的地方，我会以提问的形式先记下来，然后尝试回答自己的问题，这里做一个整理。

## 选举

### term的作用

共识算法大都基于复制状态机模型，client的请求会被存储为一系列有序的日志。leader会把这些日志同步到集群中的其他节点上。假设有一个非常精确的时钟，每个请求被append到日志中的时间戳都不一样，则这些日志的顺序性由这个
时间戳保证，复制到其他节点时可以通过时间戳确定是否有序。现实中，节点会有时间上的精度和误差问题，Raft没有使用物理上的时钟，而是使用term当做其逻辑时钟，term递增，同一个term的log index也递增，这样通过term和index即可确定日志的有序性。

### 一个节点的term会在什么情况下进行更新

- 1.它发起一轮选举,会自增自己的term。
- 2.它通过其他节点了解自己的term已经过期，更新自己的term。

### 节点丢失连接对term的影响

一个节点与其他节点失去连接后，它会不断发起选举请求，并递增自己的term，过一段时间后，它的term'会比其他的节点都要大一些。节点回到集群中后，其他节点会把currentTerm更新为term’，state更新为Follower，VotedFor设置为null。当term改变的时候，votedFor必须改变，或者保持null。集群会重新发起选举， 如果它的log足够新，它有可能被选举为leader。

### 选举结果的影响

- 1.赢得选举成为leader
- 2.失败，重新发起选举
- 3.通过其他节点认识到自己的term已经过期，转为Follower。

### 怎么保证一个term最多只能有一个leader

一个Follower在一个term中只能投票给一个节点，并且投票信息会被持久化。

### 多个节点发起相同term的选举

一个Candidate发起选举后，会首先投票给自己（votedFor=self），当收到对方相同term的投票请求后，根据一个节点在一个term之后投票给一个节点的约束，会拒绝这个请求。

### 重置选举超时timer的时机

对于Follower和Candidate来说，无论何时，只要收到合法的AppendEntries和RequestVote请求（不包括过期term的请求），应该重置选举超时timer：因为收到这两个请求，可以认为集群中存在leader（虽然不一定存在leader）。

### 为什么只有在收到合法的投票请求时才需要重置选举timer

在不可靠的网络环境，每个server的log都不一样，能满足选举为leader的节点很少。如果任意一个投票请求都重置选举timer，会导致少数满足选举条件（log up-to-date）的节点，被多数不满足选举条件的节点发起的选举请求中断，推迟选举进程，甚至可能在比较长的时间内一直没能选举出leader。相反，如果对于log out-of-date节点的投票请求，不重置选举timer，则log up-to-date节点的选举过程不会被打断，能更快完成选举成为leader。

### 心跳检查

收到心跳发出的AppendEntries时，即使没有携带任何数据，也要做log的检查。

### log up-to-date检查的逻辑

论文中对log是否up-to-date的定义为：
>If the logs have last entries with different terms, then the log with the later term is more up-to-date. If the logs end with the same term, then whichever log is longer is more up-to-date.

为什么up-to-date的检查仅仅检查term和log的长度，不需要检查log是否一样？term是log逻辑上的时钟，如果term不一样，直接可以判断term更大的log更新。如果term一样的话，根据复制的性质，相同位置的log内容一定是一样的，直接判断长度就行了。

## 持久化

### 为什么持久化term

term必须严格递增，必须持久化。此外，还可以通过比较term来判断RequestVote和AppendEntries请求是否过期。

### 为什么要持久化votedFor

Raft选举要求一个term最多只能投票给一个节点，如果不持久化votedFor信息，可能会出现一个term投票给多个节点的情形，会影响Raft算法的正确性。比如在一个term内，一个节点先投票给了A，然后reboot了，如果没有持久化，可能又投票给了B，这样会出现A和B两个节点同时获得了大多数的投票，导致出现两个leader。

### 为什么持久化日志

类似事务，Raft commit了日志就保证日志永久的被持久化，也就是说commitIndex只能递增，不能回退。而日志提交位置是leader通过复制到大多数节点的最大nextIndex决定，如果log没有持久化（某个节点reboot导致log丢失），则会影响对提交日志的判断，已经提交的日志很可能会丢失。

### 为什么不持久化commitIndex

commitIndex leader可以通过matchIndex[]得到，不需要持久化。
