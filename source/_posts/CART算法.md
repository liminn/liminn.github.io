﻿---
title: CART算法
mathjax: true
date: 2017-12-7 22:30:50
categories: 
- 机器学习
tags:
- CART
- 二叉树
- 回归树
- 分类树
- 平方误差
- 基尼指数
---
　　CART(classification and regression tree)是广泛应用的决策树学习方法，既可以用于分类也可以用于回归。
## 一、简介
（1）CART决策树的定义：CART假设决策树是**二叉树**，内部结点特征的取值只有“是”和“否”，左分支是取值为“是”的分支，右分支是取值为“否”的分支。这样的决策树等价于递归地二分每个特征，将输入空间即特征空间划分为有限个单元，并在这些单元上确定预测的概率分布，也就是在输入给定的条件下输出的条件概率分布。
- 注：ID3/C4.5算法中决策树的每个叶子结点对应一个决策。CART算法中分类回归树的每个叶子结点给出一个预测分数(score)。

（2）CART决策树的生成：CART决策树的生成就是递归地构建二叉决策树的过程，对**回归树**用**平方误差最小化准则**，对**分类树**用**基尼指数最小化准则**，进行特征选择，生成二叉树。
（3）CART决策树的学习：同样包括三个步骤，特征选择、树的生成及剪枝。
<!-- more --> 
## 二、回归树的生成
　　**一个回归树**对应着**特征空间的一个划分**以及在**划分的单元上的输出值**；假设已将输入空间划分为$M$个单元$R_1,R_2,...,R_m$，并且在每个单元$R_m$上都有一个固定的输出值$c_m$，于是回归树的模型可表示为：
$$f(x) = \sum_{m=1}^Mc_m I(x\in R_m)$$
　　当输入空间的划分确定时，CART回归树用平方误差$\sum\limits_{x_i\in R_m}(y_i-f(x_i))^2$来表示回归树对于训练集的预测误差，用平方误差最小的准则求解每个单元上的最优输出值$\hat{c}_m$。
- **注**：$\sum\limits_{x_i\in R_m}(y_i-f(x_i))^2$即为求解单元$R_m$上的最优输出值$\hat{c}_m$时的代价函数。

　　易知，单元$R_m$上的$c_m$的最优值$\hat{c}_m$是$R_m$上的所有输入实例$x_i$对应的输出$y_i$的均值，即
$$\hat{c}_m=ave(y_i|x_i \in R_m)$$
### 2.1 最小二乘回归树生成算法
输入：训练数据集$D$
输出：回归树$f(x)$
在训练数据集所在的特征空间中，递归地将每个区域划分为两个子区域并决定每个子区域上的输出值，构建二叉决策树；
（1）选择最优切分变量 $j$ 和切分点$s$ ，求解：
$$\min_{j,s}[\min_{c_1} \sum_{x_i\in R_1(j,s)}(y_i-c_1)^2 + \min_{c_2} \sum_{x_i\in R_2(j,s)}(y_i-c_2)^2]$$
遍历变量$j$，对固定的切分变量$j$扫描切分点$s$，选择使上式预测误差（平方误差）达到最小值的对$(j，s)$；其中$R_m$是被划分的输入空间，$c_m$是空间$R_m$对应的固定输出值。
- **注**：$$[\min_{c_1} \sum_{x_i\in R_1(j,s)}(y_i-c_1)^2 + \min_{c_2} \sum_{x_i\in R_2(j,s)}(y_i-c_2)^2]$$即为确定变量$j$上的切分点$s$时的代价函数。

（2）用选定的对$(j,s)$划分区域并决定相应的输出值：
$$R_1(j,s)=\lbrace x\mid x^{(j)} \le s \rbrace , \quad R_2(j,s)=\lbrace x\mid x^{(j)}\gt s \rbrace\\
\hat c_m = {1\over N_m}\sum_{x_i\in R_m(j,s)}y_i , \quad x\in R_m ,\quad m=1,2$$
（3）继续对两个子区域调用步骤（1），（2），直至满足停止条件。
（4）将输入空间划分为$M$个区域$R_1,R_2,\ldots,R_M$，生成决策树：
$$f(x) = \sum_{m=1}^M\hat c_m I(x\in R_m)$$

### 2.2 回归树示例
　　书和网络资料中，关于CART回归树的讲解极少，举例解释回归树生成算法。本题改编自《统计学习方法》的“提升树”章节中，求回归树也是回归问题的提升树算法的第一个步骤。
　　题目描述：训练数据见下表，$x$的取值范围为区间$[0.5,10.5]$,$y$的取值范围为区间$[5.0,10.0]$,学习这个回归问题的CART回归树**树桩**模型。
　　注：(1)该题为了简化计算过程，训练样本只有一个特征变量，即$j=1$，故省去了遍历变量$j$的步骤；(2)该题只需求一个CART回归树的树桩，即只需对数据集划分一次，产生一个内部结点和两个叶结点即可。

|$x_i$|1|2|3|4|5|6|7|8|9|10|
|:--:|:--:|:--:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|$y_i$|5.56|	5.70|	5.91|	6.40|	6.80|	7.05|	8.90|	8.70|	9.00|	9.05|

首先通过以下优化问题：
$$\min_{j,s}[\min_{c_1} \sum_{x_i\in R_1(j,s)}(y_i-c_1)^2 + \min_{c_2} \sum_{x_i\in R_2(j,s)}(y_i-c_2)^2]$$
求解训练数据的切分点$s$:
$$R_1=\lbrace x\mid x\le s\rbrace , \quad R_2=\lbrace x\mid x\gt s\rbrace$$
容易求得在$R_1$,$R_2$内部使得平方损失误差达到最小值的$c_1$,$c_2$为：
$$c_1={1\over N_1}\sum_{x_i \in R_1}y_i , \quad c_2={1\over N_2}\sum_{x_i \in R_2}y_i$$
这里$N_1$,$N_2$是$R_1$,$R_2$的样本点数。
求训练数据的切分点，根据所给数据，考虑如下切分点：
$$1.5 ,2.5 ,3.5 ,4.5 ,5.5 , 6.5 , 7.5 , 8.5 , 9.5$$
对各切分点，不难求出相应的$R1$ , $R2$ , $c1$ , $c2$及
$$m(s)=\min_{j,s}[\min_{c_1} \sum_{x_i\in R_1(j,s)}(y_i-c_1)^2 + \min_{c_2} \sum_{x_i\in R_2(j,s)}(y_i-c_2)^2]$$
例如，当$s=1.5$时，$R_1 = \lbrace 1\rbrace$, $R_2 = \lbrace 2, 3 , \ldots , 10\rbrace$ , $c_1 = 5.56$ , $c_2 = 7.50$ ,
$$m(s)=\min_{j,s}[\min_{c_1} \sum_{x_i\in R_1(j,s)}(y_i-c_1)^2 + \min_{c_2} \sum_{x_i\in R_2(j,s)}(y_i-c_2)^2] = 0+15.72 = 15.72$$
现将$s$及$m(s)$的计算结果列表如下：

|$s$| 	1.5|	2.5|	3.5|	4.5|	5.5|	6.5|	7.5|	8.5|	9.5|
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|$m(s)$|	15.72|	12.07|	8.36|	5.78|	3.91|	1.93|	8.01|	11.73|	15.74|

由上表可知，当$s=6.5$的时候达到最小值，此时$R_1 = \lbrace 1 ,2 , \ldots , 6\rbrace$, $R_2 ={7 ,8 ,9 , 10}$ , $c_1=6.24$, $c_2=8.9$, 所以回归树$T_1(x)$为：
$$T_1(x) =
\begin{cases}
6.24, & x\lt 6.5 \\
8.91, & x \ge 6.5 \\
\end{cases}$$
$$f(x) = \sum_{m=1}^2c_m I(x\in R_m)$$

## 三、分类树的生成
分类数用基尼系数选择最优特征，同时决定该特征的最优二值切分点。
###3.1 基尼指数
分类问题中，假设有$K$个类，样本点属于第$k$类的概率为$p_k$，则概率分布的基尼指数定义为:
$$Gini(p) = \sum\limits^{K}_{k = 1} p_k(1-p_k)= 1 -  \sum\limits^{K}_{k = 1}p_k^2 $$
对于二分类问题，若样本点属于第1个类的概率是$p$，则概率分布的基尼指数为
$$Gini(p)=2p(1-p)$$
对于给定的样本集合$D$，其基尼指数为
$$Gini(D)= 1 -  \sum\limits^{K}_{k = 1} (\frac{|C_k|}{|D|})^2$$
这里，$C_k$是$D$中属于第$k$类的样本子集，$K$是类的个数。
如果样本集合$D$根据特征$A$是否取某一可能值$a$被分割成$D_1$和$D_2$两部分，即
$$D_1=\{(x,y)\in D|A(x)=a\}, \quad D_2=D-D_1$$
则在特征$A$的条件下，集合$D$的基尼指数定义为
$$Gini(D, A) =  \frac{|D_1|}{|D|}Gini(D_1) + \frac{|D_2|}{|D|}Gini(D_2)\tag{2.1}$$
基尼指数$Gini(D)$表示集合$D$的不确定性，基尼指数$Gini(D,A)$表示经$A=a$分割后，集合$D$的不确定性。**基尼指数值越大，样本集合的不确定性也越大，这一点与熵相似**。
### 3.2 CART分类树生成算法
输入：训练集$D$，停止计算的条件
输出：CART决策树
（1）设结点的训练数据集为$D$，计算现有特征对该数据集的基尼指数，此时对每一个特征$A$，对其可能取得每个值$a$，根据样本点对$A=a$的测试为“是”或者“否”将$D$分割为$D_1$和$D_2$两部分，利用式（2.1）计算$A=a$时的基尼指数；
（2）在所有可能的特征$A$以及它们所有可能的切分点$a$中，选择基尼指数最小的特征及其对应的切分点作为最优特征与最优切分点。依最优特征与最优切分点，从现结点生成两个子结点，将训练集依特征分配到两个子结点中去；
（3）对两个子结点递归地调用（1），（2）直至满足停止条件；
（4）生成CART决策树；
算法停止计算的条件是结点中的样本个数小于预定阈值，或样本集的基尼指数小于预定阈值（也就是样本基本属于同一类），或者没有更多特征。
###3.3 示例
见李航的《统计学习方法》书中。
## 四、剪枝
暂略，参照另一篇总结:[ID3/C4.5算法](http://zhanglimin.com/2017/12/06/ID3%E5%92%8CC4.5%E7%AE%97%E6%B3%95/)。
## 五、树模型的优缺点
树模型的优点：
- 容易解释
- 不要求对特征做预处理
 – 能处理离散值和连续值混合的输入
 – 对特征的单调变换不敏感(只与数据的排序有关)
 – 能自动进行特征选择
 – 可处理缺失数据
- 可扩展到大数据规模

树模型的缺点：
- 正确率不高：建树过程过于贪心
 – 可作为Boosting的弱学习器（深度不太深）
- 模型不稳定（方差大）：输入数据小的变化会带来树结构的变化
 – Bagging：随机森林
- 当特征数目相对样本数目太多时，容易过拟合

## 六、参考
- 李航，统计学习方法
- [林轩田，机器学习技法](https://www.youtube.com/playlist?list=PLXVfgk9fNX2IQOYPmqjqWsNUFl2kpk1U2)
- 周志华，机器学习
- [mathematicalmonk, Machine Learning(2.1-2.5)](https://www.youtube.com/playlist?list=PLD0F06AA0D2E8FFBA)





