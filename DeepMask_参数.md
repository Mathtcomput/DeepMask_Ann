##  computeProposals.lua的几个参数  ##
- **np** --> number of proposals to save in test **=="5"**
- **si** --> initial scale **=="-2.5"**
- **ss** --> scale step  **=="0.5"**
- **sf** --> final scale  **=="0.5"**

## 构造的例子 ##

![](http://i.imgur.com/I0tGcIA.png)（1000*1000 包括四种大小的圆）
### 分别检测四种大小的圆 ###
![](http://i.imgur.com/3xPMD8x.png)（参数 -np 2 -si -2 -sf -1.5 -ss 0.5）
![](http://i.imgur.com/znLJx2O.png)（参数 -np 8 -si -1 -sf -0.5 -ss 0.1）
![](http://i.imgur.com/2A9KSn9.png)（参数 -np 2 -si -0.5 -sf 0 -ss 0.1）
![](http://i.imgur.com/xcXm05z.png)（参数 -np 9 -si 0.5 -sf 0.8 -ss 0.1）(用的是deepmask 前面用的sharpmask)

## 测试的例子 ##
- ![](http://i.imgur.com/TXzx9kJ.png)（原图）
- ![](http://i.imgur.com/gMuM83Q.png)（默认参数）
- ![](http://i.imgur.com/86sSOIu.png)（**deepmask result** -np 7 -si -1 -sf 0.5 -ss 0.1）
- ![](http://i.imgur.com/ae0F2K6.png)（**sharpmask result** -np 7 -si -1 -sf 0.5 -ss 0.1）
## 结果分析 ##
- 对比结果来看,直观上**DeepMask**比**SharpMask**要好一些,可能的原因是后者对噪声敏感一些（毕竟是精细化版本）也有可能是参数没有调好。
- 细节比较还是**SharpMask**要好（参看左上角的零件边界）。
