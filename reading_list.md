---
layout: page
title: Reading List
permalink: /reading_list/
---

调度方面关注的会议有：

  - **系统软件**：OSDI（CCF-A），SOSP（CCF-A），ATC（CCF-A），Eurosys（CCF-B），ICDCS（CCF-B）等
  - **体系结构**：ASPLOS（CCF-A），ISCA（CCF-A），MICRO（CCF-A）等
  - **通信**：SIGCOMM（CCF-A），INFOCOMM（CCF-A），NSDI（CCF-B）等

论文收集和整理可以参考张营师兄的文章[如何收集和整理论文](https://ying-zhang.github.io/misc/2016/we-love-paper)。

#### 资源管理系统 Resource Management & Scheduling System

  - 《Large-scale cluster management at Google with
    Borg》（Eurosys'15）[pdf](https://pdos.csail.mit.edu/6.824/papers/borg.pdf),
    [译文](https://ying-zhang.github.io/yi/2017/x-eurosys15-borg-cn/)，[Youtube](https://www.youtube.com/watch?v=0W49z8hVn0k)
  - 《Mesos: A Platform for Fine-Grained Resource Sharing in the Data
    Center》（NSDI'11）[usenix](http://static.usenix.org/events/nsdi11/tech/full_papers/Hindman_new.pdf)，[mesos](http://mesos.apache.org/)
  - 《Apache Hadoop YARN: yet another resource
    negotiator》（SOCC'13）[acm](https://dl.acm.org/citation.cfm?id=2523633)，[YARN](https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/YARN.html)
  - 《Apollo: scalable and coordinated scheduling for cloud-scale
    computing》（OSDI'14）[usenix](https://www.usenix.org/system/files/conference/osdi14/osdi14-paper-boutin_0.pdf)
  - 《Omega: flexible, scalable schedulers for large compute
    clusters》（Eurosys'13）[acm](https://dl.acm.org/citation.cfm?id=2465386)
  - 《Firmament: Fast, Centralized Cluster Scheduling at
    Scale》（OSDI'16）[usenix](https://www.usenix.org/conference/osdi16/technical-sessions/presentation/gog)，[firmament](https://github.com/camsas/firmament)
  - 《Sparrow: distributed, low latency
    scheduling》（SOSP'13）[acm](https://dl.acm.org/citation.cfm?id=2522716)
  - 《Mercury: Hybrid centralized and distributed scheduling in large
    shared
    clusters》（ATC'15）[usenix](https://www.usenix.org/system/files/conference/atc15/atc15-paper-karanasos.pdf)
  - 《Fuxi: a Fault-Tolerant Resource Management and Job Scheduling
    System at Internet
    Scale》（VLDB'14）[pdf](http://www.vldb.org/pvldb/vol7/p1393-zhang.pdf)
  - 《Advancements in YARN Resource
    Manager》（2018）[pdf](https://www.microsoft.com/en-us/research/uploads/prod/2018/02/yarn-big-data-encyclopedia-2018.pdf)，[译文](https://ying-zhang.github.io/yi/2018/x-adv-yarn/)
  - 《Multi-resource Packing for Cluster
    Schedulers（Tetris）》（SigComm'14）[pdf](https://www.cs.cmu.edu/~xia/resources/Documents/grandl_sigcomm14.pdf)，[YARN-2745](https://issues.apache.org/jira/browse/YARN-2745)，[pptx](https://issues.apache.org/jira/secure/attachment/12677137/sigcomm_14_tetris_talk.pptx)

#### 改进资源效率和调度策略 Improving Resource Efficiency & Scheduling Policy

  - 《Altruistic Scheduling in Multi-Resource
    Clusters》（OSDI'16）[usenix](https://www.usenix.org/conference/osdi16/technical-sessions/presentation/grandl_altruistic)
  - 《Pado: A data processing engine for harnessing transient resources
    in
    datacenters》（Eurosys'17）[acm](https://dl.acm.org/citation.cfm?id=3064181)
  - 《GRAPHENE:Packing and Dependency-aware Scheduling for Data-Parallel
    Clusters》（OSDI'16）[usenix](https://www.usenix.org/sites/default/files/osdi16_full_proceedings_interior.pdf#page=89)
  - 《3sigma: distribution-based cluster scheduling for runtime
    uncertainty》（Eurosys'18）[acm](https://dl.acm.org/citation.cfm?id=3190515)
  - 《Efficient Queue Management for Cluster
    Scheduling》（Eurosys'16）[acm](https://dl.acm.org/citation.cfm?id=2901354)
  - 《Dominant Resource Fairness: Fair Allocation of Multiple Resource
    Types》（NSDI'11）[usenix](http://static.usenix.org/events/nsdi11/tech/full_papers/Ghodsi.pdf)
  - 《Monotasks: Architecting for Performance Clarity in Data Analytics
    Frameworks》（SOSP'17）[acm](https://dl.acm.org/citation.cfm?id=3132766)
  - 《ROSE: Cluster Resource Scheduling via Speculative
    Over-subscription》（ICDCS'18）[eprints](http://eprints.lancs.ac.uk/125028/)

#### 异构作业混合部署 Colocating Heterogeneous Workloads

  - 《Heracles: improving resource efficiency at
    scale》（ISCA'15）[acm](http://dl.acm.org/citation.cfm?doid=2749469.2749475)
  - 《Don't cry over spilled records: Memory elasticity of data-parallel
    applications and its application to cluster
    scheduling》（ATC'17）[usenix](https://www.usenix.org/conference/atc17/technical-sessions/presentation/iorgulescu)[DSS](https://github.com/epfl-labos/DSS)
  - 《Bolt: I Know Waht You Did Last Summer... In the
    Cloud》（ASPLOS'17）[acm](http://dl.acm.org/citation.cfm?id=3037703)
  - 《Quasar: resource-efficient and QoS-aware cluster
    management》（ASPLOS'14）[acm](https://dl.acm.org/citation.cfm?id=2541941)
  - 《Interference management for distributed parallel applications in
    consolidated
    clusters》（ASPLOS'16）[acm](https://dl.acm.org/citation.cfm?id=2872388)
  - 《Dirigent: Enforcing QoS for latency-critical tasks on shared
    multicore
    systems》（ASPLOS'16）[acm](https://dl.acm.org/citation.cfm?id=2872394)
  - 《CPI2: CPU performance isolation for shared compute
    clusters》（EuroSys'13）[Google
    Pub](https://ai.google/research/pubs/pub40737)

#### 大规模机器学习作业的调度 Scheduling for Deep Learning Clusters

  - 《Optimus: an efficient dynamic resource scheduler for deep learning
    clusters》（Eurosys'18）[acm](https://dl.acm.org/citation.cfm?id=3190517)[Optimus
    Src](https://github.com/eurosys18-Optimus/Optimus)
  - 《Scaling Distributed Machine Learning with the Parameter
    Server》（OSDI'14）[usenix](https://www.usenix.org/system/files/conference/osdi14/osdi14-paper-li_mu.pdf)
  - 《Litz: An Elastic Framework for High-Performance Distributed Machine
    Learning》（CMU-PDL-17-103，ATC'18）[CMU PDL TR
    List](http://www.pdl.cmu.edu/Publications/pubs-tr.shtml)
  - 《Project Adam: Building an Efficient and Scalable Deep Learning
    Training
    System》（OSDI'14）[usenix](https://www.usenix.org/system/files/conference/osdi14/osdi14-paper-chilimbi.pdf)
  - 《Gandiva: Introspective Cluster Scheduling for Deep
    Learning》（OSDI'18）[usenix](https://homes.cs.washington.edu/~patelp1/papers/gandiva-osdi18.pdf)
  - 《Topology-aware GPU scheduling for learning workloads in cloud
    environments》（SC'17）[acm](https://dl.acm.org/citation.cfm?id=3126933)
  - [ A
    List](https://github.com/gaocegege/papers-notebook/labels/area%2Flarge-scale-ml%20)

#### 杂项 Miscellaneous

  - 《The Tail at Scale》（CACM'13）[Google
    Pub](https://ai.google/research/pubs/pub40801)
  - 《Cluster Scheduling for
    Datacenters》（CACM'18）[ACM](https://cacm.acm.org/magazines/2018/5/227184-research-for-practice/fulltext)
  - 《Resource Management with Deep Reinforcement
    Learning》（HotNets'16）[pdf](https://people.csail.mit.edu/alizadeh/papers/deeprm-hotnets16.pdf)
  - 《Reconciling high server utilization and sub-millisecond
    quality-of-service》（EuroSys'14）[pdf](http://csl.stanford.edu/~christos/publications/2014.mutilate.eurosys.pdf)
  - 《Heterogeneity and dynamicity of clouds at scale - Google trace
    analysis》（SoCC'12）[pdf](http://www.pdl.cmu.edu/PDL-FTP/CloudComputing/googletrace-socc2012.pdf)
  - 《Job Scheduling without Prior Information in Big Data Processing
    Systems》（ICDCS'17）
  - 《The evolution of cluster scheduler
    architectures》[blog](http://firmament.io/blog/scheduler-architectures.html)，[
    译文](http://www.infoq.com/cn/articles/scheduler-architectures%20)
  - 《阿里巴巴云化架构创新之路》（ArchSummit'17）[Infoq
    CN](http://www.infoq.com/cn/presentations/the-way-of-innovation-of-alibaba-cloud-architecture)
  - 《阿里巴巴调度与集群管理系统Sigma》（ArchSummit'17）[Infoq
    CN](http://www.infoq.com/cn/presentations/alibaba-scheduling-and-cluster-management-system-sigma)

#### 研究组 Research Groups

  - Cluster Resource Management - Microsoft Research [
    MSR](https://www.microsoft.com/en-us/research/project/cluster-resource-management/%20)
  - Cloud Efficiency - Microsoft Research [
    MSR](https://www.microsoft.com/en-us/research/group/cloud-efficiency/%20)
  - Distributed Systems and Parallel Computing from Google Pubs [ Google
    Pubs](https://ai.google/research/pubs?area=DistributedSystemsandParallelComputing%20)
  - LABOS from EFPL [ EFPL](https://labos.epfl.ch/page-43468-en.html%20)
  - Parallel Data Lab Project - Cloud Scheduling (TetriSched) from CMU [
    CMU](http://www.pdl.cmu.edu/TetriSched/index.shtml%20)
  - Christos Kozyrakis [
    Stanford](http://csl.stanford.edu/~christos/%20), Christina
    Delimitrou [ Cornell](http://www.csl.cornell.edu/~delimitrou/%20)
  - Malte Schwarzkopf [ MIT](https://people.csail.mit.edu/malte/%20)
  - Shivaram Venkataraman [ UC Berkeley](http://shivaram.org/#pubs%20),
    Kay Ousterhout[ UC
    Berkeley](https://people.eecs.berkeley.edu/~keo/%20)
  - Robert Grandl [ Wisc](http://pages.cs.wisc.edu/~rgrandl/%20)
