---
layout: post
title:  "Git将本地项目上传到Github!"
date:   2019-01-09 13:31:01 +0800
categories: jekyll
tag: jekyll
---

* content
{:toc}


First POST build by Jekyll.
github:https://github.com/nishibaiyang


Git将本地项目上传到Github
------------------------

开始：

### 创建本地git 仓库，并添加部分文件，提交

1. 在本地创建一个版本库（即文件夹），通过git init把它变成Git仓库
2. 把项目复制到这个文件夹里面，再通过git add .把项目添加到仓库；
3. 再通过git commit -m "注释内容"把项目提交到仓库；

以上步骤截图：
![all](https://img-blog.csdn.net/20170621092707724?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvRGF5YnJlYWsxMjA5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

### 通过命令生成本地git仓库公私钥，将公钥添加github上即可(已添加忽略)
1. 在Github上设置好SSH密钥后，新建一个远程仓库，通过git remote add origin https://github.com/guyibang/TEST2.git将本地仓库和远程仓库进行关联；
2. 最后通过git push -u origin master把本地仓库的项目推送到远程仓库（也就是Github）上。

以上步骤截图：
![all](https://img-blog.csdn.net/20170621093322105?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvRGF5YnJlYWsxMjA5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)




[jekyll]:      http://jekyllrb.com
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-help]: https://github.com/jekyll/jekyll-help