# [2016_SegNet(SegNet: A Deep Convolutional Encoder-Decoder Architecture for Image Segmentation)](https://arxiv.org/pdf/1511.00561.pdf)

### 摘要

- 针对xx提出，平衡效果和速率
- 提出SegNet，SegNet由编码网络(an encoder network)、解码网络(a corresponding decoder network)以及逐像素的分类层(a pixel-wise classification layer)组成。
- 编码网络为VGG16的前13个卷积层，解码网络与编码网络对应，逐步恢复出输入分辨率
- 尤其，解码网络中的上采样方式采用unpooling：
 - 解码网络用在解码网络中对应max-pooling步骤所计算得到的池化索引(pooling indices)来完成非线性的上采样。
 - 该上采样方法不需要学习
 - 上采样后的特征图是稀疏的(sparse)，然后通过可训练的卷积核进行卷积，来得到密集的(dense)特征图
- 对比试验表明：

### 创新点

#### unpooling操作
![屏幕快照 2018-10-28 下午8.45.58](media/15406243214219/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-10-28%20%E4%B8%8B%E5%8D%888.45.58.png)
如Fig3所示，为unpooling操作示意图：

- 在编码网络的最大池化层，存储每一个池化窗口的最大特征的位置，即对于每一个2x2的池化窗口，只需要2bits。
- 在解码网络中，通过最大池化的索引来实现上采样
 - 疑问：原理疑问，记录位置，那值呢；代码疑问，代码没看懂


#### SegNet结构
![屏幕快照 2018-10-28 下午5.38.24](media/15406243214219/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-10-28%20%E4%B8%8B%E5%8D%885.38.24.png)
如Fig2所示，为SegNet结构：

- 编码网络由13个卷积层构成，对应于VGG16中的前13个卷积层(抛弃后续全连接层，显著地减少参数)
 - 对于卷积层，只使用$f=3$，$s=1$，”same”填充,每一卷积层均为：Conv+BN+ReLU。
 - 对于池化层，只使用$f=2$，$s=2$的最大池化层(没有重叠窗口)，使得空间尺寸减半。
- 解码网络和编码网络的每一层一一对应，解码网络也有13层
- 解码网络最终的输出，喂给多类别的softmax分类器，来独立地为每个像素计算所属类别概率。


#### class balancing
xxx

### 模型比较

各模型表现比较

速度大小比较



![屏幕快照 2018-10-28 下午9.03.25](media/15406243214219/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-10-28%20%E4%B8%8B%E5%8D%889.03.25.png)

### 模型代码


### 参考文献

- [论文：SegNet](https://arxiv.org/pdf/1511.00561.pdf)
- [官方代码：caffe-segnet](https://github.com/alexgkendall/caffe-segnet)
- [Keras代码：Semantic-Segmentation](https://github.com/liminn/Semantic-Segmentation)
- [语义分割论文-SegNet](http://hellodfan.com/2017/11/10/%E8%AF%AD%E4%B9%89%E5%88%86%E5%89%B2%E8%AE%BA%E6%96%87-SegNet/)


