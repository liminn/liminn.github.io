---
title: EM算法
mathjax: true
top: true
date: 2018-2-2 22:08:50
categories: 
- 机器学习
---
　　EM(expectation maximization)是一种迭代算法，求含有隐变量的概率模型参数的极大似然估计法。每次迭代包含两步：E步，求期望；M步，求极大化。
<!-- more --> 
# 1.符号表示
　　一般地，用$Y$表示观测随机变量的数据，$Z$表示隐随机变量的数据。$Y$和$Z$连在一起称为完全数据，观测数据$Y$又称为不完全数据。
　　假设给定观测数据$Y$，其概率分布是$P(Y|\theta)$，其中$\theta$是需要估计的模型参数，那么不完全数据$Y$的似然函数是$P(Y|\theta)$，对数似然函数$L(\theta)= logP(Y|\theta)$。
　　假设$Y$和$Z$的联合概率分布是$P(Y,Z|\theta)$，那么完全数据的对数似然函数是$logP(Y,Z|\theta)$。
# 2.Q函数
　　完全数据的对数似然函数$logP(Y,Z|\theta)$关于在给定观测数据$Y$和当前参数$\theta^{(i)}$下对未观测数据$Z$的条件概率分布$P(Z|Y,\theta^{(i)})$的期望称为Q函数，即：
$$Q(θ,θ^{(i)}) = E_z[\log P(Y,Z|θ) | Y, θ^{(i)}] = \sum_Z \log P(Y,Z|θ)P(Z|Y,θ^{(i)})$$
# 3.EM算法
输入: 观测变量$Y$，隐变量数据$Z$，联合分布$P(Y,Z|θ)$，条件分布$P(Z|Y,θ)$；
输出: 模型参数$θ$
(1) 选择参数的初始值$θ^{(0)}$，开始迭代。
说明：参数的初值可以任意选择，但需注意EM算法对初值是敏感的。
(2) E步: 记$\theta^{(i)}$为第$i$次迭代参数$\theta$的估计值, 在第$i+1$次迭代的E步，计算
$$\begin{align}
Q(\theta, \theta^{(i)}) &= E_Z [log  P(Y, Z | \theta) | Y, \theta^{(i)}]\\ &= \sum\limits_Z log  P(Y, Z | \theta) P(Z|Y,\theta^{(i)} ) 
\end{align} $$
这里$P(Z|Y,θ^{(i)})$是在给定观测数据$Y$和当前的参数估计$θ^{(i)}$下隐变量数据$Z$的条件概率分布。
说明：E步求$Q(\theta, \theta^{(i)})$，Q函数式中$Z$是未观测数据，$Y$是观测数据。注意，$Q(\theta, \theta^{(i)})$的第一个变元表示要极大化的参数，第2个变元表示参数的当前估计值。每次迭代实际在求Q函数及其极大。
(3) M步: 求使$Q(θ,θ_i)$极大化的$\theta$,确定第$i+1$次迭代的参数的估计值$θ^{(i+1)}$
$$\theta^{(i+1)} = arg\max\limits_\theta Q(\theta, \theta^{(i)})$$
说明：M步求$Q(\theta, \theta^{(i)})$的极大化，得到$\theta^{(i+1)}$，完成一次迭代$\theta^{(i)}$->$\theta^{(i+1)}$。每次迭代使得似然函数增大或达到局部极值。
(4) 重复(2), (3)两步, 直到收敛。
说明：给出停止迭代条件，一般是对较小的正数$\xi_1,\xi_2$，若满足$||θ^{(i+1)}-θ^{(i)}||<\xi_1$或者$||Q(θ^{(i+1)},θ^i)-Q(θ^{(i)},θ^{(i)})|| < \xi_2$则停止迭代。
# 4.Jensen不等式
## 凸函数
函数$f(x)$是凸函数：
- 当定义域为实数，即$x\in R$，对于所有$x$，$f(x)$的二阶导数$f^{''}(x) \geq 0$；
- 当$x$是向量时，该函数的$hessian$矩阵$H$是半正定，即$H\geq 0$；

函数$f(x)$是严格凸函数：
- 当对于所有$x$，$f^{''}(x) > 0 (x \in R)$或$H$正定，即$H>0$（$x$为向量）。

## 凹函数
函数$f(x)$是（严格）凹函数,当且仅当$-f(x)$是（严格）凸函数。
## Jensen不等式
Jensen不等式：
- 当$f$是凸函数，$X$是随机变量，则$E[f(X)] \ge f(EX)$
 - 此外，当$f$是严格凸函数，$E[f(X)] = f(EX)$当且仅当$X=E(X)$的概率为1（例如$X$是一个常数）的时候成立。
- 当$f$是凹函数，不等式方向反向:$E[f(X)] \le f(EX)$

示例（记忆方法）：
<img src="http://ozruihqgo.bkt.clouddn.com/%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0/EM%E7%AE%97%E6%B3%95/jensen%E4%B8%8D%E7%AD%89%E5%BC%8F.JPG" width="50%" height="50%">  
- 上图中，$f$是一个凸函数，在图中用实线表示。另外$X$是一个随机变量，有0.5的概率取值为$a$，另外有0.5的概率取值为$b$，已在图中$x$轴上标出。这样，$X$的期望值$E(X)$也就在图中所示的$a$和$b$的中点位置。
- 图中在$y$轴上标出了$f(a)$、$f(b)$和$f(EX)$。接下来函数的期望值$E[f(x)]$在$y$轴上就处于$f(a)$和$f(b)$之间的中点的位置。
- 如图所示，在这个例子中由于$f$是凸函数，很明显$E[f(X)] \ge f(EX)$。

# 5.Q函数推导
面对一个含有隐变量的概率模型，目标是极大化观测数据（不完全数据）$Y$关于参数$\theta$的对数似然函数，即极大化
$$\begin{align} L(\theta) &= log  P(Y | \theta) = log \sum\limits_Z P(Y, Z | \theta) \\ &= log ( \sum\limits_Z   P(Y|Z,\theta) P(Z |\theta))  \end{align}$$
注意到这一极大化的主要困难是式中有未观测数据并有包含和（或积分）的对数。
事实上，EM算法是通过迭代逐步近似极大化$L(\theta)$的。假设在第$i$次迭代后$\theta$的估计值是$\theta^{(i)}$。我们希望新估计值$\theta$能使$L(\theta)$增加，即$L(\theta)>L(\theta^{(i)})$，并逐步达到极大值。为此，考虑两者的差：
$$
L(\theta) - L(\theta^{(i)}) = log (\sum\limits_z P(Y|Z,\theta) P(Z |\theta)) - log  P(Y | \theta^{(i)}) 
$$
利用Jensen不等式得到其下界：
$$\begin{split}
L(\theta) - L(\theta^{(i)}) &= log (\sum\limits_zP(Y|Z,\theta) P(Z |\theta)) - log  P(Y | \theta^{(i)}) 
\\ &= log(\sum\limits_Z P(Z|Y,\theta^{(i)} ) \frac {P(Y|Z,\theta) P(Z |\theta)}{P(Z|Y,\theta^{(i)})}) - log  P(Y | \theta^{(i)}) \\ &\ge \sum\limits_Z P(Z|Y,\theta^{(i)} ) log\frac {P(Y|Z,\theta) P(Z |\theta)}{P(Z|Y,\theta^{(i)})} - log  P(Y | \theta^{(i)}) \\ &= \sum\limits_Z P(Z|Y,\theta^{(i)} ) log \frac {P(Y|Z,\theta) P(Z |\theta)}{P(Z|Y,\theta^{(i)}) P(Y|\theta^{(i)})}
\end{split}
$$
令
$$B(\theta, \theta^{(i)}) = L(\theta^{(i)}) + \sum\limits_Z P(Z|Y,\theta^{(i)} ) log \frac {P(Y|Z,\theta) P(Z |\theta)}{P(Z|Y,\theta^{(i)}) P(Y|\theta^{(i)})} 
\tag{13}$$
则
$$L(\theta)\geq B(\theta, \theta^{(i)})$$
即函数$B(\theta, \theta^{(i)})$是$L(\theta)$的一个下界，而且由式（13）可知，
$$L(\theta^{(i)})= B(\theta^{(i)}, \theta^{(i)})$$
因此，任何可以使$B(\theta, \theta^{(i)})$增大的$\theta$，也可以使$L(\theta)$增大。为了使$L(\theta)$有尽可能大的增长，选择$\theta^{(i+1)}$使$B(\theta, \theta^{(i)})$达到极大，即
$$\theta^{(i+1)} = arg\max\limits_\theta B(\theta, \theta^{(i)})$$
现在求$\theta^{(i+1)}$的表达式。省去对$\theta$极大化而言是常数的项，有
$$\begin{split}
\theta^{(i+1)} &= arg\max\limits_\theta\ B(\theta, \theta^{(i)}) \\ 
&= arg\max\limits_\theta \ L(\theta^{(i)}) + \sum\limits_Z P(Z|Y,\theta^{(i)} ) log \frac {P(Y|Z,\theta) P(Z |\theta)}{P(Z|Y,\theta^{(i)}) P(Y|\theta^{(i)})}  \\
&=arg\max\limits_\theta\ \sum\limits_Z P(Z|Y,\theta^{(i)} ) log(P(Y| Z, \theta)P(Z|\theta)) \\
&= arg\max\limits_\theta\ \sum\limits_Z P(Z|Y,\theta^{(i)} ) log  P(Y, Z | \theta) \\ 
&= arg\max\limits_\theta\ Q(\theta, \theta^{(i)})
\end{split} $$
上式等价于EM算法的一次迭代，即求Q函数及其极大化。**EM算法是通过不断求解下界的极大化逼近求解对数似然函数极大化的算法**。

# 6.参考资料
- [Andrew Ng, CS229](http://open.163.com/movie/2008/1/O/T/M6SGF6VB4_M6SGKGMOT.html)
- [CS229, notes8](http://cs229.stanford.edu/notes/cs229-notes8.pdf)
- 李航，统计学习方法
- [Richard Xu，Expectation Maximization](https://www.youtube.com/watch?v=Bq5s80ZCmC0&t=4s)


