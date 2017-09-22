---
layout: post
title: TF-Slim指南  (二)
subtitle: 用InceptionV3 对X光片做尘肺病分类实验
date: 2017-07-06
categories: blog
tags: [深度学习]
description: TF-Slim指南之二
---
<span style='color:OrangeRed '>使用TF-Slim中写好的InceptionV3网络对胸部X光片进行分类, 包括了对结果进行了分析以及探索了一下CNN常用的图像预处理方法</span>
## 1.结果分析
承接第一篇文章的结尾, 对我的实验结果进行分析  
### 1.1参数设置的问题吗?
结果不好, 很自然就想到是不是参数的问题. 都知道深度学习参数的设置是门玄学(或者说整个机器学习的参数设置都有这么一丝味道???), 知乎上都戏称炼丹. 深度学习的参数的设置以及训练策略的选择确实是一个很吃经验的地方. 但是细想, 我觉得应该不是参数的问题 , 因为我的finetune是按照官方scrpits目录下训练脚本设定的, batchsize,是最大的不同(batchszie 与内存,显存相关), 可以说它原本脚本参数设定已经是非常合适的, 就算我与它有些许出入, 不至于我的整个模型基本无效.

### 1.2 finetune与scratch的原因吗?  

![](/img/my_article_images/20170706-tfslim-my-project/loss.png){: .center-image}  
上图是本次finetune实验的loss变化图, 可以看见loss几乎改变很小, 是不是意味着模型几乎没有学习到什么内容? 费解.  

我个人的理解是,使用finetune(我们的实验修改了两层, InceptionV3/Logits, InceptionV3/AuxLogits ), 这种方法相当于是在一个拟合好的模型基础上, 然后在一个很小的范围进行边边角角的修改,所以才会出现这种曲线. 如果是完全重新(scratch)训练模型,是不是意味着起始loss会比较高而loss的变化曲线也会更加陡峭?  

为了对比, 我又用尘肺病数据进行scratch方法的训练, 因为实验条件限制, 只训练了10000代(耗时3天, deep的方法就!是!要!吃!硬!件!啊!天! , 对,我是用CPU跑的, 还是要保持围笑...),得到loss曲线如下图,可以看见起始loss很高, 下降很快很平稳, 虽然只训练了10000代但也印证了我对于finetune与scratch的一些的理解.  

![](/img/my_article_images/20170706-tfslim-my-project/scratch_loss.png){: .center-image}  

finetune这个方法是迁移学习的的思路, 即在目标任务数据较少时, 期望借助从源领域学习到的知识, 迁移到目标领域. 因为CNN较低层学习到了图片上物体的一些通用特征, 而较高的层学习到的是各类的特有特征( 我的理解大概是, 像素--->纹理,形状等--->部位--->整体 这么个比方). 像素, 纹理, 形状这些低级的特征是所有物体的都可能有的一些共性(类似与线性代数中的基向量) , 而部位, 整体这些就是各种物体所独有的特征, 所以把在源领域训练好的CNN模型迁移到目标领域的时候, 较低层的特征类似而只需要训练最后几层的参数, 这是finetune的一个基本思路. 但是这种迁移, 目前我所知道到一些情形是多用在自然物体数据上, 比如ImageNet训练出的模型(1000分类)迁移到几十种花的图片分类的问题上. 在自然物体数据上这样做是OK的, 那医学图片与自然图片至少在我们看来差别还是蛮大的, 它们也有这种共性吗?  

 所以, 我首先猜测用scratch训练我们的数据会不会好, 但由于前文所诉, 一是硬件条件不够, 二是这么做没有意义, deep learning就是基于海量数据才会有很好的效果, 2000张的图片直接scratch训练效果我觉的不乐观, 三是直接scratch的话调参的因素可能会比较大, 目前我还没有这么多经验.  
 
 后来查了一些paper , finetune即使在医学图片的数据集上也是有效果的.参考[Deep Convolutional Neural Networks for Computer-Aided Detection: CNN Architectures,Dataset Characteristics and Transfer Learning](https://arxiv.org/abs/1602.03409) 引用一段,
>Our hypothesis on CNN parameter transfer learning is the following: despite the disparity between natural images and natural images, CNNs comprehensively trained on the large scale well-annotated ImageNet may still be transferred to make medical image recognition tasks more effective. Collecting and annotating large numbers of medical images still poses significant challenges. On the other hand, the mainstream deep CNN architectures (e.g., AlexNet and GoogLeNet) contain tens of millions of free parameters to train, and thus require sufficiently large numbers of labeled medical images.  

![](/img/my_article_images/20170706-tfslim-my-project/03.png){: .center-image}  

### 1.3 预处理的原因吗?
接下来我把目光放了分析图像预处理的问题, 事实上数据的预处理绝对是机器学习结合实际问题时要花费大精力去处理的部分. 在这里有必要描述一下任务以及数据集.  

* 医学上如何从X光片鉴别尘肺病简单示例如下图, 只有0,1,2,3 期的类别标签,  其中0期为正常, 个人理解 ,分类依据主要是依据的是聚集的絮状物的多少  

![](/img/my_article_images/20170706-tfslim-my-project/04.png){: .center-image}  

* 我们的原始数据 , 是X光片被直接扫描得到的图片, 一张图片大约4M左右, 细节部分有不够清晰的地方.如图  

![](/img/my_article_images/20170706-tfslim-my-project/05.png){: .center-image}  

最开始的想法, 原图细节有不清晰的地方以及有噪点, 我们运用增强算法加强边缘, 降低噪点, 结果如图.  

![](/img/my_article_images/20170706-tfslim-my-project/06.png){: .center-image}  

然而事实上, 几种经典CNN,  AlexNet, VGG-16, GoogLeNet(Inception), ResNet 入口的图片尺寸都在200\*200左右, 最大是299*299 ≈90000像素 ≈3 \*90kb=270kb. 这样设计是因为他们都是为ImageNet数据集设计的, 自然图片在200~300Kb图片完全能辨识出是什么物体, 而医学图片却不然, 通常是需要医生很仔细的辨认,这正是医院放射科效率低, 深度学习用于医学图像处理提升诊断效率的巨大意义所在.  

我们一张图片大约是4M, 这就意味着使用这些网络的话, X光片在preprocession会被压缩到270k 左右,会丢失大量的信息, 这就意味着我们之前的对于细节的强化处理基本失效. 并且随着我的深入分析, [ 运用CNN的可视化技术(原理及其技术在另一篇文章提到)](), 从图上可看出, 发现使用的InceptionV3 在依据肺部的区域, 以及骨头的边缘, 气管的纹理对X光片进行分类. 这就说明了, 肺部区域, 骨头是我的模型分类的依据. 而聚集的絮状物并没有起到什么决定因素.  

![](/img/my_article_images/20170706-tfslim-my-project/07.png){: .center-image}  

综上, 我们可以得到结论, 直接使用整张X图片(原图或简单压缩)后都是不可取的, 因为 一是使用InceptionV3(或者其他经典的CNN结构)的入口图片尺寸只允许270kb大小的图片, 在这个尺度下X光片以及丢失了很多有用信息. 二是 肺部区域, 骨头是我们这个模型的分类实质上的依据, 而非絮状物, 实质上, 当噪声已经远大于目标信息, 我觉得这个模型肯定是失败的.  

所以, 我有查了很多资料, 主要是**kaggle2016 肺癌检测比赛**, **天池肺癌检测比赛**, 这两个任务医学图像分析实质上是一个detection任务, 数据不仅要有分类的标签, 还必须有病灶的位置标记. 一般做法是将病灶区域采样, 然后训练出关于病灶区域的分类CNN模型(二分类or 多分类), 然后用滑动窗口扫描整张图片, 跳出可能是病变的区域, 综合判断.   

而由于尘肺病的医学上的定义, 它需要结合整个肺部的病灶区域才能给出判断, 所以还不能这样做, 但是整张X光片输入去训练,是走不通的,所以, 深度学习用于图片分析, 有所能, 有所不能.根据任务的性质与数据的特点, 正确去界定,选择合适的预处理与方法会节省很多功夫.  
尽管但是我也想了一些基于我们数据的一些可能预处理方法,





