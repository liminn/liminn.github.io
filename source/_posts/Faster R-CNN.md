---
title: Faster R-CNN
mathjax: true
date: 2019-01-18 22:26:50
categories: 
- 目标检测
tags:
---

# 摘要

- 提出RPN(Region Proposal Network)来提供区域建议。其中，RPN与检测网络共享CNN，因此RPN可以几乎无代价的提供区域建议(3~10ms)。
- RPN网络提供区域建议，Fast R-CNN作为检测网络，因此，`Faster R-CNN` = `RPN` + `Fast R-CNN`。

<!-- more -->

# 创新点
## Faster R-CNN总体结构

<img src="/images/Faster R-CNN/1.png"  width = "400" height = "400"/>
如上图所示，为Faster R-CNN的总体结构示意图。

## RPN

工作原理
loss function：

## 训练流程
RPN如何和Fast R-CNN共享计算的
训练流程，推理流程

# R-CNN家族对比


# 总结


# 参考文献
- [论文：Faster R-CNN](http://openaccess.thecvf.com/content_iccv_2015/papers/Girshick_Fast_R-CNN_ICCV_2015_paper.pdf)
- [论文：Rich feature hierarchies for accurate object detection and semantic segmentation](https://arxiv.org/pdf/1311.2524v3.pdf)
- [博客(极好)：Object Detection for Dummies Part 3: R-CNN Family](https://lilianweng.github.io/lil-log/2017/12/31/object-recognition-for-dummies-part-3.html#fast-r-cnn)
- [视频：Paper Review : Faster R-CNN for Real-time Object Detection](https://www.youtube.com/playlist?list=PLkRkKTC6HZMzp28TxR_fJYZ-K8Yu3EQw0)
 


