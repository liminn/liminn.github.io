---
title: SegNet
mathjax: true
date: 2019-01-09 22:26:50
categories: 
- 语义分割
tags:
---

# 摘要

- 针对道路场景理解(scene understanding)应用，为了在推理时的内存和计算时间方面都更有效率，设计了SegNet。
- SegNet由编码器网络(an encoder network)和解码器网络(a corresponding decoder network)构成：
 - 编码网络为VGG16的前13个卷积层，移除了VGG16中的全连接层；
 - 解码网络和编码网络的每一层一一对应，解码网络也有13层，解码网络中的上采样方式采用unpooling。

<!-- more -->

# 创新点

## unpooling操作
<img src="/images/SegNet/1.png"  width = "800" height = "200"/>

如Fig3所示，为unpooling操作示意图，从下往上看：

- $a$,$b$,$c$,$d$对应于特征图的值，依据Max-pooling Indices，来对特征图进行上采样，得到稀疏的特征图，然后通过卷积(带有可学习的卷积核),来得到密集的特征特征图。


## SegNet结构
<img src="/images/SegNet/2.png"  width = "800" height = "200"/>

如Fig2所示，为SegNet结构：
对于编码器网络，编码网络为VGG16的前13个卷积层，移除了VGG16中的全连接层：
- 采用VGG16中的前13个卷积层，因此可以用分类任务中的预训练权重来初始化训练过程
 - 对于卷积层，只使用$f=3$，$s=1$，`same`填充,每一卷积层均为：`Conv`+`BN`+`ReLU`
 - 对于池化层，只使用$f=2$，$s=2$的最大池化层(没有重叠窗口)，使得空间尺寸减半
- 抛弃全连接层的做法：
 - 不仅在编码器最深层的输出保持了更高的分辨率特征图
  - 而且显著减少了SegNet编码网络中的参数(从134M到14.7M)

对于解码器网络，解码网络和编码网络的每一层一一对应，解码网络也有13层，解码网络中的上采样方式采用unpooling：
- 解码器采用对应的编码器中的max-pooling操作计算得到的最大池化索引(max-pooling indices)，来对输入特征图执行非线性上采样(non-linear upsampling)；
 - 对编码器的每一个特征图，存储每一个池化窗口的最大特征值的位置;对于每一个2x2的池化窗口，只需要2bits；
- 该上采样方法不需要学习，上采样得到的特征图是稀疏的，然后经过后续卷积(带有可训练的卷积核)来得到密集的特征图；
- 上采样后的特征图是稀疏的(sparse)，然后通过可训练的卷积核进行卷积，来得到密集的(dense)特征图。
- 在解码过程中，重用(reusing)max-pooling索引，有几个实际的优点：
 - 1.改善边缘描绘;
 - 2.减少了参数数量；
 - 3.该上采样方式可以施加在任意编码解码结构中

相比于UNet： 

- UNet没有重用最大池化索引，而是将整个特征图（需要消耗更多的内存）传输给对应解码器，并和经过上采样(通过转置卷积)的解码器特征图进行串联。

# 模型比较
<img src="/images/SegNet/3.png"  width = "800" height = "200"/>
如Table3所示，在CamVid数据集上，比较了SegNet和其他网络的表现：
- `40K`，`80K`及`>80K`意为迭代次数
- DeconvNet的评价指标上的表现接近SegNet，但它的计算开销比SegNet更大。

<img src="/images/SegNet/4.png"  width = "800" height = "200"/>
如Table6所示，为多种网络结构对于计算时间和硬件资源需求的统计对比：
- SegNet在推理所占显存方面，最小。
- DeepLabV1在前向传播时间、反向传播时间、训练所占显存以及模型大小方面，均为最优。
- 疑问：SegNet的推理时间比FCN还大，这不是打自己脸吗？

# 总结

- FCN、UNet及SegNet本质上还是为了更好的效果，SegNet的效果确实是当时最优；在效率/性能方面，它们都不行。
- 在效果方面：`SegNet` >= `DeconvNet` > `DeepLabV1` > `FCN`

# 参考文献

- [论文：SegNet: A Deep Convolutional Encoder-Decoder Architecture for Image Segmentation](https://arxiv.org/pdf/1511.00561.pdf)
- [官方代码：alexgkendall/caffe-segnet](https://github.com/alexgkendall/caffe-segnet)
- [代码及注释：meetshah1995/pytorch-semseg](https://github.com/liminn/pytorch-semseg/blob/master/ptsemseg/models/segnet.py)


