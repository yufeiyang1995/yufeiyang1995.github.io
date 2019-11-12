---
layout:     post
title:      Zookeeper简介，原理及应用
category: blog
description: 分布式这么有用了，学习一下～
---

## Zookeeper简介，原理及应用

之前在实习的时候用到了Elasticsearch，看了看它内部的实现原理，发现ES自己实现了一套分布式一致性解决方法，觉得还挺有意思的。学习过程中很多资料都提到了Zookeeper，想起来大三的时候就在hadoop里看到的Zookeeper组件但是一直不知道它是干嘛的，最近不是特别忙就研究一下Zookeeper。

### 简介

ZooKeeper是一个高可用的一致性协调框架，用来为分布式应用提供一致性服务的软件，提供的功能包括：配置维护、域名服务、分布式同步、组服务等。这些功能实现起来非常复杂，但Zookeeper可以提供给开发者性能高效，功能稳定的服务。

### Zookeeper原理

ZooKeeper为了高可用的一致性协调，实现了一种通用的一致性算法ZAB（ZooKeeper Atomic Broadcast）

ZAB是在Fast Paxos算法基础上进行了扩展改造而来的，另一种一致性算法Raft也和ZAB非常类似，所以为了弄懂Zookeeper实现原理，我们比较这三种分布式一致性算法（Paxos，raft和ZAB）的异同：

#### Paxos

首先，我们来介绍一下Paxos算法，Lamport大神在1990年提出的这个基于消息传递且具有高度容错特性的一致性算法，他提出了三种Paxos算法：Basic Paxos，Multi Paxos和Fast Paxos。Basic Paxos只能对一个值形成决议，决议的形成至少需要两次网络来回，所以一般只用来进行理论研究；Multi Paxos和Fast Paxos则对它进行一系列的改动，使更适合直接应用在实际项目中。

**Basic Paxos**

Paxos算法运行在允许宕机故障的异步系统中，不要求可靠的消息传递，可容忍消息丢失、延迟、乱序以及重复。它利用大多数 (Majority) 机制保证了2F+1的容错能力，即2F+1个节点的系统最多允许F个节点同时出现故障。

Paxos将系统中的角色分为提议者 (Proposer)，决策者 (Acceptor)，和最终决策学习者 (Learner)：

* **Proposer**: 提出提案 (Proposal)。Proposal信息包括提案编号 (Proposal ID) 和提议的值 (Value)。

* **Acceptor**：参与决策，回应Proposers的提案。收到Proposal后可以接受提案，若Proposal获得多数Acceptors的接受，则称该Proposal被批准。

* **Learner**：不参与决策，从Proposers/Acceptors学习最新达成一致的提案（Value）

Paxos算法通过一个决议分为两个阶段：

* Prepare阶段：Proposer向Acceptors发出Prepare请求，Acceptors针对收到的Prepare请求进行Promise承诺

* Accept阶段：Proposer收到多数Acceptors承诺的Promise后，向Acceptors发出Propose请求，Acceptors针对收到的Propose请求进行Accept处理。

  | Proposer                                                     | Acceptor                                                     |
  | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | 1. 选择一个proposal number n                                 |                                                              |
  | 2. 广播Prepare(n)和Value给所有的Accepter                     |                                                              |
  |                                                              | 3. 回复Prepare(n):<br />如果n > minProposal，则minProposal = n，同时将acceptedProposal和acceptedValue返回 |
  | 4. Proposer接收到过半数回复后，如果发现有acceptedValue返回，将所有回复中acceptedProposal最大的acceptedValue作为本次提案的value，否则可以任意决定本次提案的value；<br />如果没超半数，则重新发起Prepare请求 |                                                              |
  | 5. 广播Accept(n, value)给所有Accepter                        |                                                              |
  |                                                              | 6. Acceptor比较n和minProposal<br />如果n>=minProposal，则acceptedProposal=minProposal=n，acceptedValue=value，本地持久化后，返回；否则，返回minProposal |
  | 7. Proposer接收到超半数的成功回复后，将结果通知给所有Learner；如果没到半数，则重新发起Prepare请求 |                                                              |

**Multi Paxos**

原始的Paxos算法只能对一个值进行决议，而且每次决议至少需要两次网络来回，在实际应用中可能会产生各种各样的问题，所以不适合直接应用在实际工程中。因此，Multi Paxos解决了这一问题，可以连续确定多个值并提高效率，基于Basix Paxos主要做了如下改进：

* 针对每一个要确定的值，运行一次Paxos算法实例（Instance），形成决议。每一个Paxos实例使用唯一的Instance ID标识。
* 在所有Proposers中选举一个Leader，由Leader唯一地提交Proposal给Acceptors进行表决。这样没有Proposer竞争，解决了活锁问题。在系统中仅有一个Leader进行Value提交的情况下，Prepare阶段就可以跳过，从而将两阶段变为一阶段，提高效率。

所以Multi Paxos的实际过程是：首先进行Leader选举，利用Basic Paxos来实现。选出Leader后，只由Leader来提交Proposal，如果Leader出现宕机，则重新选举Leader，在系统中仅有一个Leader可以提交Proposal。

Multi Paxos通过改变Prepare阶段的作用范围至后面Leader提交的所有实例，从而使得Leader的连续提交只需要执行一次Prepare阶段，后续只需要执行Accept阶段，将两阶段变为一阶段，提高了效率。

**Fast Paxos**

在之前提到的Paxos协议中，消息最后到达Learner一般都要经历 Client-->Proposer-->Acceptor-->Learner 总共3个步骤；为了更快的让消息到达Learner，可以跳过Proposer这一步，直接将请求发送给Accepter，由Leader在中间进行检查。

* Leader检测到冲突之后，根据规定的算法从冲突中选择一个数据，重新发送Accept请求。
* 当检测到冲突的时候，如果Acceptors自己就能解决冲突，那么就完全不需要Leader再次发送Accept请求了，这样就又减少了一次请求，节省了时间

#### ZAB

基于Paxos，Zookeeper实现了ZAB算法，个人感觉ZAB和Multi Paxos更像一点。

ZAB的算法流程为：

- Leader election：leader选举过程，electionEpoch自增，在选举的时候lastProcessedZxid越大，越有可能成为leader

- Discovery：

  - （1）leader收集follower的lastProcessedZxid，这个主要用来通过和leader的lastProcessedZxid对比来确认follower需要同步的数据范围
  - （2）选举出一个新的peerEpoch，主要用于防止旧的leader来进行提交操作（旧leader向follower发送命令的时候，follower发现zxid所在的peerEpoch比现在的小，则直接拒绝，防止出现不一致性）

- Synchronization：

  follower中的事务日志和leader保持一致的过程，就是依据follower和leader之间的lastProcessedZxid进行，follower多的话则删除掉多余部分，follower少的话则补充，一旦对应不上则follower删除掉对不上的zxid及其之后的部分然后再从leader同步该部分之后的数据

- Broadcast

  正常处理客户端请求的过程。leader针对客户端的事务请求，然后提出一个议案，发给所有的follower，一旦过半的follower回复OK的话，leader就可以将该议案进行提交了，向所有follower发送提交该议案的请求，leader同时返回OK响应给客户端

#### Raft

Raft算法是因为该作者觉得Paxos太难以理解了，所以就提出了Raft，一定程度上也可以看作是Paxos的一种改进。它强化了leader的地位，把整个协议可以清楚的分割成两个部分，并利用日志的连续性做了一些简化： （1）Leader在时。由Leader向Follower同步日志 （2）Leader挂掉了，选一个新Leader，Leader选举算法。

算法演示：（这个演示很简单明白，就不用文字讲了）

[Raft](http://thesecretlivesofdata.com/raft/)

#### 主要区别

由于Paxos偏理论，Raft和ZAB又都是基于他实现的，他们在整体思路上有一些地方还是挺像的，但细节上有些不同，所以我们主要比较这两种算法，下面是我总结的几个区别：

* 选举过程
  * 基本实现：
    * ZooKeeper：在每次leader选举完成之后，都会进行数据之间的同步纠正，所以每一个轮次，大家都日志内容都是统一的；
    * Raft：在leader选举完成之后没有这个同步过程，而是靠之后的AppendEntries RPC请求的一致性检查来实现纠正过程，则就会出现上述案例中隔了几个轮次还不统一的现象
  * 加入已完成选举集群：
    * ZooKeeper：该server启动后，会向所有的server发送投票通知，这时候就会收到处于LOOKING、FOLLOWING状态的server的投票（这种状态下的投票指向的leader），则该server放弃自己的投票，判断上述投票是否过半，过半则可以确认该投票的内容就是新的leader。
    * Raft：比较简单，该server启动后，会收到leader的AppendEntries RPC,这时就会从RPC中获取leader信息，识别到leader，即使该leader是一个老的leader，之后新leader仍然会发送AppendEntries RPC,这时就会接收到新的leader了（因为新leader的term比老leader的term大，所以会更新leader）
* 上一轮次的Leader
  * 处理策略
    * Zookeeper：采取激进的策略，对于所有过半还是未过半的日志都判定为提交，都将其应用到状态机中
    * Raft：对于之前term的过半或未过半复制的日志采取的是保守的策略，全部判定为未提交，只有当当前term的日志过半了，才会顺便将之前term的日志进行提交
* 异常处理
  * Follower挂了
    * 
  * Leader挂了

### Zk基本特性

Zookeeper在设计了一套分布式一致性算法后，提供了一系列的基本特性，供开发者使用：

* 构建高可用分布式集群
  * Zookeeper的分布式一致性算法保证了集群的高可用性
* 集群配置统一管理
  * Zookeeper的数据模型类似文件系统，可以方便的进行集群配置的管理
* 发布和订阅
  * 支持服务的发布和状态监听机制
* 分布式锁
  * Zookeeper利用选举模式和数据模型可以实现分布式锁，保证分布式系统的同步
* 数据的强一致性
  * 基于分布式一致性算法进行保证，当数据修改之后会同步到其他备份上

### Zk主要应用

* Hadoop
  * 分布式锁，保证只有一个ResourceManager创建成功
  * 监听状态
  * 状态存储
* Hbase
  * 利用Zookeeper主从备份，状态的保存和监控，实现系统容错
* Kafka
  * 注册节点管理服务器
  * 注册多节点，实现负载均衡
* Dubbo
  * 用于服务注册