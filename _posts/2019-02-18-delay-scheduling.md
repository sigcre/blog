---
layout:     post
title:      "Spark 中的 Delay Scheduling「延迟调度」"
subtitle:   "A Simple Technique for Achieving Locality and Fairness in Cluster Scheduling"
date:       2019-02-28 18:35:33
author:     Jie CHEN
reflink:    http://jetmuffin.com/2019/03/01/delay-scheduling/
catalog:    true
multilingual:   false
mathjax:    true
tags:
    - spark
    - data locality
    - fairness
    - 资源调度
---

《Dealy Scheduling: A Simple Technique for Achieving Locality and Fairness in Cluster Scheduling》这篇论文是 Spark 作者 Matel Zaharia 发表在 Eurosys'10 上的一篇比较有代表性的文章，文章的思路很简单，就是以等待的方式，来获取更好的数据本地性的机会。这个理念也被加入到 Spark 中。

## Background

传统的多任务调度中，fairness 常常是比较重要的指标，要保证多个 job 公平地获取到资源的 slot，而不会出现饥饿的情况。最朴素的公平调度算法像 max-min 算法，就是应用简单的贪心：

```
Algorithm 1:
if node n has a free slot then
  sort jobs in increasing order of number of running tasks
  for j in jobs do
    if j has unlaunched task t then
      launch t on n
    endif
  endfor
endif
```

而另外一个角度，数据本地性则是影响 data-analytic 这类大数据任务的处理速度关键因素之一。以 Spark 为例，data locality 有 **PROCESS_LOCAL，NODE_LOCAL，RACK_LOCAL，ANY** 这几种（NO_PREF 比较特殊，暂时不考虑）。低级别的数据本地性往往带有数据传输通信开销，上述几种数据本地性分别需要进行**进程间，节点间，机柜间**的通信，传输时间及总数据处理时间依次提升。为了保证处理速度更快，data-analytic 任务一般需要尽量高的数据本地性。

这就造成了 fairness 和 data locality 之间的 tradeoff，往往数据本地性会因为需要满足 fairness 而得不到满足。

## Delay Scheduling

针对这个 tradeoff，很简单的一个 optimization 就是 **delay**。当为了满足 fairness，即有个 slot 空闲时，当前队首的任务可用选择不使用这个 slot 执行，而等待一段时间，等下一个 data locality 等级更高的 slot 的出现。

### Scheduling Responsiveness

首先需要理解的第一个概念是 **Scheduling Responsiveness**，即调度的「响应时间」。在经典的多任务公平调度场景下，一个任务如果理论将被分配到的资源份额为 `F` slots，那么从它提交给调度器，调度器分配给它资源份额，再它真正得到这 `F` 个 slots 并进行运行的总时长为调度的响应时间。而响应时间事实上取决于集群资源释放的频率（如果是非常富裕的集群的话，调度就相当于 FIFO，响应时间为零，这里不考虑这种情况）。我们假设任务平均时长是 `T`，并且集群总共有 `S` slots，那么等待一个 slot 就需要耗时 `T/S`，一整个 job 需要等待 `FT/S` 时间。另外一个 job 的所有 task 都开始需要资源进行执行的时长为 `J`。

那么在下面几种场景下，**等待时间**对调度的**响应时间**不影响。

- 集群有很多的 job（`f=F/s` 非常小）
- 集群的 job 很小，需要的 slots 很少（`F` 很小）
- job 很长（task 一个一个到来很慢，`J` 很大）

换言之，以上几种情况都是达到了 `J >> FT/S` 的情况。

作者还分析了 Facebook，Yahoo! 的任务 trace，发现大多数的 task 运行时长都很短，且集群 slot 释放速度很快，可以满足大多数的 job（虽然大多数时间下集群都是 full 状态）。

### Locality Problem

接着我们来回顾传统的公平调度为什么会出现 Locality 的问题。

首先是 **Head-of-line** 的问题。从 trace 中发现，大多数的 task 读取的数据都很小，因此每个 task 读取的 data blocks 很少（例如在 HDFS 中 1 block 为 64MB，那么小 task 在单副本情况下只有 1 个 block 可读）。那么我们使用 Algorithm 1 的贪心算法时会发现，如果仅仅用 `running tasks` 来进行排序，那么排在最前面的 job（也就是 `head-of-line`），就必须把它的第一个找到的未执行的 task 跑在这个 slot 上（data locality 随缘）。如果这个 head-of-line job 很小（极端的讲，只有一两个 task），那么它得到数据本地性的概率就很小了。

其次，是 **sticky slots** 问题，同一个 job 经常会跑在同一个 slot 上。这主要是满足 fairness 时的 max-min 造成的，例如当前调度器中所有任务的 task 数量相同，有一个 slot `n` 空闲，那么 head-of-line job `j` 被允许跑一个 task 在 `n` 上。假如这时这个 task 结束，slot `n` 再次空闲，那么根据 Algorithm 1 当前 running task 最小的还是 `j`，`j` 的任务又跑在 slot `n` 上了。sticky slots 造成一个 job 难以摆脱某个 slot 的数据本地性差的境况。

### Analysis

简单修改 Algorithm 1，我们就可以引入 delay 的机制：

```
Algorithm 2:
if node n has a free slot then
  sort jobs in increasing order of number of running tasks
  for j in jobs do
    if j has unlaunched task t with data on n then
      launch t on n
      set j.skipcount = 0
    else if j has unlauched task t then
      if j.skipcount >= D then
        launch t on n
      else
        j.skipcount ++
      endif
    endif
  endfor
endif
```

当一个 slot 空闲时，首先判断这个 slot 是不是满足 head-of-line job 的 task 的数据本地性，如果不满足，那么我们跳过这个 slot，直到到达阈值才接受非本地数据的运行。那么问题就转化成了：

- data locality 和跳过的次数 `D` 有定量的什么关系？
- 一个 job 需要跳过多少次 `D`，或者需要等待多少时间 `t` 来满足一定的 data locality

假设一个 $M$ 节点的集群，每个节点有 $L$ 个 slot，共有 $S=ML$ 个 slot。调度器收到了一个 job j 包含 $N$ 个任务，$P_j$ 是包含 j 数据的节点数，j 的所有 task 运行时长为 $T$，每个 task 的数据有 $R$ 个副本。

我们的 delay 机制让 j 等待 $D$ 次才允许跑在非本地的 slot 上。那么一个 task 跳过 $D$ 次都没找到满足条件 slot 的概率为 $(1-p_j)^D$。而跳过一次，一整个 job j（当前 task 数位 $K$）没找到任何一个满足本地性 task 的概率是 $p_j=1-(1-\frac{K}{M})^R$，因此，经过 $D$ 次跳跃，job j 满足数据本地性的概率是 $1-(1-p_j)^D=1-(1-\frac{K}{M})^{RD}$

简单缩放可以得到 $1-(1-\frac{K}{M})^{RD} \geq 1-e^{-RDK/M}$ （根据 $e^x \geq 1+x$ 推出），那么整个 job j 的满足本地性概率则是做个求和（$K$ 随着任务执行慢慢从 $N$ 减少到 $1$），即为：

$$
\begin{aligned}
l(D) \geq & \frac{1}{N}\sum_{K=1}^{N}{1-e^{-RDK/M}} \\
     \geq & 1 - \frac{1}{N}\sum_{K=1}^{N}{e^{-RDK/M}} \\
     \geq & 1 - \frac{e^{-RD/M}}{N(1-e^{-RD/M})}
\end{aligned}
$$

令满足数据本地性的概率 $\lambda=l(D)$，那么：

$$
D \geq - \frac{M}{R}\ln{(\frac{(1-\lambda)N}{1+(1-\lambda)N})}
$$

回顾之前我们提到一个 slot 释放的时间是 $\frac{T}{S}$，等待 $D$ 次就耗时 $\frac{T}{S}D$。简单举个实际的例子，我们要达到 $\lambda=0.95$的数据本地性，$N=20$，$R=3$，那么就需要 $D\geq 0.23M$，而每次 job 需要等待的时间是 $\frac{D}{S}T=\frac{D}{LM}T=\frac{0.23}{L}T$。如果每个节点有 $L=8$ 个 slot，那么久需要等待任务时长 $T$ 的 $2.8%$ 的时间。

## Implementation

### Hadoop

论文在 Hadoop 上实现了 delay scheduling（其实那个时候没有 spark），算法也很简单，在 Algorithm 2 的基础上，把等待次数改成等待时间。

```
Algorithm 3:
if node n has a free slot then
  sort jobs in increasing order of number of running tasks
  for j in jobs do
    if j has node-local task t on n then
      set j.wait = 0 and j.level = 0
      launch t on n
    else if j has rack-local task t on n and (j.level >= 1 or j.wait >= W1) then
      set j.wait = 0 and j.level = 1
      launch t on n
    else if j.level = 2 or (j.level = 1 and j.wait >= W2) or (j.level = 0 and j.wait >= W1 + W2) then
      set j.wait = 0 and j.level = 2
    else
      set j.skipped = true
    endif
  endfor
endif
```

论文的实验也是在 Hadoop 上进行，这里不再具体讨论了。

### Spark

在 Spark 的调度中用到了论文中的 delay scheduling，和在 Hadoop 实现一样，Spark 需要指定每个 level 的等待阈值（即 `W1`, `W2`），对于每个提交的 spark job，需要指定 `spark.locality.wait.process`，`spark.locality.wait.node`，`spark.locality.wait.rack`。

具体的实现是在类 `TaskSetManager` 中，它首先将所有的 task 放到不同 `NODE_LOCAL`，`PROCESS_LOCAL`，`RACK_LOCAL`，`ANY` 四个 map 里，当有可用资源 slot 时，resourceOffer 方法里首先根据 `job.wait` 判断当前可用的最大 locality，再根据 locality 过滤出 task，若有 task 可用执行，则返回 task 中的最优的一个。


```scala
// Implementation in Spark

TaskSetManager::resourceOffer(execId: String, host: String,maxLocality: TaskLocality.TaskLocality): Option[TaskDescription]
    if (maxLocality != TaskLocality.NO_PREF) // 如果资源不是 NO_PREF 的，有 locality 的
        ...
        allowedLocality = getAllowedLocalityLevel(curTime) // 获取当前 taskSet 允许执行的 locality。getAllowedLocalityLevel 随时间而变化
        if (allowedLocality > maxLocality)  // 如果资源的 locality 级别高于 taskSet 允许的级别
            allowedLocality = maxLocality // 那么提升 taskSet 的级别
        task =  findTask(execId, host, allowedLocality) // 根据允许的 locality 级别去找一个满足要求的 task
                                                        // 从最优的 locality 级别(process_local)开始找，返回一个满足 locality 的 task（为最优级别）
        ...
    task match case Some((index, taskLocality, speculative)) // 找到了一个可行的 task
        val info = new TaskInfo(taskId, index, attemptNum, curTime, execId, host, taskLocality, speculative)
        if (maxLocality != TaskLocality.NO_PREF) // NO_PREF will not affect the variables related to delay scheduling
            currentLocalityIndex = getLocalityIndex(taskLocality) // Update our locality level fJie CHEN
            reflink:    http://jetmuffin.com/2019/03/01/delay-scheduling/or delay scheduling
            lastLaunchTime = curTime // 更新最近执行 task 的时间，计算当前 locality 时需要
        ...
        addRunningTask(taskId) // 加入执行 task 中
        logInfo("Starting %s (TID %d, %s, %s, %d bytes)"
        sched.dagScheduler.taskStarted(task, info) // 通知调度器有 task 开始运行
            eventProcessActor ! BeginEvent(task, taskInfo)
        return Some(new TaskDescription(taskId, execId, taskName, index, serializedTask)) // 返回匹配的 task
    case _ => return None // 没有满足 locality 要求的 task，返回 None
```

然而几个参数的具体数值，还是需要任务的提交者自己去给定。通常一个 data-analytic 的任务会被提交多次，因此用户对一个 job 的时长有所判断，可用根据经验给出。若对于未知的任务，一般给一个比较保守的值。若要在无先验知识的情况下去给定这几个阈值，那么可能就涉及到 **online runtime estimation** 之类的概念了。

## 参考文献

- Zaharia M, Borthakur D, Sen Sarma J, et al. **Delay scheduling: a simple technique for achieving locality and fairness in cluster scheduling**[C]//Proceedings of the 5th European conference on Computer systems. ACM, 2010: 265-278.
- Max-Min Fairness (Wikipedia).[http://en.wikipedia.org/wiki/Max-min_fairness](http://en.wikipedia.org/wiki/Max-min_fairness).
- Carl A W, William E W. **Lottery Scheduling a flexible proportional-share resource management**[C]//Proceedings of the 1st USENIX Symposium on Operating Systems Design and Implementation (OSDI). 1994.
- Isard M, Prabhakaran V, Currey J, et al. **Quincy: fair scheduling for distributed computing clusters**[C]//Proceedings of the ACM SIGOPS 22nd symposium on Operating systems principles. ACM, 2009: 261-276.Jie CHEN
reflink:    http://jetmuffin.com/2019/03/01/delay-scheduling/