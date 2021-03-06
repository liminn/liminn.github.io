﻿---
title: ID3/C4.5算法
mathjax: true
date: 2017-12-6 21:30:50
categories: 
- 机器学习
tags:
- 决策树
- ID3
- C4.5
- 熵
- 条件熵
- 信息增益
- 信息增益比
- 预剪枝
- 后剪枝
- 连续值
- 缺失值
---
　　决策树(decision tree)是机器学习中一种基本的分类与回归方法。
　　决策树模型的3种经典学习算法：ID3，C4.5，CART。ID3与C4.5主要适用于分类问题；CART既适用于分类也适用于回归预测问题。
　　本文主要记录ID3与C4.5算法，两者的区别主要在于前者用的是信息增益来选择特征，后者用的是信息增益比来选择特征以及增加对连续值特征的处理和缺失值特征的处理。

<!-- more --> 

## 一、简介
（1）决策树定义：决策树由结点(node)和有向边(directed edge)组成。结点有两种类型：内部结点(internal node)和叶结点(leaf node)组成。内部节点表示一个特征或属性，叶结点表示一个类或者回归预测值。
（2）决策树决策过程：从根结点开始，对样本的某一特征进行测试，根据测试结果，将样本分配到其子结点；这时，每一个子结点对应着该特征值的一个取值。如此递归对样本进行测试并分配，直至达到叶结点；
（3）决策树模型的学习包含3个步骤：特征选择，决策树的生成（局部最优化过程），决策树的剪枝（全局最优化过程）。
## 二、特征选择
### 2.1 熵的定义
　　熵表示随机变量不确定性的度量。设$X$是一个取有限值的离散随机变量，其概率分布为：`$P(X=x_i)=p_i,\ i=1,2,...,n$`，则随机变量`$X$`的熵定义为：$$H(X) = -\sum_{i=1}^{n}p_ilog_2p_i$$
　　由定义可知，熵只依赖于$X$的分布，而与$X$的值无关，所以熵也可以记为$H(p)$。
熵越大，随机变量的不确定性越大。当随机变量只取两个值时，例如1，0时，熵$H(p)$随概率$p$变化的曲线如图（分布为伯努利分布时熵与概率的关系）：
<img src="http://ozruihqgo.bkt.clouddn.com/%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0/ID3%E4%B8%8EC4.5/Binary_entropy_plot.svg.png" width="30%" height="30%">　　当$p=0$或$p=1$时$H(p)=0$，随机变量完全没有不确定性。当$p=0.5$时，$H(p)=1$，熵取值最大，随机变量不确定性最大。
### 2.2 条件熵的定义
　　设有随机变量$(X,Y)$，其联合概率分布为
$$P(X=x_i,Y=y_j)=p_{ij},i=1,2,...,n;j=1,2,...,m$$
　　条件熵$H(Y|X)$表示在已知随机变量$X$的条件下随机变量$Y$的不确定性。随机变量$X$给定的条件下随机变量$Y$的条件熵$H(Y|X)$，定义为$X$给定条件下$Y$的条件概率分布的熵对$X$的数学期望
$$H(Y|X)=\sum_{i=1}^{n}p_iH(Y|X=x_i)$$
　　这里，$p_i=P(X=x_i),i=1,2,...,n.$
　　当熵和条件熵中的概率由数据估计（特别是极大似然估计）得到时，所对应的**熵**与**条件熵**分别成为**经验熵**和**经验条件熵**。
### 2.3 信息增益的定义
（1）符号定义
- $D$：训练数据集；
- $C_k$：第$k$个类的数据集； 
- $D_i$：根据特征$A$的第$i$个取值划分出来的数据集;
- $D_{ik}$：根据特征$A$的第$i$个取值划分出来的数据集与第$k$个类的数据集的交集；
- 数据集总共有$K$个类，特征$A$有$n$个取值，符号$|\cdot |$表示数据集容量。

（2）数据集$D$的经验熵$H(D)$：
$$\begin{eqnarray*}
H(D)&=&-\sum_{k=1}^{K}p_klog_2p_k\\
    &=&-\sum_{k=1}^{K}\frac{|C_k|}{|D|}log_2\frac{|C_k|}{|D|}
\end{eqnarray*}
$$
（3）特征$A$给定条件下$D$的经验条件熵$H(D|A)$(尝试用特征$A$划分会有$n$个分支，相当求于求特征$A$的Expected Entropy，也可写做$EH(A)$)：
$$\begin{eqnarray*}
H(D|A)&=&\sum_{i=1}^n\frac{|D_i|}{|D|}H(D_i)\\
&=&-\sum_{i=1}^n\frac{|D_i|}{|D|}\sum_{k=1}^{K}\frac{|D_{ik}|}{|D_i|}log_2\frac{|D_{ik}|}{|D_i|}
\end{eqnarray*}
$$
（4）计算特征$A$对样本集$D$的信息增益$g(D,A)$：
$$g(D,A)=H(D)-H(D|A)$$
　　给定训练数据集$D$和特征$A$，经验熵$H(D)$表示对数据集$D$进行分类的不确定性。而经验条件熵$H(D|A)$表示在特征$A$给定的条件下对数据集$D$进行分类的不确定性。那么它们的差，即信息增益$g(D,A)$，就**表示由于特征$A$而使得对数据集$D$的分类的不确定性减少的程度**。
　　显然，对于数据集$D$而言，信息增益依赖于特征，不同的特征往往具有不同的信息增益。**信息增益大的特征具有更强的分类能力**。
　　**ID3采用信息增益来选择最优划分属性**。信息增益准则对可取值数目较多的特征有所偏好，为减少这种偏好可能带来的不利影响，**C4.5决策树算法不直接采用信息增益，而是使用“信息增益比”来选择最优划分属性**。
### 2.4 信息增益比
　　特征$A$对训练数据集$D$的信息增益比`$Gain\_ratio(D,A)$`定义为：其信息增益$g(D,A)$与训练数据集$D$关于特征$A$的值的分裂信息`$split\_info(A)$`之比，即
$$Gain\_ratio(D,A)=\frac{Gain(D,A)}{split\_info(A)}$$
　　分裂信息定义如下：
$$
\begin{eqnarray*}
split\_info(A)&=&-\sum_{i=1}^np_ilog_2p_i
\\&=&-\sum_{i=1}^n\frac{|D_i|}{|D|}log_2\frac{|D_i|}{|D|}
\end{eqnarray*}
$$
　　其中，$n$是特征$A$的取值个数。`$split\_info(A)$`称为特征$A$的“固有值”(intrinsic value)。属性$A$的可能取值数目越多（即$n$越大），则`$split\_info(A)$`的值通常会越大。
　　**信息增益比准则对可能取值数目较少的特征有所偏好**，因此C4.5算法并不是直接选择增益率最大的候选划分特征，而是使用了一个启发式：**先从候选划分特征中找出信息增益高于平均水平的特征，再从中选择增益率最高的**。
## 三、决策树的生成
　　输入：训练数据集$D$，特征集$A$，信息增益阈值$\epsilon$
　　输出：决策树$T$
　　(1) 从根结点开始，数据集$D$全部分配在根结点；
　　(2) 若$D$中所有样本都属于同一类$C_K$，则将$C_k$作为该结点的类标记，并返回$T$；
　　(3) 若$A=\varnothing$，则将$D$中样本数最大的类$C_k$作为该结点的类标记，并返回$T$；
　　(4) 否则，计算特征集$A$中各特征对$D$的信息增益，选择信息增益最大的特征$Ag$；
　　(5) 如果特征$A_g$的信息增益小于阈值$\epsilon$，那么将$D$中样本数最大的类$C_k$作为该结点的类标记，并返回$T$；
　　(6) 否则，对特征$Ag$的每一可能值$a_i$，依照$A_g=a_i$将$D$分割为若干个非空子集$D_i$，并将$D_i$中样本数最大的类作为类标记构建子结点，由结点跟子结点组成树$T$，返回$T$；
　　(7) 对于第 $i$ 个子结点，以$D_i$为训练集，以$A-\{Ag\}$为特征集，递归地调用步骤(2)~(5)，得到子树$T_i$，返回$T_i$.
## 四、剪枝处理
　　剪枝是决策树学习算法对于“过拟合”的主要手段。决策树的剪枝策略有“预剪枝”和“后剪枝”。
### 4.1 预剪枝
　　预剪枝是指在决策树生成过程中，对每个结点在划分前先进行估计，若当前结点的划分不能带来决策树泛化性能提升，则停止划分并将当前结点标记为叶节点。
　　预剪枝使得决策树的很多分支没有“展开”，这不仅降低了过拟合的风险，还显著减少了决策树的训练时间开销，和测试时间开销；但另一方面，有些分支的当前划分虽不能提升泛化性能、甚至可能导致泛化性能暂时下降，但在其基础上进行的后续划分却有可能导致性能显著提高；预剪枝基于“贪心”本质禁止这些分支展开，给预剪枝决策带来了欠拟合的风险。
### 4.2 后剪枝
　　后剪枝是先从训练集生成一颗完整的决策树，然后自底向上地对非叶结点进行考察，若将该结点对应的子树替换为叶结点能带来决策树泛化性能提升，则将该子树替换为叶结点。
　　后剪枝决策树通常比预剪枝决策树保留了更多的分支。一般情况下，后剪枝决策树的欠拟合风险很小，泛化性能往往优于预剪枝决策树。但后剪枝过程是在生成决策树之后进行的，并且要自底向上地对树中的所有非叶结点进行逐一考察，因此其训练时间开销比未剪枝决策树和预剪枝决策树都要大得多。
　　注：对于如何判断决策树泛化性能能否提升，可采用留出法，即预留一部分数据用作“验证集”以进行性能评估。
## 五、连续值和缺失值处理
### 5.1 连续值处理
　　现实学习任务中经常会遇到连续属性，对于连续属性，采用连续属性离散化技术。C4.5决策树算法中采用最简单的策略——二分法对连续属性进行处理。将这一特征下的所有取值从小到大排序，然后依次选择相邻两个数的中值进行二元划分，计算信息增益比，从而选出最佳划分点。
### 5.2**缺失值处理（待补）**
　　C4.5采用了下述解决方案：
　　(1)对于如何在属性值缺失的情况下进行划分属性选择
　　(2)对于给定属性划分，若样本在该属性上的值缺失，如何对样本进行划分
## 六、常见问题
### 6.1 C4.5比起ID3的改进：
　　(1)在ID3中用信息增益选择特征时，会偏向于选择取值多的特征（不一定是最好的特征）；而采用信息增益比，可以削弱这种影响；
　　注：需要注意的是，信息增益比准则对可能取值数目较少的特征有所偏好，因此C4.5算法并不是直接选择增益率最大的候选划分特征，而是使用了一个启发式：先从候选划分特征中找出信息增益高于平均水平的特征，再从中选择增益率最高的。
　　(2)增加对连续值特征的处理步骤，将值排序，然后依次选择相邻两个数的中值进行二元划分，计算信息增益比，从而选出最佳划分点；
　　(3)**增加缺失值的处理步骤,（待补）**
### 6.2 决策树的优点
　　(1)计算简单，可解释性强；
　　(2)适合处理有缺失属性值的样本：空缺值相当于分裂时加多一个分支；
　　(3)能够处理不相关的特征.
### 6.3 决策树的缺点
　　(1)容易过拟合；
　　(2)不适合大样本数据集：每次分裂都需要遍历整个样本.
## 七、参考
- 李航，统计学习方法
- 周志华，机器学习
- [Nando de Freitas, CPS540](https://www.youtube.com/playlist?list=PLE6Wd9FR--EdyJ5lbFl8UuGjecvVw66F6)
- [林轩田，机器学习技法](https://www.youtube.com/playlist?list=PLXVfgk9fNX2IQOYPmqjqWsNUFl2kpk1U2)




