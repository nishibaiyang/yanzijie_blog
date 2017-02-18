---
layout: post
title:  "Fiddler个人心得!"
date:   2017-02-03 13:31:01 +0800
categories: jekyll
tag: jekyll
---

* content
{:toc}


First POST build by Jekyll.


Fiddler个人心得
------------------------

#Fiddler#
Fiddler就不用过多的介绍了，知道的朋友都应该了解这个功能强大的软件。接下来就是记录我平常用的一些操作。
##配合Chrome的使用##
Chrome有很多强大的插件，其中有一款方便代理的插件Proxy SwitchySharp，收索安装后会出现下图所示：
![如图](http://i.imgur.com/yuk4h9n.png)
安装完成以后可以设置自己需要的浏览器代理，如图
![](http://i.imgur.com/A8lowIw.png)
这样你就设置了一个fiddler的代理，接下来就可以使用fiddler代理（浏览器相应的网站可以映射到fiddler设置的ip地址上）
##Fiddler Willow重定向##
Willow是Fiddler下的一个插件。Willow的作用是重定向。
首先安装后配置，进入willow界面后，通过右键->Add Project ->Add Rule可以添加规则。
![](http://i.imgur.com/ylQCSkA.png)
还可以配host
![](http://i.imgur.com/F0LnOnC.png)
左边的操作界面如图：
![](http://i.imgur.com/jEmuDtW.png)
其中黄色的部分就是重定向的链接。
##cgi save##
任意选择一个cgi点击右键，save->select sessions,就可以保存这条请求cgi的请求包。
##移动端抓包##

- 首先修改PC端Fiddler配置，进入Tools菜单的Fiddler Options中的Connections。 
- 将Allow remote computers to connect 选中，重启fiddler后生效。
- 接下来是配置移动端的http代理。
iOS系统直接连接到局域网内，将wifi中下方的http代理信息填写。服务器就是pc所在的ip地址，端口就是刚刚配置的8888。
然后在移动端中访问页面，就可以通过fiddler来抓包了。
###禁用缓存###
在进行调试的过程中，我们希望可以立即显示出效果，所以不希望有缓存，我们可以在fiddler里面设置禁用缓存： 
Rules->Performance->Disable Caching
###伪造数据请求###
可以使用composer构造请求报文进行快速测试，可以指定重新发送某条请求。
![](http://img.blog.csdn.net/20151213191145736)
将左侧的请求拖到composer中，修改请求头后点击execute，就会产生一个新的请求提交，再查看。


[jekyll]:      http://jekyllrb.com
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-help]: https://github.com/jekyll/jekyll-help
