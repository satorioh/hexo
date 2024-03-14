---
title: 目标检测之Faster R-CNN原理
date: 2024-03-13 14:44:32
categories: CV
tags:
  - 目标检测
  - RCNN
permalink: object-detection-faster-rcnn
---
本文综合了网上部分讲解资料，旨在归纳并梳理Faster R-CNN算法相关原理，方便日后查阅复习

## 一、目标检测与Faster R-CNN
目标检测与图像分类同属于计算机视觉（CV）范畴，但前者较后者在难度和复杂度层面又上了一个台阶，因为目标检测不光需要解决“是什么”（分类）的问题，还需要解决“在哪儿”（定位）的问题。
![目标检测](../images/od_example.png)
<!--more-->
### 1.目标检测算法分类
基于深度学习的目标检测算法，主要分为两种：one-stage算法和two-stage算法。one-stage（一阶段）算法是在获取的图像特征上直接做分类+回归（定位），而two-stage（两阶段）算法则多了一步：获取图像特征，生成Region Proposal（候选区），再进行分类+回归（定位）

### 2.Faster R-CNN介绍
Faster R-CNN是由Ross B.Girshick在2016年提出的，做为two-stage算法的经典之作，它在前两作（R-CNN和Fast-RCNN）的基础上，又有了很大改进，最终在VOC2007测试集测试mAP达到73.2%，目标检测速度可达5帧/秒（但依然无法实现视频实时检测，所以后来有了one-stage著名的YOLO系列）。虽然提出的较早，但它至今仍是许多目标检测算法的基础，所以对其原理的学习和掌握有助于更好的了解后续模型和算法。

## 二、Faster R-CNN原理详解
### 1.网络结构
![网络结构](../images/fastrcnn_net.png)
如上图，Faster R-CNN网络主要由五部分组成：
- Input(输入): 对于输入的图像，首先需要缩放至固定大小MxN，然后再将MxN图像送入网络。虽然Faster R-CNN本身并不限制输入图像的大小，但是在实际训练过程中，太大的图像容易撑爆内存
- Backbone(主干网络)：也有叫它Extractor的。主要为Conv layers（卷积层），来提取图像特征（feature maps），用于后续的RPN层和全连接层。此处可以使用ZF/vgg/Resnet/MobileNet等。以vgg16为例，包含了13个conv层+13个relu层+4个pooling层
- RPN(Region Proposal Network)：最大的创新点，解决了前两代算法使用ss（selective search）生成region proposals（候选区）太慢的问题。通过softmax判断anchors（下文会讲）属于positive或者negative，再利用bounding box regression修正anchors获得精确的proposals（第一次修正，后面还有一次），这里输出的proposals又称为RoIs（Region of Interests）
- ROI Pooling：输入的是Backbone给到的feature maps和RPN生成的RoIs，综合这些信息后输出proposal feature maps，送入后续全连接层，用于最终的分类和第二次的bounding box（bbox，检测框）regression来修正检测框
- RoI Head：全连接层，有些书里将这部分称之为检测头，主要是利用proposal feature maps计算bbox的类别，同时再次bounding box regression获得检测框最终的精确位置。

下面分别详细介绍这五部分

### 2.Input
主要操作如下：
- 对图片进行缩放，相应的bounding boxes也要进行同等尺度的缩放（不然对不上）
- 归一化处理

会返还4个值供后续训练使用：
- images ： 3×H×W ，BGR三通道，宽W，高H
- bboxes： 4×K , K个bounding boxes，包含每个bounding box的左上角和右下角的座标，形如（Y_min,X_min, Y_max,X_max）
- labels：K， 对应K个bounding boxes的label（对于VOC取值范围为[0-19]）
- scale: 缩放的倍数, 原图H' ×W'被resize到了HxW（scale=H/H' ）

### 3.Backbone
这里面其实包含了conv，pooling，relu三种层。以vgg16为例，共有13个conv层，13个relu层，4个pooling层，值得注意的是：

- 所有的conv层都是：kernel_size=3，pad=1（即填充一圈0），stride=1。即经过conv层后，输出尺寸不变，依旧为MxN
- 所有的pooling层都是：kernel_size=2，pad=0，stride=2。即经过pooling层后，输出尺寸变为原来的1/2

以vgg16为例，因为有4个pooling层，所以一个MxN大小的矩阵经过Backbone后变为(M/16)x(N/16)，即下采样16倍，这个比列（16）很重要，后续feature map映射回原图时需要通过它来计算

### 4.RPN
RPN其实也是一个神经网络，有自己的loss function，以及相关概念，它实现了对Region Proposal的初步二分类和定位
#### (1)Anchor Box
在RPN中，作者提出了Anchor的概念。Anchor是人为预定义的边框(先验框)，也就是一组预设的边框。在训练时，以真实的边框位置相对于预设边框的偏移来构建训练样本。这就相当于，**预设边框先大致在可能的位置“框“出来目标，然后再在这些预设边框的基础上进行调整**。

在一幅图像中，要检测的目标可能出现在图像的任意位置，并且目标可能是任意的大小和任意形状，为了尽可能的框出目标可能出现的位置，预定义边框通常有上千个甚至更多

Anchor Box的生成是以Backbone最后生成的Feature Map上的点为中心的（可以映射回原图），以vgg16为例，使用vgg对输入的图像下采样了16倍，也就是Feature Map上的一个点对应于输入图像上的一个16×16的正方形区域（感受野）。根据预定义的Anchor，Feature Map上的一点为中心，就可以在原图上生成9种不同形状不同大小的边框，如下图：
![anchor to feature map](../images/anchor_feature_map.png)

作者论文中用到的anchor有三种尺寸(scale)和三种比例(ratio)，如下图所示，三种尺寸分别是小（蓝128）中（红256）大（绿512），三个比例分别是1:1，1:2，2:1。3×3的组合总共有9种anchor。

![anchor](../images/fastrcnn_anchor.png)

例如，一张800x600的原始图片，经过vgg下采样后(生成特征矩阵)16倍大小，大小变为50*38，每个点设置9个anchor，则总数为:
```python
ceil(800/16) * cei1(600/16) * 9 = 50*38*9 = 17100
```
本质上，scale是用来表示目标的大小，ratio是用来表示目标的形状

#### (2)网络结构
![RPN网络](../images/fastrcnn_rpn.png)
结合上面两张图，可以看到RPN网络实际分为2条线，上面一条通过softmax分类anchors获得positive和negative分类，下面一条用于计算对于anchors的bounding box regression偏移量，以获得精确的proposal。

#### (3)流程步骤
##### a.做3x3卷积
对于Backbone输出的feature map，通道数为256（作者使用的ZF为256，如果是vgg则为512），RPN会先做一个3x3的卷积，此时输出通道数不变

##### b.设置anchor box
对于feature map上的每个点，计算出k（默认k=9）个anchor boxes(注意和proposal的差异)。

##### c.对anchor box分类
将这些anchor boxes输入到一个1*1的卷积层（如上图“RPN网络”的上路分支），获得一个特征向量，输出通道数为2k（默认k=9的话，就是2x9=18），至于为什么要乘以2，是因为后面需要用来区分每个anchor box是属于positive(前景，包含目标)，还是negative(背景，不包含目标)。然后做一个softmax（前后的2个reshape是为了便于softmax分类），这里也可以使用sigmoid来实现。

那RPN是如何对这些anchor boxes区分是positive还是negative的呢？论文中规定，符合下面条件之一的即为positive:
- 与任意GT(Ground Truth)区域的IoU大于0.7
- 与GT区域的IoU最大的anchor(也许不到0.7)

而与任意GT的区域的IoU都小于0.3的anchor设为negative，对于既不是positive也不是negative的anchor以及跨越图像边界的anchor就直接舍弃掉。

##### d.获取bounding box coordinates
在RPN网络图的下路，通过1x1的卷积，生成了一个包含4k（默认k=9的话，就是4x9=36）个coordinates（dx, dy, dw, dh, 相对于真实物体框的偏移）的特征向量，并结合上路分类获取的2k个score(如下图)，一起进入到最后的Proposal Layer

  ![RPN中的anchor box](../images/fastrcnn_rpn_anchor.png)
![RPN中的分类与回归](../images/fasterrcnn_rpn_cls_rpn.png)

##### e.Proposal Layer
Proposal Layer负责综合所有coordinates(dx, dy, dw, dh)变换量和positive anchors，计算出精准的proposal，送入后续RoI Pooling Layer

该层有3个输入：
- positive or negative anchors分类结果(score)
- 上述anchor对应的bbox reg的coordinates(dx, dy, dw, dh)变换量
- im_info：对于原始图像PxQ，在Input时做了缩放，reshape到了MxN尺寸，则im_info=[M, N, scale_factor]
- feature_stride：Backbone中下采样的倍数。以vgg16为例，4个pooling层，每经过一个pooling，图像尺寸变为原来的1/2，所以feature_stride=16

该层的流程如下图 (摘自《深度学习之PyTorch物体检测实战》)：
![Proposal Layer流程](../images/fasterrcnn_rpn_prop_layer.png)

首先生成大小固定的全部Anchors，然后将网络中得到的回归偏移作用到Anchor上使Anchor更加贴近于真值，并修剪超出图像尺寸的Proposal，得到最初的建议区域。在这之后，按照分类网络输出的得分对Anchor排序，保留前12000个得分高的Anchors。由于一个物体可能会有多个Anchors重叠对应，因此再应用非极大值抑制（NMS）将重叠的框去掉，最后在剩余的Proposal中再次根据RPN的预测得分选择前2000个，作为最终的Proposal，输出到下一个阶段。

其中，对于anchor box的回归（定位）偏移调整又是如何实现的呢？这里偷个懒，直接贴一篇大神的解释：[Fast R-CNN中的边框回归](https://www.cnblogs.com/wangguchangqing/p/10393934.html)，讲的很详细

那NMS去掉重叠框又是怎么操作的呢？具体过程如下图：
![NMS](../images/nms.png)


### QA
#### 1.RPN具体是如何对anchor做分类和回归的？



参考文章：

[一文读懂Faster RCNN](https://zhuanlan.zhihu.com/p/31426458)

[从编程实现角度学习Faster R-CNN](https://zhuanlan.zhihu.com/p/32404424)

[目标检测中的Anchor详解](https://www.cnblogs.com/wangguchangqing/p/12012508.html)

[Fast R-CNN中的边框回归](https://www.cnblogs.com/wangguchangqing/p/10393934.html)

[精读Faster RCNN](https://zhuanlan.zhihu.com/p/82185598)

[Faster R-CNN理论合集](https://www.bilibili.com/video/BV1af4y1m7iL/)

[原始论文](https://arxiv.org/abs/1506.01497)
