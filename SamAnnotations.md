# 训练数据标注格式 #
## 训练数据 ##
    COCO train数据大小为 13.5 GB
	COCO val数据大小为 6.6 GB
	标注信息为对应的 json 文件，训练之前需要进行格式转换，即 .json --> .t7
## 训练数据格式举例说明（val数据 id 000328） ##
###  json 文件的 "images:"部分  ###
![](http://i.imgur.com/8UaSfKT.png)
	
	json文件标注图像的尺寸，id.
	idx 为转换过程中的映射。 （-- create mapping from cat/img index to anns indices for that cat/img）
###  json文件的"segmentation："部分  ###
	这一部分是我们需要标注的部分，其结构为
![](http://i.imgur.com/KiVAlcp.png)
	
 - category_id --> 标注Object的类别标签
	
 - bbox 	--> 标注的Object所在位置的矩形框，为4维向量，x,y,w,h(即左上角框的位置，框的width,height)
	
 - segmentation--> 标注instances的轮廓（坐标信息）

 - area 		-->segmentation 的面积
	
 - iscrowd 	--> 训练图片是否为群体实例，即Object instances 超过10个 记为 iscrowd : 1,否则记为 iscrowd : 0。当 ann.iscrowd == 1 时，并不标注每一个instance的轮廓（或者说 Mask），此时的segmentation标注与 == 0 情况不同(没看懂0.0)
 - category_idx,idx,image_idx -->格式转换过程生成
 
## 数据input ##
### 数据的正负样本准则 ###
	如果满足以下两个条件称为正样本即 y = 1
 - (1) patch中的Object大致位于patch中心
 - (2) Object完全在patch中，并且尺寸在一定范围
![](http://i.imgur.com/wZqcE9M.png)
 
### 具体数据处理 ###	
	input patch如果被称为一个标准正样本，则Object正位于patch 中心，并且尺寸为128（此时patch为224*224，即 ObjectSize = patchSize*128/224）
	
 - 正样本构造：拿到标准正样本之后，我们来构造正样本，即对inputpatch和ground truth mask 进行一定程度的偏移和尺度的缩放（shift <= 16pixels,scale = 2^.25）,即经过这种处理之后，其 标签 y = 1.
 - 负样本构造：shift >= 32 pixels,scale > 2^(+-)1，经过这种处理过后 y = -1.
