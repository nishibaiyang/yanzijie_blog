---
layout: post
title:  "Flutter试驾之Flutter值得原生开发借鉴的地方!"
date:   2018-06-01 13:31:01 +0800
categories: jekyll
tag: jekyll
---

* content
{:toc}


First POST build by Jekyll.github:https://github.com/nishibaiyang


Flutter试驾之Flutter值得原生开发借鉴的地方
------------------------
首先这个标题是别人问我的问题。

刚听到这个问题顿时就蒙了，这是站在多高的高度才能回答的大佬的问题。其实也是个开放前瞻性的问题。

我的回答是从Dart语言本身的优点出发。后来静下来仔细想想，Dart跟其他语言（如Swift或Kotlin）比起来在语法方面并没有什么优势。晴天霹雳。

###布局热加载
在热加载之后不需要重新导航到之前的页面，这比Android的Instant Run不知道要领先多少年。
###反应式的编程风格（类似MVVM）
当发生事件变更时，在“全局状态”里更新这些值，并让Flutter进行UI重绘。Flutter将自动更新文本标签的值。这也是React等框架具备的。
###一切都是widget，组件化
在Android和iOS上，部件所对应的就是各种View类。

Flutter采用了不同的概念，部件不仅仅是结构化的元素。Flutter的部件架构更多地使用了组合，而不是继承，所以部件架构更加强大和灵活。遵循组合大于集成的原则，Flutter从简单的元部件开始，可以构建出非常复杂的部件。Flutter的Container Widget就是由一系列元部件组成的。
###编译成本地代码
Flutter的应用被编译成本地代码，所以性能方面不存在问题。事实上，我认为它比Java或Swift更适合用来开发游戏应用。使用反应式编程风格开发的UI代码更加清晰，再加上良好的性能，非常适合用来开发游戏。

这也是比reactnative优越的地方，rn的话还是要转化为原生的view，这样每次转化是有性能消耗的，但是Flutter直接编译成机器语言，这样就相当于执行原生代码了。
###跨平台
这个就不用说了。
















[jekyll]:      http://jekyllrb.com
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-help]: https://github.com/jekyll/jekyll-help
