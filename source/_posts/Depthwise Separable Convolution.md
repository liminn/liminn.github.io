---
title: Depthwise Separable Convolution
mathjax: true
date: 2018-12-29 22:20:50
categories: 
- 轻量型CNN
tags:
---

# 常规卷积
**示例**：
<img src="/images/Depthwise Separable Convolution/1.png"  width = "600"/>

- 上图为常规卷积示例，假设步长为1，same填充：
 - 单个核($D_k \times D_k \times M$)进行卷积后的输出为$D_G \times D_G \times 1$
 - $N$个核($D_k \times D_k \times M$)进行卷积后的输出为$D_G \times D_G \times N$
<!-- more -->

**计算量**：
<img src="/images/Depthwise Separable Convolution/2.png"  width = "600"/>

- 上图为常规卷积的计算量：
 - 单个核($D_k \times D_k \times M$)单次的乘法次数为${D_k}^2 \times M$
 - 单个核($D_k \times D_k \times M$)的总体乘法次数为${D_G}^2 \times {D_k}^2 \times M$
 - $N$个核($D_k \times D_k \times M$)的总体乘法次数为$N \times {D_G}^2 \times {D_k}^2 \times M$

# 深度可分离卷积
<img src="/images/Depthwise Separable Convolution/3.png"  width = "600"/>

- The first layer is called a depthwise convolution, it performs lightweight filtering by applying a single convolutional filter per input channel. 
- The second layer is a 1 × 1 convolution, called a pointwise convolution, which is responsible for building new features through computing linear combinations of the input channels.
- 上两段话摘自[MobileNetV2](https://arxiv.org/pdf/1801.04381.pdf)。
 
**depthwise convolution**：
<img src="/images/Depthwise Separable Convolution/4.png"  width = "600"/>
- 上图为depthwise convolution示例图，假设步长为1，same填充：
 - 单个核($D_k \times D_k \times 1$)进行卷积后的输出为$D_G \times D_G \times 1$（只对输入的一个通道进行操作）
 - $M$个核($D_k \times D_k \times 1$)进行卷积后的输出为$D_G \times D_G \times M$（每个核对输入的一个通道进行操作）

**pointwise convolution**：
<img src="/images/Depthwise Separable Convolution/5.png"  width = "600"/>
- 注：图中的Filtering Stage应改为Combination Stage
- 上图为pointwise convolution的示例图，假设步长为1，same填充：
 - 单个核($1 \times 1 \times M$)进行卷积后的输出为$D_G \times D_G \times 1$
 - $M$个核($1 \times 1 \times M$)进行卷积后的输出为$D_G \times D_G \times M$
 - 就是$1\times1 $卷积

**计算量**：
<img src="/images/Depthwise Separable Convolution/6.png"  width = "600"/>

# 常规卷积与深度可分离卷积的计算量对比
<img src="/images/Depthwise Separable Convolution/7.png"  width = "600"/>
- 以$N=1024$, $D_k=3$为例：
 - 深度可分离卷积的计算量约为常规卷积的$\frac{1}{10}$

# 常规卷积与深度可分离卷积的参数个数对比
<img src="/images/Depthwise Separable Convolution/8.png"  width = "600"/>
- 参数个数对比同计算量对比

# 参考文献
- [视频：Depthwise Separable Convolution - A FASTER CONVOLUTION!](https://www.youtube.com/watch?v=T7o3xvJLuHk&t=0s&list=LLPZqbzAbJOX-hE-IRAxB7sg&index=2)



















