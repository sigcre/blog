---
layout: post
title: "分布式深度学习资源调度"
description: "There is no one who loves pain itself, who seeks after it and wants to have it, simply because it is pain..."
comments: true
keywords: "deep learning, scheduling"
author: Qingping LI
reflink: https://konnase.github.io/2018/09/14/deep_learning/deep_learning_scheduling
---
## 分布式机器学习相关会议
- 各大机器学习会议上（NIPS， ICML，ICLR）
- 各大系统会议上（SOSP，OSDI，ATC，EuroSys，SoCC）
- 应用对应的顶级会议上（CVPR，KDD）

## 常用的深度学习数据集
- ImageNet：15 million 带标签的高分辨率224*224的RGB图像，共22,000catagories，1.2TB
- ImageNet 2012: 1.2 million 带标签的高分辨率224*224的RGB图像，共1,000个分类，138GB；50,000张测试图像，1,000个分类
- MNIST：输入是28*28是的二值图，输出是0-9这是个数字，60,000 训练图像，10,000测试图像
- Cifar-10：由60,000张32*32的RGB彩色图片构成，共10个分类。50,000张训练，10,000张测试（交叉验证）

## 现有的一些分布式深度学习平台
- 腾讯的Mariana
- Google的DistBelif
- 微软的Adam

<!--more-->

## 神经网络相关内容

### 神经网络
- 多层神经网络需要非线性映射。如果全连接层没有非线性部分，只有线性部分，那么计算之后，线性的多层神经网络其实可以转换成一层的神经网络。故加入非线性层，多层神经网络才有意义。
- 为防止使用sigmoid激活函数可能导致的梯度消失问题，激活函数一般取relu函数

### 提高验证准确性的方法
- 增加数据集：Adding more data augmentations often reduces the gap between training and validation accuracy. Data augmentation could be reduced in epochs closer to the end.
- 初始学习率设置大一点：Start with a large learning rate and keep it large for a long time. For example, in CIFAR10, you could keep the learning rate at 0.1 for the first 200 epochs and then reduce it to 0.01.
- batch size不能太大：Do not use a batch size that is too large, especially batch size >> number of classes.
  - batch\_size增大，会使训练速度加快，不过太大会导致learning rate不好调，因为lr和batch size之间不是线性关系
  - batch\_size太小，模型训练的慢

### Data Augmentation
人工增加训练集的大小，包括反射(image reflection)、平移(translation)、旋转(rotation)、加噪声等方法从已有的数据中创造一批新的数据

### Batch Normalization
CNN网络在训练的过程中，前一层的参数变化影响着后面层的变化（因为前面层的输出是后面的输入），而且这种影响会随着网络深度的增加而不断放大。在CNN训练时，绝大多数都采用mini-batch使用随机梯度下降算法进行训练，那么随着输入数据的不断变化，以及网络中参数不断调整，网络的各层输入数据的分布则会不断变化，那么各层在训练的过程中就需要不断的改变以适应这种新的数据分布，从而造成网络训练困难，难以拟合的问题。 

BatchNorm的目的是将每一层的输入数据进行归一化(normalization)，使得每一层的数据分布是稳定的--均值0方差1，增加两个可学习的参数 β 和 γ ，对数据进行缩放和平移，从而达到加速训练的目的

### momentum
用来加速训练过程的。引入momentum后，采用如下公式：
```
    v = mu * v - learning_rate * dw
    w = w + v
```
v初始为0，mu是一个超参数，一般设置为0.9。这样理解：如果上一次v和这一次的负梯度方向是相同的，则w下降的幅度会加大，从而使收敛的速度加快；反之，w下降的幅度会减小，收敛的速度减慢。

### Softmax函数
即归一化指数函数，Softmax函数将向量等比例压缩到[0,1]之间，且保证所有元素之和为1。在多分类问题中用作输出层。
$$ softmax(i) = \frac{e^i}{\sum_{j}e^j} $$

向量里面的每个元素都可能取到，只是取到的概率由softmax函数求出的值给出。

在做反向传播的时候，也很方便，只需将softmax算出来的类别向量对应的真正结果的那一维减一就可以了。比如通过若干层的计算，最后得到的某个训练样本的向量的分数是[ 1, 5, 3 ],那softmax计算出来的概率就是$[\frac{e^1}{e^1 + e^3 + e^5},\frac{e^5}{e^1 + e^3 + e^5},\frac{e^3}{e^1 + e^3 + e^5}]=[ 0.015, 0.866, 0.117 ]$，如果这个样本正确的分类是第二个的话，那么计算出来的偏导就是$[ 0.015, 0.866 - 1, 0.117 ] = [ 0.015, -0.134, 0.117 ]$，然后根据这个偏导做反向传播，更新参数值。

### resnet模型
![](/assets/images/resnet_params.png)

## 分布式机器学习的研究领域

### 从统计、优化理论、优化算法角度来做
  
关注以下问题：通过分布式并行或其他方法加速训练后，这个新算法还能保证收敛到之前相同的最优值（或者一个满意的最优值）吗？在分布式环境下，收敛能有多快，比非分布式训练快多少？收敛的有多接近，和单机上跑出来的最优解一样吗？收敛需要什么假设？*应该怎么设计训练过程*（比如，怎么抽样数据、怎么更新参数）从而保证能接近某个最优解，同时还保证加速？

### 从机器学习的模型角度
  
修改原有的模型；提出新的模型；

### 优化方向
- 异步训练：参数服务器架构；减小通信开销（计算开销<<通信开销）
- 数据并行：将数据分成很多小块，加快训练模型；要求数据之间没有依赖性
- 模型并行：如何划分模型？模型之间的关联性要考虑

  Petuum和Tensorflow既支持数据并行也支持模型并行
  - 将CNN中的不同层放到不同的device(CPU or GPU)上
    
    这样拆分使得参数被分配到多个device上，对参数量巨大的模型就可以提供很好的支持
  - 将CNN中某一层拆开放到不同的device上
    
    实现难度较大

- 对反向传播算法的重调度：使计算时间和通信时间尽可能重叠
- 模型更新：对于全连接层，反向传播的时候，下一层计算的$\Delta w$

### 并行收敛算法满足的假设
- 训练程序的计算任务集中在参数更新函数上
- 每个iteration，数据之间没有依赖性
- 每个iteration开始之前，每个计算节点需要比较容易的拿到模型参数；每个节点计算完成之后，需要比较容易的将梯度收集起来应用于参数更新:
$ \theta(t+1) = \theta(t) - \alpha * \Delta(\theta(t),D_p(t)) $

前两个条件容易满足，但第三个条件在多机环境下涉及网络通讯，如何保证参数共享以及梯度同步成了保证第三个条件要解决的核心问题。

### All reduce并行算法
设数据量为 X，N GPUs，如果使用一个main gpu来做参数更新并把更新之后的参数再发送会其他worker，则此过程交换的参数为`2*X*(N-1)`

这是 Baidu Silicon Valley AI Lab在GTC 2017上提出了allreduce的实现算法，一些分析如下：

#### Multi-Tower Scaling
[Effectively scaling deep learning framework](http://on-demand.gputechconf.com/gtc/2017/presentation/s7543-andrew-gibiansky-effectively-scakukbg-deep-learning-frameworks.pdf)提到分布式训练中网络开销巨大，计算节点之间传递的参数数目太多，如果网络带宽不大，传输开销很大，影响扩展性。

下图指出使用一个main gpu来做参数同步的话，在他们的benchmark中，使用Titan X Maxwell GPU计算单个gpu发来的参数耗时在300ms左右，如果N的数目巨大，这将成为巨大的瓶颈。同时，通信开销在(N-1)/3 s，表明通信开销也是随N的数目线性增长。
![communication overhead](/assets/images/communication_overhead.png)
下图指出随着N的数目的增大，花在通信上的开销的比列越来越大
![communication overhead](/assets/images/communication_overhead_1.png)
遂提出了ring all reduce算法来解决高communication overhead的问题

##### 采用allreduce方式后

  ![allreduce](/assets/images/pscl_fig_2_allreduce.png)
  ![allreduce](/assets/images/pscl_fig_4_allreduce.png)

  Process 1-4上面分别由4个chunk
  - scatter reduce: 执行完N-1步之后，每个Process上都会有一个完整的chunk（比如Process 4上包含了c\_11, c\_21, c\_31和c\_41，即所有进程的chunk 1）
  - all gather: 然后将完整的chunk overwrite到其他不含有完整chunk的Process上

  每个GPU会迭代2(N-1)次，并在每次迭代中发送 X/N 大小的数据。则完成一次reduce过程交换的参数量为`2*X*(N-1)/N`

##### Scaling with tensorflow
  下图指出，Baidu在tensorflow的训练流图中的back prop 和weight update之间增加了ring allreduce，使用MPI实现GPUs之间的参数传递

  ![scaling with tensorflow](/assets/images/scaling_with_tensorflow.png)

  结果是使用40个GPUs，训练速度几乎呈线性增长

## 分布式深度学习调度
### 多任务调度
深度学习作业都是运行时间较长的作业，所以一般一个timeslot都会比较大，比如可能40 min或者1 hour

## 参考文献
- [张昊的知乎专栏](https://zhuanlan.zhihu.com/p/30976469)
- [深度学习--Batch Normalization 算法介绍](https://blog.csdn.net/lhanchao/article/details/70308092)
- [Softmax的理解和应用](https://blog.csdn.net/superCally/article/details/54234115)
- [technologies behind distributed deep learning allreduce](https://preferredresearch.jp/2018/07/10/technologies-behind-distributed-deep-learning-allreduce/)