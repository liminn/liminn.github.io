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
 - `bottom-up pathway`：通过ResNet，构成自下而上的路径。
 - `top-down pathway`：通过对各层级特征进行逐步上采样，构成自上而下的路径。
 - `lateral connections`：横向连接将`bottom-up pathway`与`top-down pathway`中对应层级的特征相加，弥补丢失的空间信息，增强定位能力。
- 文中将FPN与RPN以及Faster R-CNN相结合，大幅提升了原有效果。

<!-- more -->

# 创新点
## FPN总体结构

<img src="/images/FPN/2.png"  width = "450" height = "400"/>
如上图所示，为FPN的总体结构示意图。

## Bottom-up pathway

## Bottom-up pathway

## FPN with RPN

## FPN with Fast R-CNN or Faster R-CNN

## Segmentation

# 总结


# 参考文献

- [论文：Feature Pyramid Networks for Object Detection](https://arxiv.org/pdf/1612.03144.pdf)
- [博客：Understanding Feature Pyramid Networks for object detection (FPN)](https://medium.com/@jonathan_hui/understanding-feature-pyramid-networks-for-object-detection-fpn-45b227b9106c)
- [x](https://www.jiqizhixin.com/articles/2017-07-25-2)
- [x](https://blog.csdn.net/u014380165/article/details/72890275)
 


