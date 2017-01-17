# DeepMask总结 #
[http://blog.csdn.net/u011718701/article/details/53763352](http://blog.csdn.net/u011718701/article/details/53763352)
[https://adeshpande3.github.io/Analyzing-the-Papers-Behind-Facebook's-Computer-Vision-Approach/](https://adeshpande3.github.io/Analyzing-the-Papers-Behind-Facebook's-Computer-Vision-Approach/)


## 整体概述 ##

最近，FAIR开发了一项用于发现和切割单张图像中每个物体的新技术。
这一技术的主要驱动算法是DeepMask——一个新的图像分割框架，以及SharpMask——一个图像分割refine模型，二者结合使得FAIR的机器视觉系统能够感知并且精确的描绘一张图片中任何物体的轮廓。这一识别管道中的最后一步，研究院使用了一个特殊的卷积网络，称为MultiPathNet，为图片中检测到的物体添加标签。
也就是说Facebook研究院的物体检测系统遵循一个三阶段的过程：
- （1）DeepMask生成初始物体mask
- （2）SharpMask优化这些mask
- （3）MutiPathNet识别每个mask框定的物体。

## DeepMask 思想 ##
DeepMask用来产生region proposal.
整体来讲，给定一个image patch作为输入，DeepMask会输出一个与类别无关的mask和一个相关的score估计这个patch完全包含一个物体的概率。它最大的特点是不依赖于边缘、超像素或者其他任何形式的low-level分割，是首个直接从原始图像数据学习产生分割候选的工作。还有一个与其他分割工作巨大的不同是，DeepMask输出的是segmentation masks而不是bounding box。
## DeepMask 网络结构 ##
### 第一篇文章介绍的DeepMask网络结构 ###
训练阶段，网络的一个输入样本k是一个三元组，包括：
- （1）image patch xk；
- （2）与image patch对应的二值mask，标定每个像素是不是物体；
- （3）与image patch对应的标签yk，yk=1需要满足两个条件：一是这个patch包含一个大致在中间位置的物体，二是物体是全部包含在这个patch里的，并且在一定的尺度范围内。否则yk=-1，包括patch只包含一个物体的一部分的情况。注意只有yk=1的时候，mask才用，yk=-1时不使用mask。


![](http://i.imgur.com/6EAEbgF.png)

作为迁移学习，网络的主干部分（trunk）为VGG，事先在ImageNet预训练好（pre-trained）。
具体两个子网络可以参看
[http://blog.csdn.net/u011718701/article/details/53763352](http://blog.csdn.net/u011718701/article/details/53763352)

### 第二篇SharpMask提出了改进的DeepMask ###
这个模型称作DeepMask-ours。SharkMask的第一步做的就是DeepMask，不过再次优化了模型。(github DeepMask的代码就是DeepMask-ours)
#### 两点改进 ####

- Trunk Architecture 具体说来把DeepMask的主干部分由VGG-A（or VGG-D）换为了ResNet-50,并将最后的feature dimension由1024改为了128（出于速度的考虑，并且对于效果并没有多大影响）。
![](http://i.imgur.com/oo9vKuE.png)

- Head Architecture (同样出于效率的考虑)
  DeepMask的两个分支（mask and score prediction）都需要一个大的卷积，并且在inference阶段，因为score branch 需要额外的一个pooling step，因此需要有额外的处理来match segmentation branch.因此对其进行了优化。

      ![](http://i.imgur.com/waL7guI.png)
      （a）为原始DeepMask，最后改进后的模型是(d)head C.
## DeepMask Code ##

![](http://i.imgur.com/lzrxZdU.png)
- DataSample.lua 采样图像的预处理，数据集为COCO，包括一些crop，resize，jitter操作。
运行样例
####  masksample  ####
![](http://i.imgur.com/dss7nrq.jpg)（数据集原始图像 500*375）
![](http://i.imgur.com/mIBBecv.jpg)（按照json文件box框裁剪的图像 256*256（224+32），实际上应该用192（160+32））
![](http://i.imgur.com/NAWarGG.jpg)（对应的mask(经过jitter处理之后)56*56）
#### scoresample ####
![](http://i.imgur.com/7y34P5x.jpg)(原始图像)
![](http://i.imgur.com/jCzEYIg.jpg)（正样本 即 y=1）
![](http://i.imgur.com/jaXcVS1.jpg) (负样本 即 y=-1)
- DeepMask.lua -->网络结构定义的文件，包括去掉ResNet后面几层，以及head C的实现。
- InferDeepMask.lua --> 模型训练好之后，将模型应用到整张图（包括多个位置和尺度）
- TrainerDeepMask.lua --> 定义网络训练的文件。
- train.lua --> 训练的主文件。（包括模型的一些配置参数）。
- computeProposals.lua --> 调用InferDeepMask,取top proposals,得到最终结果。
这里取Number of Proposals = 5，即取top 5 scores,masks,结果如下图：
![](http://i.imgur.com/PuVWvex.png)
（第一列为得到的scores,第二列为Scales索引，第3，4列大致为Mask矩阵位置索引）
将Masks画出来即为下图：
![](http://i.imgur.com/PIaUfvN.png)
