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
 - Fast R-CNN通过CNN首先得到整张图像的特征图，所有的区域建议/ROI**共享该特征图**，通过RoI pooling层，得到各区域建议对应的固定尺寸的输出特征图； 而R-CNN对图像中的每一个区域建议/ROI，通过CNN独立地提取特征；
 - Fast R-CNN通过多任务loss(a multi-task loss)，同时训练目标分类器(object classifier)和包围边框回归器(bounding-box regressor)；
- 综上，Fast R-CNN通过`RoI pooling layer`和`multi-task loss`，成功联合了R-CNN中的模块2、模块3与模块4；通过共享计算，无论训练阶段或推理阶段，均加速了R-CNN，故称为Fast R-CNN。

<!-- more -->

# 创新点
## Fast R-CNN总体结构

<img src="/images/Fast R-CNN/1.png"  width = "600" height = "200"/>
如上图所示，为Fast R-CNN的总体结构示意图：
xx

## RoI pooling layer
RoI pooling layer的目的：
- 对输入的任意尺寸($H \times W$)的RoI，输出固定尺寸的特征图($h \times w$)

RoI pooling layer工作流程简述：
- 将RoI的坐标映射到特征图上；其中，映射规则很简单：将RoI的各坐标乘以“特征图与原图宽高尺寸的比值”，即得到了RoI映射到特征图上的坐标。
- 依据想要的输出尺寸($h \times w$)，将RoI映射在特征图上的区域进行分块，划分为$h \times w$个池化区域块，各块内取最大值，即得到$h \times w$大小的固定尺寸的输出特征图。

RoI pooling layer工作流程示例：
<img src="/images/Fast R-CNN/2.png"  width = "1000" height = "400"/>
- 左1为Fast R-CNN通过CNN首先得到整张输入图像的特征图，左2为RoI映射在特征图上的区域，左3为池化区域分块，左4为固定尺寸的输出特征图。
<img src="/images/Fast R-CNN/3.gif"  width = "450" height = "450"/>
- 上图为RoI pooling layer工作流程的动图示例。

## multi-task loss




# 总结


# 参考文献
- [论文：Fast R-CNN](http://openaccess.thecvf.com/content_iccv_2015/papers/Girshick_Fast_R-CNN_ICCV_2015_paper.pdf)
- [论文：Rich feature hierarchies for accurate object detection and semantic segmentation](https://arxiv.org/pdf/1311.2524v3.pdf)
- [博客(极好)：Object Detection for Dummies Part 3: R-CNN Family](https://lilianweng.github.io/lil-log/2017/12/31/object-recognition-for-dummies-part-3.html#fast-r-cnn)
- [博客：Region of interest pooling explained](https://deepsense.ai/region-of-interest-pooling-explained/)
- [博客：ROI Pooling层解析](https://blog.csdn.net/lanran2/article/details/60143861)
- [博客：边框回归(Bounding Box Regression)详解](https://blog.csdn.net/zijin0802034/article/details/77685438)
- [博客：Bounding box regression详解](https://blog.csdn.net/u011534057/article/details/51235964)
 


