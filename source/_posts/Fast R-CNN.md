---
title: Fast R-CNN
mathjax: true
date: 2019-01-17 22:26:50
categories: 
- 目标检测
tags:
---

# 摘要

- Fast R-CNN基于R-CNN做出改进，联合了[R-CNN](http://cvnotes.cn/2019/01/16/R-CNN/)中的模块2、模块3与模块4：
 - Fast R-CNN通过CNN首先得到整张图像的特征图，所有的区域建议/RoI**共享该特征图**，通过RoI pooling层，得到各区域建议对应的固定尺寸的输出特征图； 而R-CNN对图像中的每一个区域建议/ROI，通过CNN独立地提取特征；
 - Fast R-CNN通过多任务loss(a multi-task loss)，同时训练目标分类器(object classifier)和包围边框回归器(bounding-box regressor)；
- 综上，Fast R-CNN通过`RoI pooling layer`和`multi-task loss`，成功联合了R-CNN中的模块2、模块3与模块4；通过共享计算，无论训练阶段或推理阶段，均加速了R-CNN，故称为Fast R-CNN。

<!-- more -->

# 创新点
## Fast R-CNN总体结构

<img src="/images/Fast R-CNN/1.png"  width = "500" height = "100">
如上图所示，为Fast R-CNN的总体结构示意图。

## RoI pooling layer
RoI pooling layer的功能：
- 由于要与全连接层相连，因此需要提供固定尺寸的特征图。
- RoI pooling layer即是在输入图像的特征图上(见Fig1中的`Conv feature map`)，对于输入的任意$H \times W$尺寸的RoI(见Fig1中的红框)，提取出固定$h \times w$尺寸(如$7\times 7$)的输出特征图。

RoI pooling layer的工作流程简述：
- 首先，将RoI的坐标映射到特征图上，映射规则很简单：将RoI的各坐标乘以“特征图与原图宽高尺寸的比值”，即得到了RoI映射到特征图上的坐标。
- 然后，依据想要的输出尺寸(如$7\times 7$)，将RoI映射在特征图上的区域进行分块，划分为$h \times w$个池化区域块
- 最后，各块内取最大值，即得到$h \times w$大小的固定尺寸的输出特征图。
- 注意：RoI pooling layer在`坐标映射`以及`区域分块`两步骤，均在一次量化(quantization)，因此存在精度损失。在Mask R-CNN中通过`RoIAlign`进行了优化，在Mask R-CNN中再具体解释。

RoI pooling layer工作流程的动图示例：
<img src="/images/Fast R-CNN/2.gif"  width = "450" height = "450"/>

## multi-task loss
待补(公式推导及原理解释)

# 总结
- 理解Fast R-CNN的两点贡献：RoI pooling layer和multi-task loss。
- Fast R-CNN仍然存在瓶颈：其区域建议步骤仍然由传统方法获得，该方法不仅非常耗时，而且导致了Fast R-CNN不是端到端的。

# 参考文献
- [论文：Fast R-CNN](http://openaccess.thecvf.com/content_iccv_2015/papers/Girshick_Fast_R-CNN_ICCV_2015_paper.pdf)
- [博客(极好)：Object Detection for Dummies Part 3: R-CNN Family](https://lilianweng.github.io/lil-log/2017/12/31/object-recognition-for-dummies-part-3.html#fast-r-cnn)
- [博客：Region of interest pooling explained](https://deepsense.ai/region-of-interest-pooling-explained/)
- [博客：ROI Pooling层解析](https://blog.csdn.net/lanran2/article/details/60143861)
- [博客：边框回归(Bounding Box Regression)详解](https://blog.csdn.net/zijin0802034/article/details/77685438)
- [博客：Bounding box regression详解](https://blog.csdn.net/u011534057/article/details/51235964)
 


