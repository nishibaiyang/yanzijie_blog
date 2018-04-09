---
layout: post
title:  "FastAndroidFrame!"
date:   2016-10-17 13:31:01 +0800
categories: jekyll
tag: jekyll
---

* content
{:toc}


First POST build by Jekyll.


FastAndroidFrame
------------------------

#FastAndroidFrame介绍
---
快速开发android app框架
## 重复造轮子干嘛 ##
网上已经有了那么多的框架，为什么搭建一个自己的框架？也许我想说的原因和网上的答案不一样吧，我自己理解就是：急躁害怕的心。不知道各位看官怎么理解，急躁是因为想快速的开发出心中所想。害怕是因为现在的框架实在是太多了，好害怕就这样没有工作。哈哈，夸张了点。开始进入正题。
## 框架的主体 ##
基于MVP架构（对于简单的activity也可以不用mvp），所以本框架选用了MVP+RXJAVA+okhttp3+glide配合其他的（包括自己封装的一些工具）一起使用，帮助大家快速好用的开发一款APP（本人觉得好的必须的都加上去，如果你有更好的请推荐给我）。
## MVP ##
### 我眼中的MVP ###
小弟才疏德浅不敢评论哪种模式的好坏，只是自己用过之后的一些感悟。个人觉得MVP和mvc相比就是让你的自己的思路更清晰，优秀的命名规则加上好的接口设计，少量的注释别人也能轻易看懂。而且MVP还是比较好理解的，所以个人还是比较喜欢的。
#### MVP是如何工作的 ####
![图片来自网络](http://img.blog.csdn.net/20150622212916054)
所以用过的都知道MVP还是比较符合分层和模块化的思想的，这种解耦的方式可以。可以参考google给出的示例。
## RXJAVA ##
RxJava是一个天生做异步的工具，它的有点就是简洁，流式的代码结构配合lambda显得特别的好看。我的理解每一行的流式代码代表一个任务，至上而下看整体的结构一目了然。具体的请移步[How To Use RxJava](https://github.com/ReactiveX/RxJava/wiki/How-To-Use-RxJava)

rxlifecycle2以及rxpermissions2都一并加入了，拿到就能上手用。
## OkGo ##
OkGo - 3.0 震撼来袭，该库是基于 Http 协议，封装了 OkHttp 的网络请求框架，比 Retrofit 更简单易用，支持 RxJava，RxJava2，支持自定义缓存，支持批量断点下载管理和批量上传管理功能
好了其他的都不多说了，具体见代码吧。如果有其他好的，或者本文中不正确的请指出。
## Glide ##
不多说了，只要是非大量图片的app必备。大量图片建议自行更换fresco（主要内存管理更好）
##该有的都有，不该有的不敢乱加##
为什么该有的都有呢？常见工具类，沉浸式样式，xrecyclerview，logger，butterknife等等都有。不该有的是什么？作为BaseFrame保证有的都是必须的，不是必须的那就按自己的需求或项目需求添加（也是很方便的嘛）
## 该如何使用呢 ##
参考demo

后续持续更新


[jekyll]:      http://jekyllrb.com
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-help]: https://github.com/jekyll/jekyll-help
