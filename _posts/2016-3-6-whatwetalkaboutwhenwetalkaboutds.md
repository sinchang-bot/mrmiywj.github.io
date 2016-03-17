---
layout: post
category: distributedsystem
comments: true
title: 当我们在谈论分布式系统的时候我们在谈论什么[译]
tags: distributedsystem system
---
本文是我在阅读了Alvaro Videla先生的博客之后，受益良多，决定翻译下来。也征得了Videla先生本人的同意，原文链接在此 http://videlalvaro.github.io/2015/12/learning-about-distributed-systems.html


在过去的相当长一段时间内，我尝试着学习分布式系统，并且可以说，一旦你开始挖掘这方面，你会发现这一方向看起来是没有止境的，深坑一个接着一个。分布式系统中的话题是相当广泛的，伴随着一篇又一篇各个大学发出的论文，和一些书籍。事实证明，对于一个像我这样的萌新，一开始是很难抉择到底要读哪些论文，看哪些书的。

同时，我也发现有很多博客推荐了这样或者那样的对那些想要成为分布式系统工程师（不管这个词的含义是什么）的人们必读的论文。下面是列举的论文：

[FLP](http://cs-www.cs.yale.edu/homes/arvind/cs425/doc/fischer.pdf),
[Zab](http://web.stanford.edu/class/cs347/reading/zab.pdf),
[Time, Clocks and the Ordering of Events in a Distributed Systems](http://research.microsoft.com/en-us/um/people/lamport/pubs/time-clocks.pdf),
[Viewstamped Replication](http://pmg.csail.mit.edu/papers/vr.pdf),
[Paxos](http://research.microsoft.com/en-us/um/people/lamport/pubs/lamport-paxos.pdf),
[Chubby](http://static.googleusercontent.com/media/research.google.com/en//archive/chubby-osdi06.pdf)

我的问题在于很多时候我没有看到有一个理由来告诉我 _为什么_ 我应该读这篇或者那篇论文。我热爱为了满足好奇心仅仅是为了知识而学习的想法，但是同时应该考虑到阅读的先后顺序，毕竟一天只有24小时。

除了大量的论文和研究资料，就像上面所提到的，还有很多书籍。在借阅了相当多的书籍之后，我开始发现一本书的标题并不能够满足我在寻找的问题或者说它的内容并不能解决我想要解决的问题。因此，我想我想仔细讨论一下我所认为的分布式系统的最主要的概念，并且给出引用告诉你应该去哪里学习他们。

由于我是一边学习一边写下这些文字的，所以请有点耐心，我也可能会犯下一些错误，并且我会尝试解释一切我所写下的内容。

在我们开始前，我必须得告诉你我写这篇文章时参考了大量引用，所以这儿有些slides如果你感兴趣的话。

<p>
<script async class="speakerdeck-embed"
data-id="5f16b66144bb4d0ba5aac87488efecf6" data-ratio="1.7"
src="//speakerdeck.com/assets/embed.js"></script>
</p>

并且这儿有段我在Erlang User Conference in Stockholm 的演讲视频。

<p>
<iframe width="560" height="315" src="https://www.youtube.com/embed/yC6b0709HCw" frameborder="0" allowfullscreen></iframe>
</p>

让我们开始吧。

分布式系统算法可以根据不同的性质进行分类。比如说根据时序模型，根据进程间的通信方式，根据这个算法所使用的故障模式，还有很多很多，我们接下来也会看到。

接下来我们将看到的主要概念有：

- 时序模型 (Timing Model)
- 进程间通信 (Interprocess Communication)
- 故障模式 (Failure Modes)
- 故障检测器 (Failure Detectors)
- 领袖选举 (Leader Election)
- 一致性 (Consensus)
- 法定人数 (Quorums)
- 分布式系统中的时间 (Time in distributed systems)
- FLP 浏览
- 总结
- 引用

## 时序模型 (Timing model)

接下来我们要讨论 _同步模型(synchronous model)_, _异步模型(asynchronous model)_ 和 _部分同步模型(partially synchronous model)_ 。

**同步模型**是最简单的一种。每个部分通过所谓 _同步轮次(synchronous rounds)_ 进行同步的工作。 消息传递所需要的时间通常是已知的，并且我们人为假定一个进程的速度，比如一个进程要花多久才能进行算法的一步。这个模型的问题在于它并没有真实的反映现实情况，甚至说在一个分布式系统中，一个进程可以给另一个进程发送消息，并且祈祷星星是对齐的（大概意思是运转良好），所以消息可以抵达上述进程，在这样的分布式系统中描述的也不是很好。这个模型的好处在于使用这个模型可以得到理论解然后转换到其他模型中。例如说，因此模型对于时间的保证，我们可以发现有个问题在时序保证的情况下也无法解决，那么很可能在我们放松了条件的情况下也无法解决。

**异步模型** 相比而言复杂一些。 每个部分可能执行算法的每个步骤的顺序无法保证，并且他们无法确定他们执行算法每一步的速度。这个模型的问题在于虽然它的描述很简单并且相当接近现实，但是它始终是无法准确反映现实世界的。例如，一个进程可能要花无限长的时间来响应一个请求，但是在现实的项目中，我们很可能会在上述请求中强加一个时限，一旦到了时间就会停止那个请求。这个模型的一个难点就在于，我们如何能够确保一个行程的存活情况。最出名的 _不可能情况_ 可以参见这里，["Impossibility of Consensus with one Faulty Process"](http://cs-www.cs.yale.edu/homes/arvind/cs425/doc/fischer.pdf)

在**部分同步模型**中，每个部分有一些关于时序的信息，可以保证近似的同步时钟，或者他们可能知道消息传递的近似时间，或者一个进程执行算法每一步所需的近似时间。

[Distributed Algorithms](http://www.amazon.com/Distributed-Algorithms-Kaufmann-Management-Systems/dp/1558603484) Nancy Lynch的这本书有关于时许模型的部分。

## 进程间通信(Interprocess communication)

接下来我们考虑进程是 _如何_ 在系统中交换信息的。在**消息传递(message passing)**模型中，它们可以通过传递消息来完成。在**内存共享(shared memory)**模型中，它们可以通过共享变量来完成。

需要在心里牢记一件事——我们可以通过消息传递算法来建立一个共享内存对象。一个在很多书中都有的碱性粒子就是读/写寄存器的实现。我们也可以用队列或者栈来保证连续性的性质，[linearizabilty](https://cs.brown.edu/~mph/HerlihyW90/p463-herlihy.pdf)。我们不应该混淆通过一个共享变量来在进程之间 _共享内存_ 和建立在消息传递基础上的 _共享内存抽象_ 。

回到消息传递模型，我们可以通过另一种抽象来尝试理解算法：进程之间建立联系的方法（想想进程间传递消息的管道）。这些链接保证了算法使用它们时的稳定。例如，有种抽象叫 _完美链接(Perfect Links)_ 可以有可靠的传递并且不会重复。这种抽象也保证了一次传递。这种抽象很显然没反映真实世界的情况，因此算法设计者们设计了许多其他模型，更加贴近真实的系统。虽然 _完美链接(Perfect Links)_ 并不是如此现实，但是它依旧十分有用。例如如果我们可以证明一个问题即使在完美链接的情况下也无法解决，那么我们可以知道许多相关问题也可能无法解决。关于链接的问题许多作者通常认为或者假定了FIFO消息模型，[Zab](http://web.stanford.edu/class/cs347/reading/zab.pdf)

## 故障模式(Failure Modes)

我曾经写过一边关于故障模式的文章[failure modes in distributed systems](http://videlalvaro.github.io/2013/12/failure-modes-in-distributed-systems.html)它依旧值得重新多读几遍。一个分布式系统的重要特征就是它嘉定了何种故障模式。在 _故障-停止(crash-stop)_ 模型中，一个进程总是被认为是正确的，直到它发生了故障。一旦它挂了，它就真挂了，不重启了。也有一种模型叫做_故障-重启(crash-recovery)_模型，发生错误后进程将会冲洗。在这种情况下，算法包括了一种能够让进程恢复到它挂之前的状态的途径。有多种方法，比如说从一个持续的储存中读取，或者与在一个组中的其他进程进行交流。如果对某些算法来说，一个进程挂了再恢复就不算做之前的同一个进程了的话，这个算法就是没用的。这通常取决于我们是用动态分组还是静态分组。


也有些故障模式中，进程是无法接受或者传递消息的，这种被称为 _疏漏故障模式(omission failure modes)_ 当然，除了这种，还有很多其他类型的疏漏。为什么这很重要？想象这样的场景，一组进程组成了一个分布式的缓存，如果一个进程没能够回复其他进程的请求，即使它能够接收到请求并且保证了它的状态是最新的，所以它依旧可以听取客户端的读取请求。

Byzantine，一个更复杂的故障模式，或者说叫_arbitrary failures mode_。在这种模式中进程可能给它的同伴发出错误的信息。他们可能模仿或者冒充其他进程，或者说给其他进程传递了正确的数据，但是弄乱了它自己本地的数据内容。

当考虑一个系统的设计时，我们应该考虑我们想要解决怎样的进程故障。Birman([Guide to Reliable Distributed Systems](http://www.amazon.com/gp/product/1447124154)) 提出了我们通常不需要结局Byzantine 故障的观点。他根据在Yahoo的工作他们总结出Crash failure 总是比Byzantine failures更常见。（译者注：意思就是，进程本身挂掉远比进程间出现欺骗频繁的多）

## 故障检测(Failure Detector)

根据进程的故障模式和时限假定，我们可以构建出一个能够向系统报告是否有一个进程挂掉或者它可能挂掉的抽象。_完美检测器(Perfer Failure detector)_ 能够永远不给出误报。对Crash-stop模式和同步模型，我们可以仅仅通过添加时限来完成这样的算法。如果我们让进程周期性的去ping故障检测进程，我们就可以知道何时一个ping到达故障检测其（同步模型保证了这一点）。如果ping没能在实现设定的时限中到达，那么我们就可以认为其他节点挂了。

在一个更加实际的系统中，也许你不可能总是假设一个消息到达目标的时间，或者一个进程执行算法每一步所需的时间。在这个例子中我们有个进程检测`p` 将会报告一个进程`q`如果在`n`毫秒后还不答复的话就报告它是被 _怀疑_ 的。如果`q`之后答复了，那么`p`就会把`q`从怀疑进程列表中移除来，并且增加`N`，因为它并不知道它本身与`q`之间的网络延迟，但是它想要停止怀疑`q`是挂了的，因为`q`依旧活的好好的，只不过是ping回来的时间比`N`长了点。如果`q`挂了，那么`p`就会首先怀疑它是挂了的，然后就再也不修改这样的看法了。这种算法的更好的描述可以详见[Introduction to Reliable and Secure Distributed Programming](http://www.amazon.com/Introduction-Reliable-Secure-Distributed-Programming/dp/3642152597/)

故障检测器通常提供了两种兴趣：完备性和准确性。对于总是完美的故障检测来说，我们有如下性质：

- **强完备**每个挂掉的程序总能被发现
- **强正确**没有正确的进程被怀疑

故障检测对于解决在异步模型中的一致性是非常重要的。非常重要的的不可能性结果可以在这篇论文中看到[FLP](http://cs-www.cs.yale.edu/homes/arvind/cs425/doc/fischer.pdf)。这篇论文讨论了在进程可能出故障的分布式异步模型中的一致性的不可达性。可以通过这篇论文来了解。[circumvent the problem](http://www.cs.utexas.edu/~lorenzo/corsi/cs380d/papers/p225-chandra.pdf).

## 领袖选择(Leader Election)

与故障检测的问题相反，为了确定哪个进程没有挂并且工作良好。这种进程将被网络中的其他伙伴新人并且被认为是可以协调分布式的行动的领导者。这种就是类似于[Raft](https://raft.github.io) or
[Zab](https://web.stanford.edu/class/cs347/reading/zab.pdf)提出里的基础领导者的分布式算法。

在协议中有一个领导者可以在节点中产生不对称性，因为那些不是领导者的节点将会成为追随者。这将会导致领导者节点最终成为许多操作的节点，因此基于我们试图解决的问题，使用一个需要领导者选举的算法也许不是我们想要的。需要注意的是大部分协议都是通过一致推出一个领导者来达到一致性。

[Paxos](http://research.microsoft.com/en-us/um/people/lamport/pubs/lamport-paxos.pdf),
[Zab](https://web.stanford.edu/class/cs347/reading/zab.pdf) or
[Raft](https://raft.github.io) for some examples.

## 一致性(Consensus)

一致性问题最先是在这篇文章中[Reaching Agreement in the Presence of Faults](http://research.microsoft.com/en-us/um/people/lamport/pubs/reaching.pdf)由Pease, Shostak 和Lamport 提出的，他们是这样介绍这个问题的：

> 容错系统(Fault-tolerant systems)经常需要在各个独立的进程和核心能够达到某件事的统一意见取得通均值。例如，在一个冗余系统中的处理器想要周期性同步它们内部的始终。或者它们也许得需要在一个各个核心看来数值略有区别的传感器读值上保持一致。

所以一致性是在各个独立部分之间保持统一的问题。这些进程对于某个问题会有不同的数值，例如像他们传感器的当前度数，然后得基于他们提出的数值达到一个统一的行为。例如，一辆车也许会就提供突破温度等级信息有许多传感器。这些传感器读数因为精度会有所不同，但是ABS电脑需要知道到底多大的压力来完成突破。这就是一个我们每天都得面对的一致性问题。[Fault-Tolerant Real-Time Systems](http://www.springer.com/us/book/9780792396574)这本书中解释了自动化工业中的一致性还有其他问题。

一个实现了某种一致性形式的进程通过 _提出(propose)_ 和 _决定(decide)_ 的api来工作。当一致决定开始时，一个进程将提出一个值并且在整个系统中所有值的基础上，决定最后的值。这种算法将满足如下性质：

- **Termination**: 每个正确的进程总会最终决定一些值
- **Validity**: 如果一个进程决定了 _v_ , 那么 _v_ 肯定是由某个进程提出的
- **Integerity**: 没有进程会决定两次
- **Agreement**: 没有两个正确的进程最后决定的值不同。

更多细节可以参照下面的论文：

- [Introduction to Reliable and Secure Distributed Programming](http://www.amazon.com/Introduction-Reliable-Secure-Distributed-Programming/dp/3642152597/), Chapter 5.
- [Fault-tolerant Agreement in Synchronous Message-passing Systems](http://www.amazon.com/Fault-tolerant-Agreement-Synchronous-Message-passing-Distributed/dp/1608455254/).
- [Communication and Agreement Abstractions for Fault-tolerant Asynchronous Distributed Systems](http://www.amazon.com/Communication-Abstractions-Fault-tolerant-Asynchronous-Distributed/dp/160845293X/).


## 法定人数(Quorums 他妈的我实在不知道这个怎么翻译）

Quorums 是一种用来设计容错分布式系统的工具。 Quorum是指一系列用来理解系统的行为的想交的进程，因为有些进程可能会失效。

例如说，如果我们有一个使用N个使用Crash-failure模式的进程的算法，我们有一组法定的进程不管何时只有大部分的进程才能对系统进行特定操作，比如说写数据库。如果小部分的进程挂了，例如`N/2 - 1`个进程挂了，那么我们就有一个大部分的进程知道对系统的最后的操作是什么。比如说,Raft在记录系统日志的时候就采用了多数性。当系统中有一半的服务器记录下日志副本时，领导者将会在它的状态机上记录一个入喉。领导者加上一半的成员构成多数。这就是Raft的优势，不用等到整个集群都记录下日志再继续。


另一个例子就是，假设我们想要限制一个进程访问共享资源。这个资源是被`S`进程组所守护的。当`p`想要获取它所需的资源时，它必须先询问`S`中的大部分进程的请求。当多数`S`的进程同意`p`的请求后，`q`进入系统并且试图访问这块共享资源。无论它访问`S`中的哪个进程,`q`都不会获得多数进程的同意来访问这块资源，直到`p`释放之后它才可以访问。具体可以参考这篇论文。[The Load, Capacity, and Availability of Quorum Systems](http://www.cs.utexas.edu/~lorenzo/corsi/cs395t/04S/notes/naor98load.pdf)

Quorums 并不是总是指一组进程的多数。有时候他们甚至需要更多的进程来组成一个合法的数量来完成某个操作，就像一组可能出现Byzantine错误的N个进程。如果`f`是可以忍受的进程错误数量，一个Quorum 将是一组比`(N + f) /2 `更多的进程。详见[Introduction to Reliable and Secure Distributed Programming](http://www.amazon.com/Introduction-Reliable-Secure-Distributed-Programming/dp/3642152597/).

如果你对这个话题感兴趣，有一整本讲这个话题的书:

[Quorum Systems: With Applications to Storage and Consensus](http://www.amazon.com/Quorum-Systems-Applications-Consensus-Distributed/dp/1608456838/)

## 分布式系统中的时序问题

理解时序与其后果是分布式系统中的最大问题之一。我们习惯了生活中的一件事挨着一件事的概念,在其中有个概念非常容易定义，[happened before](https://en.wikipedia.org/wiki/Happened-before)。但是当我们有一系列分布式的进程，它们在交换消息，同步得获取资源，等等活动的时候，我们如何能确定哪个活动先发生呢？为了能够回答这样的问题，进程需要共享一个同步锁，并且准确知道电子在网络中移动需要多久，CPU调度需要多久，等等等等。很明显这不是一个现实中可能存在的系统。

在这方面开创性的论文是[Time, Clocks, and the Ordering of Events in a Distributed System](http://research.microsoft.com/en-us/um/people/lamport/pubs/time-clocks.pdf) 介绍了逻辑时钟的概念。逻辑时钟(Logical Clocks) 是一种给系统中每个活动都附加一个值的方法，这个值与真正的时间没有关系，而是与在系统中的活动的过程有关。

有许多这方面的算法,[Vector Clocks](http://zoo.cs.yale.edu/classes/cs426/2012/lab/bib/fidge88timestamps.pdf)
or
[Interval Tree Clocks](http://gsd.di.uminho.pt/members/cbm/ps/itc2008.pdf).

Justin Sheehy有个关于分布式系统中的时序非常有趣的讨论[There Is No Now](https://queue.acm.org/detail.cfm?id=2745385)

我想声明的是，时序问题在分布式系统中是最重要的问题。**我们必须忘掉同步的概念**。这关系到关于绝对主义的旧的信念，我们之前总是认为一件事是可实现的。物理学告诉我们即使是光也需要时间从一个地方到另一个地方，所以不管何时它到达我们的眼睛，被我们的大脑所处理，不管这个光有什么含义，它都是这个世界的旧的影像。这个概念在Umberto Eco 的书[Inventing the Enemy](http://www.amazon.com/Inventing-Enemy-Essays-Umberto-Eco/dp/0544104684),中有"Absolute and Relative"这章专门讨论。


## FLP 浏览

让我们来浏览下**Impossibility of Distributed Consensus with One Faulty Process**这篇论文并且试着将它与我们之前讨论的概念结合起来来结束这篇文章。

> The consensus problem involves an asynchronous system of processes, some of which may be unreliable.

所以我们有一个 _异步系统_ ，在这个系统中既没有时序的假定，也没有进程的速度啊或者传递消息所需的时间啊之类的嘉定。我们只知道有些进程可能会挂掉。

在通常的技术讨论中，[asynchronous](https://en.wikipedia.org/wiki/Asynchronous_I/O) 也许会指的是一种进程请求的方式，例如RPC那样，当一个进程 _p_ 向进程 _q_ 发出一个异步请求后，当 _q_ 在处理这个请求时，_p_ 将继续做其他事情，也就是说 _p_ 不会阻塞来等待一个答复。我们可以看到这个定义是与分布式系统中的定义完全不同的，所以没有这样的只是，很难完全理解在FLP中的第一句话。

这篇论文接下来说到：

> In this paper, we show the surprising result that no completely asynchronous consensus protocol can tolerate even a single unannounced process death. We do not consider Byzantine failures, and we assume that the message system is reliable—it delivers all messages correctly and exactly once.

所以这篇论文仅仅考虑 _Crash-stop_ 的故障模式（就像上文所说 _Fail-stop_ ）。我们可以知道没有遗漏的错误，因为消息系统是可靠的。

最后他们还加上了这样的限制：

> Finally, we do not postulate the ability to detect the death of a process, so it is impossible for one process to tell whether another has died (stopped entirely) or is just running very slowly.

所以我们也无法使用故障检测器。


概括一下， 这意味着FLP不可能将异步系统用到Fail-stop的进程上，通过可靠的消息系统，并且不知道进程的死亡。不知道不同的分布式模型的相关理论，可能会忽略不少细节，或者以与作者所想表达的意思不同的方式来解释。

更加详细的FLP相关可以从这里看到。

[A Brief Tour of FLP Impossibility](http://the-paper-trail.org/blog/a-brief-tour-of-flp-impossibility/)

[Stumbling over Consensus Research: Misunderstandings and Issues](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.174.8238&rep=rep1&type=pdf)

[halting problem](https://en.wikipedia.org/wiki/Halting_problem)).


## 总觉

正如你所看到的，学习分布式系统是很花时间的。这是一个非常大的主题，并且有无数的子领域的研究。同时，实现和验证分布式系统也是非常复杂的。有很多不可预知的情况会将我们的系统打破。

假如我们选择了错误的Quorum那么我们的新的副本算法就会失去重要的数据？或者说我们选择了一个非常保守的Quorum就会给我们的系统带来不必要的速度下降？假如我们尝试解决的问题根本不需要一致性那么我们是否可以活的好好的？也许我们的系统有着错误的时序嘉定？或者它使用了不好的错误检测器？如果我们想要优化像Raft这样的算法，是否会使得这儿或者那儿情况下系统失效？这些在我们尝试理解分布式系统理论时都会发生。

好了，我明白了，我不会重复造轮子的，但是在如此多的知识和问题需要学习的情况下，如何开始呢，接下来怎么办呢？就像刚开始说的那样，我觉得随便读论文会让你迷失，就像在FLP论文里那样，理解第一句话就需要你知道很多时序模型。因此我推荐你下面这些论文按序读。


[Distributed Algorithms](http://www.amazon.com/gp/product/1558603484)

[Introduction to Reliable and Secure Distributed Programming](http://www.amazon.com/Introduction-Reliable-Secure-Distributed-Programming/dp/3642152597/)

## References ##

- [Marcos K. Aguilera. 2010. Stumbling over consensus research: misunderstandings and issues. In Replication, Bernadette Charron-Bost, Fernando Pedone, and André Schiper (Eds.). Springer-Verlag, Berlin, Heidelberg 59-72.](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.174.8238&rep=rep1&type=pdf)
- [Paulo Sérgio Almeida, Carlos Baquero, and Victor Fonte. 2008. Interval Tree Clocks. In Proceedings of the 12th International Conference on Principles of Distributed Systems (OPODIS '08), Theodore P. Baker, Alain Bui, and Sébastien Tixeuil (Eds.). Springer-Verlag, Berlin, Heidelberg, 259-274.](http://gsd.di.uminho.pt/members/cbm/ps/itc2008.pdf)
- [Kenneth P. Birman. 2012. Guide to Reliable Distributed Systems: Building High-Assurance Applications and Cloud-Hosted Services. Springer Publishing Company, Incorporated.](http://www.amazon.com/gp/product/1447124154)
- [Mike Burrows. 2006. The Chubby lock service for loosely-coupled distributed systems. In Proceedings of the 7th symposium on Operating systems design and implementation (OSDI '06). USENIX Association, Berkeley, CA, USA, 335-350.](http://static.googleusercontent.com/media/research.google.com/en//archive/chubby-osdi06.pdf)
- [Christian Cachin, Rachid Guerraoui, and Luis Rodrigues. 2014. Introduction to Reliable and Secure Distributed Programming (2nd ed.). Springer Publishing Company, Incorporated.](http://www.amazon.com/Introduction-Reliable-Secure-Distributed-Programming/dp/3642152597/)
- [Tushar Deepak Chandra and Sam Toueg. 1996. Unreliable failure detectors for reliable distributed systems. J. ACM 43, 2 (March 1996), 225-267.](http://www.cs.utexas.edu/~lorenzo/corsi/cs380d/papers/p225-chandra.pdf)
- [Umberto Eco. 2013. Inventing the Enemy: Essays. Mariner Books.](http://www.amazon.com/Inventing-Enemy-Essays-Umberto-Eco/dp/0544104684)
- [Colin J. Fidge. 1988. Timestamps in message-passing systems that preserve the partial ordering. Proceedings of the 11th Australian Computer Science Conference 10 (1) , 56–66.](http://zoo.cs.yale.edu/classes/cs426/2012/lab/bib/fidge88timestamps.pdf)
- [Michael J. Fischer, Nancy A. Lynch, and Michael S. Paterson. 1983. Impossibility of distributed consensus with one faulty process. In Proceedings of the 2nd ACM SIGACT-SIGMOD symposium on Principles of database systems (PODS '83). ACM, New York, NY, USA, 1-7.](http://cs-www.cs.yale.edu/homes/arvind/cs425/doc/fischer.pdf)
- [Maurice P. Herlihy and Jeannette M. Wing. 1990. Linearizability: a correctness condition for concurrent objects. ACM Trans. Program. Lang. Syst. 12, 3 (July 1990), 463-492.](https://cs.brown.edu/~mph/HerlihyW90/p463-herlihy.pdf)
- [Leslie Lamport. 1978. Time, clocks, and the ordering of events in a distributed system. Commun. ACM 21, 7 (July 1978), 558-565.](http://research.microsoft.com/en-us/um/people/lamport/pubs/time-clocks.pdf)
- [Leslie Lamport. 1998. The part-time parliament. ACM Trans. Comput. Syst. 16, 2 (May 1998), 133-169.](http://research.microsoft.com/en-us/um/people/lamport/pubs/lamport-paxos.pdf)
- [Nancy A. Lynch. 1996. Distributed Algorithms. Morgan Kaufmann Publishers Inc., San Francisco, CA, USA.](http://www.amazon.com/Distributed-Algorithms-Kaufmann-Management-Systems/dp/1558603484)
- [Moni Naor and Avishai Wool. 1998. The Load, Capacity, and Availability of Quorum Systems. SIAM J. Comput. 27, 2 (April 1998), 423-447.](http://www.cs.utexas.edu/~lorenzo/corsi/cs395t/04S/notes/naor98load.pdf)
- [Brian M. Oki and Barbara H. Liskov. 1988. Viewstamped Replication: A New Primary Copy Method to Support Highly-Available Distributed Systems. In Proceedings of the seventh annual ACM Symposium on Principles of distributed computing (PODC '88). ACM, New York, NY, USA, 8-17.](http://pmg.csail.mit.edu/papers/vr.pdf)
- [Diego Ongaro and John Ousterhout. 2014. In search of an understandable consensus algorithm. In Proceedings of the 2014 USENIX conference on USENIX Annual Technical Conference (USENIX ATC'14), Garth Gibson and Nickolai Zeldovich (Eds.). USENIX Association, Berkeley, CA, USA, 305-320.](http://ramcloud.stanford.edu/raft.pdf)
- [M. Pease, R. Shostak, and L. Lamport. 1980. Reaching Agreement in the Presence of Faults. J. ACM 27, 2 (April 1980), 228-234.](http://research.microsoft.com/en-us/um/people/lamport/pubs/reaching.pdf)
- [Stefan Poledna. 1996. Fault-Tolerant Real-Time Systems: The Problem of Replica Determinism. Kluwer Academic Publishers, Norwell, MA, USA.](http://www.amazon.com/Fault-Tolerant-Real-Time-Systems-Determinism-International/dp/079239657X/)
- [Michel Raynal. 2010. Communication and Agreement Abstractions for Fault-Tolerant Asynchronous Distributed Systems (1st ed.). Morgan and Claypool Publishers.](http://www.amazon.com/Communication-Abstractions-Fault-tolerant-Asynchronous-Distributed/dp/160845293X/)
- [Michel Raynal. 2010. Fault-tolerant Agreement in Synchronous Message-passing Systems (1st ed.). Morgan and Claypool Publishers.](http://www.amazon.com/Fault-tolerant-Agreement-Synchronous-Message-passing-Distributed/dp/1608455254/)
- [Benjamin Reed and Flavio P. Junqueira. 2008. A simple totally ordered broadcast protocol. In Proceedings of the 2nd Workshop on Large-Scale Distributed Systems and Middleware (LADIS '08). ACM, New York, NY, USA, , Article 2 , 6 pages.](http://web.stanford.edu/class/cs347/reading/zab.pdf)
- [Justin Sheehy. 2015. There Is No Now. ACM Queue](https://queue.acm.org/detail.cfm?id=2745385)
- [Marko Vukolic. 2012. Quorum Systems: With Applications to Storage and Consensus. Morgan and Claypool Publishers.](http://www.amazon.com/Quorum-Systems-Applications-Consensus-Distributed/dp/1608456838/)
