---
layout: post
title: TF-Slim指南(一): 快速上手CNN进行图片分类
date: 2017-07-01
categories: blog
tags: [深度学习]
description: TF-Slim指南之一
---
> 使用Tensorflow中的TF-Slim库快速搭建神经网络开展自己的图像识别任务

## 1.TF-Slim是什么 ##
卷积神经网络CNN在图像识别上的威力大家耳熟能详,Tensorflow, Caffe, Keras等深度学习框架都提供了一些经典的网络的使用方法.我之前使用过Caffe, 安装,可读性,易用性个人觉得是比不上Tensorflow, 包括能借鉴的文档资料也少得多, 并且我C++已经忘光了...刚好组里有一批数据需求使用Tensorflow的分布式方法开展图像识别任务. 于是我就着手从tf-slim开始搞事情!

TF-Slim 是谷歌基于Tensorflow编写的一个轻量级封装库. 提供的API, 	一个好处是,CNN里面包含了许多类似的结构或操作(如多个重复的卷积层和多次卷积操作),使用一些TF-Slim的API可以大大简化这些代码的编写. 另一个好处是,TF-Slim里面已经包含了CNN的经典网络结构的实现,阅读代码能够看见高水准的(毕竟是tensorflow团队自己写的),各个网络基于tensorflow的整个流程实现细节,包括了预处理,训练,验证等等.这比看论文就清晰多了.
 
 **使用过程主要还是要参见**[官方文档1](https://github.com/tensorflow/models/tree/master/slim#Data)和[官方文档2](https://github.com/tensorflow/tensorflow/tree/master/tensorflow/contrib/slim), 包含了:
 
 * Conv2d(卷积), Conv2dTranspose (反卷积Deconv)等CNN常见的操作的TF-Slim实现方法
 * 运行demo,数据准备(转化为TFrecods格式,后面会详细提到)方法
 * 下载checkpoints方法
 * 重新训练, train scratch方法
 * 加载checkpoints, finetuning方法
 * 验证已训练模型方法
 
基本上包括了一次图片识别任务所有步骤, 然而官方所提供的步骤只能够用于它指定数据集的图片识别任务(如cifar-10, imageNet),我需要利用这个这个框架实现在个人图片数据集上的识别任务.接下来让我们进入代码进行修改.

 **ps:** TF-slim在 [github上](https://github.com/tensorflow/models/tree/master/slim#Data)显示在model/slim下, 而上级目录model中包含了很多谷歌用Tensorflow实现的一些领域深度学习的经典demo, 最近好像又开源了图像detection的R-CNN.
## 2.使用 ##
我的需求是使用InceptionV3结构对胸部X光片进行尘肺病分类(0,1,2,3期),我的医学图片只有2500张,使用深度学习显然太少,选择finetune的方法.

### 2.1目录结构 ###
 [在这里](https://github.com/tensorflow/models/tree/master/slim#Data)是TF-Slim在github上的主页面. clone上级目录model, 选取其中的slim/到自己的文件夹即可. 因为TF-Slim(**以下简称slim**)在不断的更新, 参考教程还是要依据官方的ReadMe为准. 在这里我主要是依据我当时使用的版本(Tensorflow 1.10, 大概2017年4月左右发布的版本)进行记录,里面有一些其余的文件是我为了是实现额外的一些功能编写的,不影响原本的主体功能.下图是我的文件结构

 <div align=center>
 ![](/img/my_article_images/20170701-tensorflow-use-tf-slim/01.png)
</div>

 * checkpoints: 官方文档 **pre-trained**部分提供的的checkpoints(tensorflow里面他们称作warm-starting, 就是caffe的finetune),我下载了InceptionV3在ImageNet上的checkpoints存放在这里.
 * datasets: slim 运行示例demo时,下载公共数据集(包括了minist, cifar-10, flower, imagenet)及将它们转化为tfrecord格式的py脚本
 * deployment:不太清楚,似乎与tensorflow分布式部署相关
 * nets: 用slim写的经典CNN网络, 包括alexnet,cifarnet,inception, ResNet等
 * preprocessing: 图片放入CNN训练前的预处理过程, 验证时放入CNN前的预处理过程,[这一块我详细看了一下]()
 * scrpits: 包含了train from finetuning 以及train from scratch的详细实例
 
我主要是参考scrpits目录里的脚本,官方文档以及查看对应的代码(当然包括tensorflow的文档)理解整个流程. 在这里推荐pycharm这个Python开发的IDE, 遇到不明白的代码,鼠标选中之后按F3会直接跳转到源码部分查考注释.为了方便每次训练,仿照scrpits中sh脚本,我把我的训练命令写在了 **myscrpits.txt**里

### 2.2 数据准备 ###


 
 

 
 
 





