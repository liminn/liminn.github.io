---
title: MobileNetV1
mathjax: true
date: 2018-12-29 22:21:50
categories: 
- 轻量型CNN
tags:
---

# 摘要
- 针对移动端及嵌入式视觉应用，利用深度可分离卷积(depthwise separable convolutions)，构建轻量级的深度神经网络，即MobileNet。
- 其中引进了两个简单的超参数(宽度乘子(Width Multiplier)和分辨率乘子(Resolution Multiplier))来有效地权衡延迟(latency)和精度。
- 本质：只是用深度可分离卷积来替换常规CNN中的常规卷积
<!-- more -->

# 创新点
##网络结构##

<img src="/images/MobileNetV1/1.png"  width = "400">
<img src="/images/MobileNetV1/2.png"  width = "350">
<img src="/images/MobileNetV1/3.png"  width = "350">

- MobileNetV1结构如Table1所示：
 - **MobileNet基于深度可分离卷积构建，除了第一层为常规卷积以外**。
 - **如Fig3所示，所有层之后，均伴随BN和ReLU非线性操作**；除了最后的全连接层没有ReLU非线性，被喂给softmax层来进行分类。
 - **下采样操作通过depthwise卷积和第一层常规卷积中的带步长的卷积实现（注意这一点）**。
 - 最后的平均池化层使得空间分辨率在全连接层之前减小到1。
 - 若将depthwise卷积和pointwise卷积计为分离的层，那么MobileNet有28层。
- 如Table2所示，MobileNet花费95%的计算时间在$1\times 1$卷积上，并且$1\times 1$卷积占据了75%的参数。所有的其它参数，几乎都集中在全连接层上。

##宽度乘子(Width Multiplier)——更瘦的模型
- 第一个超参数宽度乘子$\alpha$,其作用是统一地使网络的每一层变瘦：
 - 对于一个给定层，在宽度乘子$\alpha$的作用下，输入的通道数由$M$变为$\alpha M$，输出的通道数由$N$变成$\alpha N$。
- 在宽度乘子$\alpha$的作用下：
 - 深度可分离卷积的计算量变为：$\alpha M \times {D_G}^2 \times  {D_K}^2 + \alpha M \times \alpha N \times {D_G}^2 $。
 - $\alpha \in (0,1]$ ，$\alpha$的典型设置为1，0.75，0.5和0.25。$\alpha=1$ 即是基准的MobileNet，$\alpha<1$即是简化的MobileNet。
 - 宽度乘子$\alpha$能够平方级地减少计算量和参数量，即约$\alpha ^2$。

##分辨率乘子(Resolution Multiplier)——减少表征**
- 第二个超参数分辨率乘子$\rho$，其作用是减少网络的计算量：
 - 对输入图片施加分辨率乘子$\rho$，那么网络每一层的中间表征都将随之减小相同的乘子。
 - 在实际操作中，通过设置输入分辨率，来隐式地设置$\rho$。
- 在宽度乘子$\alpha$和分辨率乘子$\rho$的作用下：
 - 深度可分离卷积的计算量变为：$\alpha M \times \rho^2{D_G}^2 \times  {D_K}^2 + \alpha M \times \alpha N \times \rho^2{D_G}^2 $。
 - $\rho \in (0,1]$ ，$\rho$通常隐式地设置，故网络的输入分辨率典型设置为224，192，160或128。$\rho=1$ 即是基准的MobileNet。
 - 分辨率乘子$\rho$能够平方级地减少计算量，即约$\rho ^2$。

# 训练策略
- MobileNet在TF上实现，通过和Inception V3相似的带有异步梯度下降的RMSprop实现。
- 用更少的正则化和更少的数据增强技术，因为小的网络不易过拟合。
- 用非常小或不使用权重衰减(L2正则)在depthwise卷积上很重要，因为它们只有过少的参数。


# 模型比较
<img src="/images/MobileNetV1/4.png"  width = "400"/>
 
 - 如Table4所示，比较了MobileNet与其全卷积实现形式：在ImageNet上，只损失了1%的精度，但节约了大量的计算量和参数。

<img src="/images/MobileNetV1/5.png"  width = "400"/>

- 如Table6所示，展示了通过不同的宽度乘子$\alpha$来收缩MobileNet时，精度、计算量和参数量之间的权衡。精度平滑的下降，直到模型结构过小，即$\alpha=0.25$时。
- 注：在ImageNet基准上，无论模型尺寸的大小，所有模型均用相同的训练参数进行训练。

<img src="/images/MobileNetV1/6.png"  width = "400"/>

- 如Table7所示，展示了通过施加不同的分辨率乘子$\rho$，即用减小的输入分辨率来训练MobileNet时，精度、计算量和参数量之间的权衡。准确率依据分辨率平滑下降。

<img src="/images/MobileNetV1/7.png"  width = "400"/>

- **如Table8所示，展示了完整MobileNet和GoogleNetV1、VGG16之间的比较**：
 - MobileNet精度接近于VGG16，同时参数量只是其$1/32$，计算量为其$1/27$。
 - MobileNet精度高于GoogleNetV1，同时参数量更少，计算量为其$1/2.7$。

<img src="/images/MobileNetV1/8.png"  width = "400"/>

- 如Table9所示，展示了通过$\alpha=0.5$且分辨率为$160\times 160$简化的MobileNet与AlexNet和Squeezenet之间的比较：
 - 简化版MobileNet比AlexNet的精度高3%，同时参数量只是其$1/45$，计算量为其$1/9.4$。
 - 简化版MobileNet比Squeezenet的精度高3%，同时参数量相同，计算量为其$1/22$。

此外，文中还展示了通过MobileNet，在细粒度分类(Fine Grained Recognition)、Large Scale Geolocalizaton、Face Attributes、目标检测(Object Detection)及Face Embeddings任务上的应用结果。

# 参考文献
- [论文：MobileNets: Efficient Convolutional Neural Networks for Mobile Vision Applications](https://arxiv.org/pdf/1704.04861.pdf)
- [视频：Depthwise Separable Convolution - A FASTER CONVOLUTION!](https://www.youtube.com/watch?v=T7o3xvJLuHk&t=0s&list=LLPZqbzAbJOX-hE-IRAxB7sg&index=2)


