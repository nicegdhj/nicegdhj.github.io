---
layout: post
title: tf-slim使用指南: 快速使用神经网络进行图片识别
date: 2017-06-28
categories: blog
tags: [Tensorflow]
description: 使用Tensorflow中tf-slim库快速搭建神经网络开展图像识别任务
---

> 使用Tensorflow中的TF-Slim库快速搭建神经网络开展自己的图像识别任务

## 1.TF-Slim是什么
卷积神经网络CNN在图像识别上的威力大家耳熟能详, Tensorflow, Caffe, Keras等深度学习框架都提供了一些经典的网络的使用方法.我之前使用过Caffe, 安装,可读性,易用性个人觉得是比不上Tensorflow, 包括能借鉴的文档资料也少得多, 并且我C++已经忘光了...刚好组里有一批数据需求使用Tensorflow的分布式方法开展图像识别任务. 于是我就着手从tf-slim开始搞事情!

TF-Slim 是谷歌基于Tensorflow编写的一个轻量级封装库. 提供的API, 	一个好处是,CNN里面包含了许多类似的结构或操作(如多个重复的卷积层和多次卷积操作),使用一些TF-Slim的API可以大大简化这些代码的编写. 另一个好处是,TF-Slim里面已经包含了CNN的经典网络结构的实现,阅读代码能够看见高水准的(毕竟是tensorflow团队自己写的),各个网络基于tensorflow的整个流程实现细节,包括了预处理,训练,验证等等.这比看论文就清晰多了.
 
 使用过程中主要还是参见[官方文档1](https://github.com/tensorflow/models/tree/master/slim#Data)和[官方文档2](https://github.com/tensorflow/tensorflow/tree/master/tensorflow/contrib/slim), 包含了:
 
 * Conv2d(卷积), Conv2dTranspose (反卷积Deconv)等CNN常见的操作的TF-Slim实现方法
 * 运行demo,数据准备(转化为TFrecods格式,后面会详细提到)方法
 * 下载checkpoints方法
 * 重新训练, train scratch方法
 * 加载checkpoints, finetuning方法
 * 验证已训练模型方法
 
基本上包括了一次图片识别任务所有步骤, 然而官方所提供的步骤只能够用于它指定的公共数据集的图片识别任务(如cifar-10, imageNet),我需要利用这个这个框架实现在个人图片数据集上的识别任务.接下来让我们进入代码进行修改.

 ** ps:** TF-slim在 [github上]((https://github.com/tensorflow/models/tree/master/slim#Data)的显示在model/slim目录中, 而model目录中包含了很多谷歌用Tensorflow实现的一些领域深度学习的经典demo, 最近好像又开源了图像detection的R-CNN.
 
 ## 2.操作
 我的需求是使用InceptionV3结构进行胸部X光片的尘肺病的分类(0,1,2,3期).
 ### 2.1 目录详情
 [在这里](https://github.com/tensorflow/models/tree/master/slim#Data)是TF-Slim(**以下简称slim**)在github上的主页面. clone整个上级目录model, 选取其中的slim/文件夹到自己的目录即可. 因为slim在不断的更新, 参考教程还是要依据官方的ReadMe为准, 在这里我主要是依据我当时使用的版本进行记录(Tensorflow 1.10, 大概2017年4月左右发布的版本),下图我是使用过一段时间后的目录中的文件,与原始文件有些区别的是我自己添加了一些额外功能的实现,不影响它的主要功能

![](img/my_article_images/20170701-tensorflow-use-tf-slim/01.png)
- checkponts 存放finetune方法所需要checkponits,在slim的主页**pre-trained models**部分提供了谷歌在Imagenet上训练好的模型
- 

 
 
 





