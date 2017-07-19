---
layout: post
title: Markdown踩坑
subtitle: 写博客遇到的问题,持续更新
date: 2017-07-07
categories: blog
tags: [markdown]
description: 练习使用markdown语法
---
我以为markdown语法就是那么简单, 我错了... 在本地编辑的好好的文章,上传到github之后, 格式就各种不能显示, 心累 ...

查过资料 markdown的文件有好几种解析器( maruku, rdiscount, kramdown,  redcarpet ) github支持的 jekyll 默认用的是kramdown 所以, 有些语法在本地显示的效果非常ok, 一上传显示效果就给给跪 ,也许就是因为这个原因吧, 所以 百度上面搜出的一些markdown语法, 比如图片居中, 字体变色啊 , 不是所有的都适用的... **还是要用Google直接把遇到的问题换成英语描述直接搜索**  

## 一些问题  

#### 不能显示文章标题
~~~
---
layout: post
title: markdown踩坑
subtitle: 
date: 2017-07-06
categories: blog
tags: [markdown]
description: 练习使用markdown语法
---
~~~

一开始文章标题一直无法显示, 我以为是有汉字的原因, 各种去找原因, 结果是, 空格, 引号后面记得空格.   
包括标题#/##/###...等 需要空格, 而有很多格式表示又不需要空格,注意区分  

#### 修改解析器
如前面所说, 图片居中, 代码块无法显示, 公式表示等等 在本地调试时效果都OK ,但是用jekyll 调试的时候,效果就GG
所以尝试修改解析器, 在 _config.yml  
``` 
markdown: redcarpet
highlighter: redcarpet
permalink: pretty
```
本来默认的是kramdown, 被我改成这个了 redcarpet, ( maruku, rdiscount,  kramdown,  redcarpet )有四种选择,然后, jekyll serve 本地调试 ,世界....都清净了...

后记
然而github默认的jekyll只认kramdown,所以只能 继续寻找kramdown下的解决方法  

#### 图片居中
直接改CSS
先在xxx/css/syntax.css 最后添加
~~~
.center-image{margin: 0 auto; display: block;}
~~~
之后对居中图片
~~~
 ![](/my-image){: .center-image}
~~~
  
  
#### 字体颜色修改

从HTML方法修改, 格式如下
~~~
<span style='color:OrangeRed '>xxx</span>
~~~
<span style='color:OrangeRed '>橘红色示例</span>  

#### 公式表达
公式显示一直有问题, 网上资料大多是提到了MathJax对数学公式进行解析, 试了很多, [参考网址](http://www.idryman.org/blog/2012/03/10/writing-math-equations-on-octopress/)    
首先在source/_layouts/default.html 文件添加
~~~HTML
<!-- mathjax config similar to math.stackexchange -->
<script type="text/x-mathjax-config">
MathJax.Hub.Config({
  jax: ["input/TeX", "output/HTML-CSS"],
  tex2jax: {
    inlineMath: [ ['$', '$'] ],
    displayMath: [ ['$$', '$$']],
    processEscapes: true,
    skipTags: ['script', 'noscript', 'style', 'textarea', 'pre', 'code']
  },
  messageStyle: "none",
  "HTML-CSS": { preferredFont: "TeX", availableFonts: ["STIX","TeX"] }
});
</script>
<script src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS_HTML" type="text/javascript"></script>
~~~

然后写公式的时候, 按照latex的一些格式写就好了, 写的时候搜索一下...

---
**不想持续更新的分界线...真是烦啊...**





 