---
layout: post
title: TF-Slim指南  (二)
subtitle: 用InceptionV3 对X光片做尘肺病分类实验
date: 2017-07-06
categories: blog
tags: [深度学习]
description: TF-Slim指南之二
---
![](/img/my_article_images/20170706-tfslim-my-project/loss.png){: .center-image}  
可以看见,此次实验的loss 几乎改变很小, 是不是意味着模型几乎没有学习到什么内容? 费解.我个人的理解是,使用finetune(我们的实验修改了两层,InceptionV3/Logits,InceptionV3/AuxLogits ) 相当于是在一个拟合好函数, 一个很小的范围进行边边角角的修改,所以才会出现这种曲线.  
用尘肺病数据完全重新(scratch方法)训练模型, 因为实验条件限制, 只训练了10000代(就耗时3天, deep的方法就!是!要!吃!硬!件!啊!天! ,泪奔...),得到loss曲线如下图  

![](/img/my_article_images/20170706-tfslim-my-project/scratch_loss.png){: .center-image}  

可以看见起始loss很高, 下降也很快很平稳, 虽然只训练了10000代但也印证了一些我上面提到一些的理解. 一度怀疑

