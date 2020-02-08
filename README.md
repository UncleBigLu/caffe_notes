## activity 的 5个Task

Task1：未修剪视频分类(Untrimmed Video Classification)。这个有点类似于图像的分类，未修剪的视频中通常含有多个动作，而且视频很长。有许多动作或许都不是我们所关注的。所以这里提出的Task就是希望通过对输入的长视频进行全局分析，然后软分类到多个类别。

Task2：修剪视频识别(Trimmed Action Recognition)。这个在计算机视觉领域已经研究多年，给出一段只包含一个动作的修剪视频，要求给视频分类。

Task3：时序行为提名(Temporal Action Proposal)。这个同样类似于图像目标检测任务中的候选框提取。在一段长视频中通常含有很多动作，这个任务就是从视频中找出可能含有动作的视频段。

Task4：时序行为定位(Temporal Action Localization)。相比于上面的时序行为提名而言，时序行为定位于我们常说的目标检测一致。要求从视频中找到可能存在行为的视频段，并且给视频段分类。

Task5：密集行为描述(Dense-Captioning Events)。之所以称为密集行为描述，主要是因为该任务要求在时序行为定位(检测)的基础上进行视频行为描述。也就是说，该任务需要将一段未修剪的视频进行时序行为定位得到许多包含行为的视频段后，对该视频段进行行为描述。比如：man playing a piano

我们的任务应该是task4

# 行为分类

## Methods 

在深度学习出现之前，表现最好的算法是iDT[公式]，之后的工作基本上都是在iDT方法上进行改进。IDT的思路是利用光流场来获得视频序列中的一些轨迹，再沿着轨迹提取HOF，HOG，MBH，trajectory4中特征，其中HOF基于灰度图计算，另外几个均基于dense optical flow(密集光流计算)。最后利用FV(Fisher Vector)方法对特征进行编码，再基于编码训练结果训练SVM分类器

深度学习出来后，陆续出来多种方式来尝试解决这个问题，包含：Two-Stream、C3D(Convolution 3 Dimension)，还有RNN方向

## 研究难点

不鲁棒，迁移最好的模型来做实验，以适应我们教研室行为检测的场景

行为识别处理的是视频，所以相对于图像分类来说多了一个需要处理的时序维度，需要学习的计算量大了很多

行为识别还有一个痛点是视频段长度不一，而且开放环境下视频中存在多尺度、多目标、摄像机移动等众多的问题。这些问题都是导致行为识别还未能实用化的重要原因。

## 

深度学习结果和iDT的结果做ensemble后总能获得一些提升

iDT算法框架主要包含：密集采样特征点，特征轨迹跟踪和基于轨迹的特征提取三个部分。

### 轨迹与轨迹描述子

### 运动描述子

HOF，HOG和MBH

## TWO STREAM方法

Two-Stream CNN网络顾名思义分为两个部分，一部分处理RGB图像，一部分处理光流图像。最终联合训练，并分类

三个贡献点

首先，论文提出了two-stream结构的CNN网络，由空间(RGB)和时间(光流)两个维度的网络组成

其次，作者提出了利用网络训练多帧密度光流，以此作为输入能在有限训练数据的情况下取得不错的结果。

最后，采用多任务训练的方法将两个行为分类的数据集联合起来，增加训练数据，最终在两个数据集上都取得了更好的效果(作者提到，联合训练也可以去除过拟合的可能)。

### TSN

TSN(Temporal Segments Networks)是在上述基础的two-Stream CNN上改进的网络。目前基于two-stream的方法基本上是由TSN作为骨干网络

解决不能对长时间的视频进行建模的问题

先将视频分成K个部分，然后从每个部分中随机的选出一个短的片段，然后对这个片段应用上述的two-stream方法，最后对于多个片段上提取到的特征做一个融合

## C3D

C3D的方法得到的效果普遍比Two-Stream方法低好几个百分点。但是C3D任然是目前研究的热点，主要原因是该方法比Two-Stream方法快很多，而且基本上都是端到端的训练，网络结构更加简洁

方法思想非常简单，图像是二维，所以使用二维的卷积核。视频是三维信息，那么可以使用三维的卷积核。所以C3D的意思是：用三维的卷积核处理视频

## RNN方法

引入了姿态监督的机制，或许能提高视频分类的效果


# 行为检测

不仅需要定位视频中可能存在行为动作的视频段，还需要将其分类。而定位存在行为动作的视频段是一个更加艰巨的任务

但我们这个项目相对简单，都坐在位置上

先定位目标，然后识别目标。所以目前很多行为检测方法都是借鉴于目标检测，主要思想基本上是Temporal Proposal提取，然后进行分类与回归操作 

利用Faster R-CNN框架思路，利用SSD框架思路，还有基于TAG网等等。还有一类方法是基于C3D做帧分类(Frame Label)，然后预测存在行为的视频段并分类，例如2017年ICCV的CDC网络

### CDC

CDC网络是在C3D网络基础上，借鉴了FCN的思想。在C3D网络的后面增加了时间维度的上采样操作，做到了帧预测(frame level labeling)。以下是文章主要贡献点。

第一次将卷积、反卷积操作应用到行为检测领域，CDC同时在空间下采样，在时间域上上采样。

利用CDC网络结构可以做到端到端的学习。

通过反卷积操作可以做到帧预测(Per-frame action labeling)

### R-C3D网络

R-C3D(Region 3-Dimensional Convolution)网络是基于Faster R-CNN和C3D网络思想。对于任意的输入视频L，先进行Proposal，然后用3D-pooling，最后进行分类和回归操作。文章主要贡献点有以下3个。

可以针对任意长度视频、任意长度行为进行端到端的检测

速度很快(是目前网络的5倍)，通过共享Progposal generation 和Classification网络的C3D参数

作者测试了3个不同的数据集，效果都很好，显示了通用性

