---
layout: post
title: TF-Slim指南  (一)
subtitle: 快速使用CNN进行图片分类
date: 2017-07-02
categories: blog
tags: [深度学习]
description: TF-Slim指南之一
---    

<span style='color:OrangeRed '>
使用TensorFlow中的TF-Slim库快速搭建神经网络开展自己的图像识别任务;  
使用 InceptionV3 网络对胸部X光片进行尘肺病分类( 0,1,2,3期 );  
医学图片只有2000张, 使用深度学习显然太少, 还要使用finetune的方法.  
</span>

卷积神经网络CNN在图像识别上的威力大家耳熟能详,Tensorflow, Caffe, Keras等深度学习框架都提供了一些经典的网络的使用方法.我之前使用过Caffe, 安装,可读性,易用性个人觉得是比不上Tensorflow, 包括能借鉴的文档资料也少得多, 并且我C++已经忘光了...刚好组里有一批数据需求使用Tensorflow的分布式方法开展图像识别任务. 于是我就着手从tf-slim开始搞事情!  
本次实验[代码存放地址](https://github.com/nicegdhj/GoDeepLearning/tree/master/my-tf-slim-demo)

## 1.TF-Slim是什么 ##
TF-Slim 是谷歌基于Tensorflow编写的一个轻量级封装库. 提供的API, 	一个好处是,CNN里面包含了许多类似的结构或操作(如多个重复的卷积层和多次卷积操作),使用一些TF-Slim的API可以大大简化这些代码的编写. 另一个好处是,TF-Slim里面已经包含了CNN的经典网络结构的实现,阅读代码能够看见高水准的(毕竟是tensorflow团队自己写的),各个网络基于tensorflow的整个流程实现细节,包括了预处理,训练,验证等等.这比看论文就清晰多了.
 
 **使用过程主要还是要参见**[官方文档1](https://github.com/tensorflow/models/tree/master/slim#Data)和[官方文档2](https://github.com/tensorflow/tensorflow/tree/master/tensorflow/contrib/slim), 包含了:
 
 * Conv2d(卷积), Conv2dTranspose (反卷积Deconv)等CNN常见的操作的TF-Slim实现方法
 * 运行demo,数据准备(转化为TFrecods格式,后面会详细提到)方法
 * 下载checkpoints方法
 * 重新训练, train scratch方法
 * 加载checkpoints, finetuning方法
 * 验证已训练模型方法
 
基本上包括了一次图片识别任务所有步骤, 然而官方所提供的步骤只能够用于它指定数据集的图片识别任务(如cifar-10, imageNet),我需要利用这个这个框架实现在个人图片数据集上的识别任务.接下来让我们进入代码进行修改.

 *ps:* TF-slim在 [github上](https://github.com/tensorflow/models/tree/master/slim#Data)显示在model/slim下, 而上级目录model中包含了很多谷歌Tensorflow实现的一些领域深度学习的经典demo, 最近好像又开源了图像detection的R-CNN.
 
 
## 2.使用 ##
 [在这里](https://github.com/tensorflow/models/tree/master/slim#Data)是TF-Slim在github上的主页面. clone上级目录model, 选取其中的slim/到自己的文件夹即可. 因为TF-Slim(**以下简称slim,  Tensorflow简称TF**)在不断的更新, 参考教程还是要依据官方的ReadMe为准. 在这里我主要是依据我当时使用的版本(Tensorflow 1.10, 大概2017年4月左右发布的版本)进行记录,里面有一些其余的文件是我为了是实现额外的一些功能编写的,不影响原本的主体功能.下图是我的文件结构: 

```
.
├── checkpoints
├── datasets
├── deployment
├── eval_image_classifier.py
├── __init__.py
├── my_guided_bp.py
├── my_imaginnet_prediction.py
├── my_model
├── my_scripts.txt
├── my_show_image.py
├── my_test_input.py
├── nets
├── preprocessing
├── ReadMe.md
├── scripts
├── synset.txt
├── train_image_classifier.py
└── utils.py
```   
主要目录:  

 * checkpoints: 官方文档 **pre-trained**部分提供的的checkpoints(tensorflow里面他们称作warm-starting, 就是caffe的finetune),我下载了InceptionV3在ImageNet上的checkpoints存放在这里.
 * datasets: slim 运行示例demo时,下载公共数据集(包括了minist, cifar-10, flower, imagenet)及将它们转化为tfrecord格式的py脚本
 * deployment:不太清楚,似乎与tensorflow分布式部署相关
 * nets: 用slim写的经典CNN网络, 包括alexnet,cifarnet,inception, ResNet等
 * preprocessing: 图片放入CNN训练前的预处理过程, 验证时放入CNN前的预处理过程,[这一块我看了一下]()
 * scrpits: 包含了train from finetuning 以及train from scratch的详细实例
 
我主要是参考scrpits里的脚本,官方文档,查看的代码(包括tensorflow的文档)理解整个训练流程. 在这里推荐pycharm这个Python开发的IDE, 遇到不明白的代码,鼠标选中之后按F3会直接跳转到源码部分查考注释.为了方便每次训练,仿照scrpits中sh脚本,我把我的训练命令写在了 **myscrpits.txt**里



## 3. 数据准备
slim 将图片转化为tf-record格式,再使用QueueRunner方式输入.  
什么是tf-record格式? TF提供了3种方式输入数据, 以拟合下列式子为例  

$$y = x_{1}+x_{2}+6 $$

* Preloaded data:  在TF中保存常量或不改变的变量,如式子中的6  
* Feeding: 在TF中保存一些变量,程序运行的每一步,让Python代码来供给数据, 用tf.placeholder占位, 如算式中的x1和x2.  
* Reading from files: 以pipeline的方式从文件中读取数据,例如假设这里的输入x1,x2非常大, 是成批的图片(多维张量)的时候.

[参考TF文档](https://www.tensorflow.org/programmers_guide/reading_data#feeding)

对于第三种方式,使用TF训练神经网络, 如果数据集比较小,而且内存足够大,可以选择直接将所有数据读进内存,然后每次取一个batch的数据出来.如果数据较多,则需要每次直接从硬盘中进行读取,后者方式效率比较低,TF为这种情况设计了tfrecord数据格式与quene队列读取模式,加快数据的处理,以下是将图片数据转化为tfrecord的实现方式.
 
### 3.1转化为tfrecord格式
[参考博客](https://kwotsin.github.io/tech/2017/01/29/tfrecords.html)  

首先,将我们的图片整理为下图的形式. 以flowers 的数据集为例, 如所有属于 tulips 类jpg格式的图片都放tulips的文件夹内, 并确flowers目录中没有除开 flower_photos 之外的其他文件夹  

```
flowers\
    flower_photos\
        tulips\
            ....jpg
            ....jpg
            ....jpg
        sunflowers\
            ....jpg
        roses\
            ....jpg
        dandelion\
            ....jpg
        daisy\
            ....jpg
```

下载我的github上的项目后, 需要修改xxx/data_preproces/create_tfrecords目录中的 create_tfrecord.py的一些参数  
 
```python  
flags = tf.app.flags
#State your dataset directory
flags.DEFINE_string('dataset_dir', '/home/data/classifyByLabel_data', 'String: Your dataset directory')

# The number of images in the validation set. You would have to know the total number of examples in advance. This is essentially your evaluation dataset.
flags.DEFINE_float('validation_size', 0, 'Float: The proportion of examples in the dataset to be used for validation')

# The number of shards to split the dataset into
flags.DEFINE_integer('num_shards', 4, 'Int: Number of shards to split the TFRecord files')

# Seed for repeatability.
flags.DEFINE_integer('random_seed', 0, 'Int: Random seed to use for repeatability.')

#Output filename for the naming the TFRecord file
flags.DEFINE_string('tfrecord_filename', '/home/data/my_tf_data/mydata', 'String: The output filename to name your TFRecord file')

FLAGS = flags.FLAGS
```  
* 第一个参数： 图片数据集所在路径
* 第二个参数：验证集数据占整个数据集比例，如 0.2，则整个数据集有 20%的图片会被于制作验证集
* 第三个参数：将 tfrecord 格式生成几部分，关系不大
* 第四个参数：随机种子，用于将整个数据集随机打乱，关系不大
* 第五个参数：tfrecord 文件生成后输出路径，其中最后的 mydata 是生成的 tfrecord 的文件名，请不要更改，因为mydata这个文件名被 我写入了xxx/tf_slim/datasets/ my_dataset.py
之后执行create_tfrecord.py生成tfrecord格式数据, 会同时包含一个label.txt文件,请保留,结果形如:  

![](/img/my_article_images/20170701-tensorflow-use-tf-slim/04.png){: .center-image}

### 3.2 修改my_dataset.py 文件
在第2部分提到过,slim在datasets 目录下只写了4个公开数据集的处理方式(包括了tf-record格式的decode, batchsize批读取等), 所以要处理我们的尘肺病图片, 还需要编写这部分的操作  
我按照了xxx/tf-slim/scrpits/finetune_inception_v3_on_flowers.sh 的脚本,找到了finetune的命令,按照datasets/flower.py 文件修改得到datasets/my_dataset.py ,使用时,需要相应参数  

~~~Python
_FILE_PATTERN = 'mydata_%s_*.tfrecord'

SPLITS_TO_SIZES = {'train_data': 800, 'validation': 200}

_NUM_CLASSES = 4

_ITEMS_TO_DESCRIPTIONS = {
    'image': 'my image ',
    'label': 'between 0 and 3',
}
~~~
* 第一个参数：tfrecord 文件名， 对应于 上一步所提到的第 5 个参数
* 第二个参数：上一步第二个参数将数据划分后，训练集，验证集各自的样本数量
* 第三个参数：分类数量
* 第四个参数：对于数据集的描述，不重要



### 4.模型训练
单机fine-tune，整理后的数据集规模也才2000张图片，对于深度学习模型而言无疑是非常小的数据集，所以我们有必要使用fine-tune迁移学习的思路，以本次使用网络InceptionV3为例，InceptionV3,使用在通用数据集ImageNet上训练好的模型，然后针对最后几层使用我们的数据集单独训练，这样训练出来的模型才会有一个比较好分类效果。
首先，我们需要预先下载训练好的模型, 在<https://github.com/tensorflow/models/tree/master/slim#Data> 列表中选择InceptionV3的checkpoint项进行下载
然后终端在xxx/tf_slim/目录下执行以下训练用的命令，包括验证命令,我都写在了xxx/tf_slim/my_scripts.txt上, 例如  

~~~python
#inceptionV3
TRAIN_DIR=/home/nicehija/PycharmProjects/3classifier_0andOthers/0_1savemodel
DATASET_DIR=/home/nicehija/PycharmProjects/3classifier_0andOthers/tf_data/0_1
DATASET_NAME=my_dataset
PRETRAINED_CHECKPOINT_DIR=/home/nicehija/PycharmProjects/beijingproject_tensorflow/slim/tmp

python train_image_classifier.py \
  --train_dir=${TRAIN_DIR} \
  --dataset_name=${DATASET_NAME} \
  --dataset_split_name=train \
  --dataset_dir=${DATASET_DIR} \
  --model_name=inception_v3 \
  --checkpoint_path=${PRETRAINED_CHECKPOINT_DIR}/inception_v3.ckpt \
  --checkpoint_exclude_scopes=InceptionV3/Logits,InceptionV3/AuxLogits \
  --trainable_scopes=InceptionV3/Logits,InceptionV3/AuxLogits \
  --max_number_of_steps=6000 \
  --batch_size=16 \
  --learning_rate=0.01 \
  --learning_rate_decay_type=fixed \
  --save_interval_secs=100 \
  --save_summaries_secs=100 \
  --log_every_n_steps=100 \
  --optimizer=rmsprop \
  --weight_decay=0.00004
~~~
* PRETRAINED_CHECKPOINT_DIR：上一步checkpoints存放路径
* TRAIN_DIR：训练出来的模型保存位置的路径
* DATASET_DIR：训练数据所在路径
* DATASET_NAME：对应于 制作tfrecord部分的(2)点提及的第5个参数，模型调用xxx/my_tf_slim/datasets/my_dataset.py文件，指定了的detfrecord的方法以及翻转，crop等数据预处理方法  

所有网络参数意义及其使用方法, 详见[官方文档1](https://github.com/tensorflow/models/tree/master/slim#Data)
训练完结之后验证模型模型的步骤, 也写在了xxx/tf_slim/my_scripts.txt上, 主要参数与训练时意义一样
如图,inceptionV3模型训练过程被顺利启动  

![](/img/my_article_images/20170701-tensorflow-use-tf-slim/end.png){: .center-image}   

### 5.结果
使用深度学习进行尘肺病期别判定, 使用InceptionV3 结构, 单机CPU, fine-tune 5000代(loss已经收敛,其实初始loss就不高,2.0左右,收敛也就1.0左右). 经过多次实验, 效果很差,4分类(0, 1, 2, 3 期,样本数量平均)准确率最高只有0.275~0.3 , 相当于模型几乎无效.  

实验效果不好, 我分析的工作写在了第二篇了
