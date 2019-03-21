---
title: FPN
mathjax: false
date: 2019-01-18 23:26:50
categories: 
- 目标检测
tags:
---

# 摘要

- FPN(Feature Pyramid Networks)的目的是通过特征金字塔来解决检测任务中的多尺度目标问题。
- `FPN` = `bottom-up pathway` + `top-down pathway` + `lateral connections `
 - `bottom-up pathway`：任意backbone ConvNet，文中通过ResNet，构成自下而上的路径。
 - `top-down pathway`：通过对`bottom-up pathway`中各金字塔层级的特征进行逐步上采样，构成自上而下的路径。
 - `lateral connections`：横向连接将`bottom-up pathway`与`top-down pathway`中对应层级的特征相加，弥补丢失的空间信息，增强定位能力。
- 文中将FPN应用于RPN以及Fast R-CNN，大幅提升了原有效果。

<!-- more -->

# 创新点
## FPN总体结构

<img src="/images/FPN/2.png"  width = "450" height = "400"/>
如上图所示，为FPN的总体结构示意图：
首先，见总体结构示意图中的"bottom-up"部分所示：
- Bottom-up pathway即为主干网络，文中选用ResNet；ResNet由`conv1`、`conv2_x`、`conv3_x`、`conv4_x`、`conv5_x`构成；
- 文中选用`conv2_x`、`conv3_x`、`conv4_x`、`conv5_x`的输出特征$\{C_2,C_3,C_4,C_5\}$为四个金字塔层级，其输出步长分别为$\{4,8,16,32\}$。
- 文中未将`conv1`的输出特征包含进金字塔之中，是因为其需要大量的内存占用。

然后，见总体结构示意图中的"top-down"部分所示：
- 首先，对$C_5$施加$1\times1$卷积(通道数固定为$d=256$)，得到$M_5$
- 然后，进行横向连接，将$M_5$进行上采样(最近邻插值)后的特征图，与$C_4$进行$1\times1$卷积(通道数固定为$d=256$)后的特征图，进行像素级加法(element-wise addition)，即得到$M_4$；依此类推，得到$M_3$和$M_2$
- **为什么要横向连接**：将$M_5$进行上采样(最近邻插值)后的特征图记为$M_{4before}$。$M_{4before}$相比于$C_4$，$M_{4before}$的**语义信息更丰富(分类能力强)，但空间信息更粗糙(定位能力弱)**。因此，将$C_4$于$M_{4before}$进行连接，补充$M_{4before}$的空间信息，弥补其定位能力的不足。

最后，见总体结构示意图中的最右侧一列：
- **为了减少上采样带来的混叠影响(reduce the aliasing effect of upsampling)**：对$M_4$进行$3\times 3$卷积，得到$P_4$。依此类推，得到$P_3$以及$P_2$。
- FPN各金字塔层级的最终特征图即为${P_5,P_4,P_3,P_2}$，分别对应于${C_5,C_4,C_3,C_2}$，它们的空间尺寸，对应相等。

疑问：文中说所有的金字塔层级共享分类器/回归器，那是如何训练的，是如何做出预测的

## FPN for RPN
RPN是一个滑动窗口类型的不区分类别的目标检测器(a sliding-window class-agnostic object detector)：
- 在原始的RPN设计中，其在CNN顶部的单尺度的特征图上，施加$3\times3$的滑动窗口，然后进行目标/非目标的二分类以及包围边框回归。
- 通过如下方式实现：在CNN顶部的单尺度的特征图上，施加一个$3\times3$的卷积层，后接两个同级的$1\times1$卷积层，分别用来做分类与回归。
- 注意：上述的一个$3\times3$的卷积层和两个$1\times1$卷积层，称为`RPN head`。
- RPN中目标/非目标的分类以及包围边框回归的目标是预先定义的`anchors`，`anchors`设定了多种尺度和宽高比，用来覆盖不同形状的目标。

**将FPN的思想用于RPN，即是替换掉RPN中的单一尺度的特征图**：
- 首先，**在FPN的每一个特征金字塔层级后面添加`RPN head`(一个$3\times 3$卷积和两个$1\times 1$卷积)**。
- 然后，因为`RPN head`会在所有特征金字塔层级的所有位置进行滑动。因此，没有必要再在一个特定金字塔层级上使用多尺度的`anchors`。
- 因此，在每一个特征金字塔层级上使用一个单一的尺度的`anchors`。具体地，定义面积为$\{32^2,64^2,128^2,256^2,512^2\}$像素的`anchors`，分别在$\{P_2,P_3,P_4,P_5,P_6\}$上单独使用。
- 注：引入$P_6$只是为了覆盖更大的$512^2$的anchor尺度,$P_6$是$P_5$的下采样特征。$P_6$没有在下面的Fast R-CNN检测器中被使用。
- 同时，与Faster R-CNN一样，对每一个尺度的`anchors`均使用3种宽高比:$\{1:2,1:1,2:1\}$。因此，在所有的金字塔层级中，共使用了15种`anchors`。

对于每个anchors的训练标签，同Faster R-CNN一样：
- anchor设定为正标签：当其与一个GT边框有着最高的IoU，或者其与任意GT边框的IoU均高于0.7
- anchor设定为负标签：当其与所有的GT边框的IoU均小于0.3

小结：
- **FPN中不同层级的特征金字塔，有着不同尺度的感受野。因此，将`RPN head`添加在所有特征金字塔层级之后的意图即是想利用起所有尺度的感受野**。
- **不同尺度的感受野适合于检测不同尺度的`anchors`。因此，便在不同的特征金字塔层级上使用符合其感受野大小的特定尺度的`anchors`**。

疑问：文中说所有特征金字塔层级上的`RPN head`共享参数，那是如何训练的，是如何做出预测的

## FPN for Fast R-CNN
Fast R-CNN是基于区域的目标检测器(a region-based object detector)，其通过RoIPool来提取特征。
Fast R-CNN在单一尺度的特征图上进行检测，因此，**将FPN用于Fast R-CNN，即是给各金字塔层级分配不同尺度的RoI**：
- 具体地，将输入图片中$w\times w$的RoI，分配给$P_k$的特征金字塔层级，其中$k =  \lfloor k_0 + log_2(\sqrt{wh}/224) \rfloor$
 - $K_0$为当RoI的宽高为$w\times h = 224^2$时的目标金字塔层级。因为Faster R-CNN将$C_4$作为单一的特征尺度(是吗，待确认，$C_5$为什么不用？)，因此文中将$K_0$设为4
 - 因此，当RoI的尺寸变得更小时(如$112\times 112$)，它应该被分配到更低的层级(如$k=3$)

然后，在所有金字塔层级的所有RoI特征之后，添加`predictor heads`:
- 即在采用RoIPool进行$7\times 7$的特征提取之后，添加两个1024维度的全连接隐藏层，分别接分类器和包围边框回归器。
- 其中，这两个全连接层被随机初始化，因为ResNet中没有该层

疑问：文中说所有的`predictor heads`共享参数，那是如何训练的，是如何做出预测的

# 实验结果
<img src="/images/FPN/3.png"  width = "800" height = "400"/>
如Table1所示，为FPN与原始RPN的AR(Average Recall)指标对比：
- 对比(a)(b)(c)，可以发现：
 - FPN显著高于原始的RPN，尤其是对于小物体和中等物体，见$AR_s$和$AR_m$。
 - 另外(a)和(b)的对比可以看出高层特征并非比低一层的特征有效。 
- 观察(d)：(d)表示只有横向连接，而没有`top-down pathway`，也就是仅对`bottom-up pathway`的每一层结果做一个$1\times1$的卷积和$3\times3$的卷积得到最终的结果。由于`bottom-up pathway`中不同层之间的semantic gaps比较大，导致特征金字塔仍然工作的不好(分类器和回归器共享参数，也做了不共享参数的实验，也不行)。这说明了`top-down pathway`的重要性。
- 观察(e)：(e)表示有自顶向下的过程，但是没有横向连接，即向下过程没有融合原来的特征。这样效果也不好的原因在于目标的location特征在经过多次降采样和上采样过程后变得更加不准确。
- 观察(f)，只使用最终的特征图$P_2$，不使用特征金字塔，效果变差；说明特征金字塔很重要。

<img src="/images/FPN/4.png"  width = "800" height = "400"/>
如Table2所示，为RPN作用于Fast R-CNN的效果：
- proposal是固定的（采用Table1（c）的做法）。与Table1的比较类似
- （a）（b）（c）的对比证明在基于区域的目标卷积问题中，特征金字塔比单尺度特征更有效。
-（c）（f）的差距很小，作者认为原因是ROI pooling对于region的尺度并不敏感。因此并不能一概认为（f）这种特征融合的方式不好，要针对具体问题来看待，像上面在RPN网络中，可能（f）这种方式不大好，但是在Fast RCNN中就没那么明显。

# 总结


# 参考文献

- [论文：Feature Pyramid Networks for Object Detection](https://arxiv.org/pdf/1612.03144.pdf)
- [博客：Understanding Feature Pyramid Networks for object detection (FPN)](https://medium.com/@jonathan_hui/understanding-feature-pyramid-networks-for-object-detection-fpn-45b227b9106c)
- [x](https://www.jiqizhixin.com/articles/2017-07-25-2)
- [x](https://blog.csdn.net/u014380165/article/details/72890275)
 


