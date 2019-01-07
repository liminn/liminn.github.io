---
title: Softmax回归模型
mathjax: true
date: 2017-12-4 20:52:50
categories: 
- 机器学习
tags:
- softmax
- 多分类
---
Softmax回归模型是逻辑回归模型在多分类问题上的推广，在多分类问题中，类别标签$y$可以取$k$个不同的值（而不是2个）。
Softmax回归模型对于诸如MNIST手写数字分类等问题是很有用的，该问题的目的是辨识10个不同的单个数字，故有$k=10$个不同的类别。
# 1.模型
## 1.1 假设函数
假设函数：

$$
\begin{align}
h_\theta(x^{(i)}) =
\begin{bmatrix}
p(y^{(i)} = 1 | x^{(i)}; \theta) \\
,
p(y^{(i)} = 2 | x^{(i)}; \theta) \\
,
\dotsc \\
,
p(y^{(i)} = k | x^{(i)}; \theta)
\end{bmatrix}
=
\frac{1}{ \sum_{j=1}^{k}{e^{ \theta_j^T x^{(i)} }} }
\begin{bmatrix}
e^{ \theta_1^T x^{(i)} } \\
,
e^{ \theta_2^T x^{(i)} } \\
,
\dotsc \\
,
e^{ \theta_k^T x^{(i)} } \\
\end{bmatrix}
\end{align}
$$

<!-- more --> 
其中，$x=[x_0,x_1,...,x_n]^T\in R^{n+1}$, $x_0^{(i)}=1$(对应偏置项)；$\theta_1, \theta_2, \ldots, \theta_k \in R^{n+1}$是模型的参数，即每一个输出单元对应一个$(n+1)$维的参数$\theta$。
注意，$\frac{1}{ \sum_{j=1}^{k}{e^{ \theta_j^T x^{(i)} }} }$这一项对概率分布进行归一化，使得所有概率之和为1。
## 1.2 代价函数
**训练集**：$\{(x^{(1)},y^{(1)}),\ldots,(x^{(m)},y^{(m)})\}$，其中，$y^{(i)}\in \{ 1,2,\ldots,k \} $。（注意此处的类别下标从1开始，而不是0）。
**代价函数**：

$$
\begin{align}
J(\theta) = \frac{1}{m} \sum_{i=1}^{m} (-\log p(y^{(i)}|x^{(i)};\theta))\\
=- \frac{1}{m} \left[ \sum_{i=1}^{m} \sum_{j=1}^{k}  1\left\{y^{(i)} = j\right\} \log \frac{e^{\theta_j^T x^{(i)}}}{\sum_{l=1}^k e^{ \theta_l^T x^{(i)} }}\right]
\end{align}
$$

其中，$$\textstyle 1\{\cdot\}$$是示性函数，其取值规则为：$$\textstyle 1\{ 值为真的表达式 \textstyle \}=1$$，$$\textstyle 1\{ 值为假的表达式 \textstyle \}=0$$。

## 1.3 参数求解
目的：$\min\limits_\theta \ J(\theta)$
对于$\textstyle J(\theta)$的最小化问题，目前还没有闭式解法。因此，使用迭代的优化算法（例如梯度下降法，或L-BFGS）。
梯度公式如下：
$$
\begin{align}
\nabla_{\theta_j} J(\theta) = - \frac{1}{m} \sum_{i=1}^{m}{ \left[ x^{(i)} \left( 1\{ y^{(i)} = j\}  - p(y^{(i)} = j | x^{(i)}; \theta) \right) \right]  }
\end{align}
$$
其中，$$\textstyle \nabla_{\theta_j} J(\theta)$$本身是一个向量，它的第$\textstyle l$个元素$$\textstyle \frac{\partial J(\theta)}{\partial \theta_{jl}}$$是$$\textstyle J(\theta)$$对$$\textstyle \theta_j$$的第$$\textstyle l$$个分量的偏导数。
参数更新：$$\textstyle \theta_j := \theta_j - \alpha \nabla_{\theta_j} J(\theta)(\textstyle j=1,\ldots,k）$$。
## 1.4 正则化
L2正则化（权重衰减）：
$$
\begin{align}
J(\theta) = - \frac{1}{m} \left[ \sum_{i=1}^{m} \sum_{j=1}^{k} 1\left\{y^{(i)} = j\right\} \log \frac{e^{\theta_j^T x^{(i)}}}{\sum_{l=1}^k e^{ \theta_l^T x^{(i)} }}  \right]
              + \frac{\lambda}{2} \sum_{i=1}^k \sum_{j=0}^n \theta_{ij}^2
\end{align}
$$
- 增加权重衰减项$$\textstyle \frac{\lambda}{2} \sum_{i=1}^k \sum_{j=0}^{n} \theta_{ij}^2$$后，代价函数变成了严格的凸函数，可保证得到唯一解。此时的Hessian矩阵变为可逆矩阵，并且因为$$\textstyle J(\theta)$$是凸函数，梯度下降法和L-BFGS等算法可以保证收敛到全局最优解。
- 此时$$\textstyle J(\theta)$$的梯度计算，如下：
$$
\begin{align}
\nabla_{\theta_j} J(\theta) = - \frac{1}{m} \sum_{i=1}^{m}{ \left[ x^{(i)} ( 1\{ y^{(i)} = j\}  - p(y^{(i)} = j | x^{(i)}; \theta) ) \right]  } + \lambda \theta_j
\end{align}
$$

# 2.Softmax回归 vs k个二元分类器
如果你在开发一个音乐分类的应用，需要对$k$种类型的音乐进行识别，那么是选择使用softmax分类器，还是使用logistic回归算法建立 $k$个独立的二元分类器呢？
这一选择取决于你的**类别之间是否互斥**：
- 情况1：如果你有四个类别的音乐，分别为：古典音乐、乡村音乐、摇滚乐和爵士乐，那么你可以假设每个训练样本只会被打上一个标签，即一首歌只能属于这四种音乐类型的其中一种。此时你应该使用类别数$k=4$的softmax回归。
- 情况2：如果你的四个类别如下：人声音乐、舞曲、影视原声、流行歌曲，那么这些类别之间并不是互斥的。例如：一首歌曲可以来源于影视原声，同时也包含人声。这种情况下，使用4个二分类的logistic回归分类器更为合适。这样，对于每个新的音乐作品，算法可以分别判断它是否属于各个类别。

# 3.参考资料
- [UFLDL Tutorial, Softmax回归](http://ufldl.stanford.edu/wiki/index.php/Softmax%E5%9B%9E%E5%BD%92)
- [美团技术点评团队，Logistic Regression模型简介](https://tech.meituan.com/intro_to_logistic_regression.html)

