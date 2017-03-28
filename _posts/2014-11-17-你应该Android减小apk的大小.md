---
layout: post
title:  "你应该Android减小apk的大小"
date:   2017-03-28 13:31:01 +0800
categories: jekyll
tag: jekyll
---

* content
{:toc}


First POST build by Jekyll.


你应该Android减小apk的大小
------------------------
本文是对Google官方文档的翻译！！
## 理解APK的结构 ##
在讨论怎样减少应用大小之前，先了解APK的结构是有用的。一个APK文件就是ZIP包，其中包含了组成你的应用的所有文件，比如Java类文件，资源文件，和一个包含被编译资源的文件。
一个APK包含了一下目录：
- META-INF/: 包含CERT.SF和CERT.RSA签名文件，也包含了MANIFEST.MF文件。（译注：校验这个APK是否被人改动过）
- assets/: 包含了应用的资源，这些资源能够通过AssetManager对象获得。
- res/: 包含了没被被编译到resources.arsc的资源。
- lib/: 包含了针对处理器层面的被编译的代码。这个目录针对每个平台类型都有一个子目录，比如armeabi, armeabi-v7a, arm64-v8a, x86, x86_64和mips。

一个APK也包含了一下文件，其中只有AndroidManifest.XML是强制的：
- resources.arsc: 包含了被编译的资源。该文件包含了res/values目录的所有配置的XML内容。打包工具将XML内容编译成二进制形式并压缩。这些内容包含了语言字符串和styles，还包含了那些内容虽然不直接存储在resources.arsc文件中，但是给定了该内容的路径，比如布局文件和图片。
- classes.dex: 包含了能被Dalvik/Art虚拟机理解的DEX文件格式的类。
- AndroidManifest.xml: 包含了主要的Android配置文件。这个文件列出了应用名称、版本、访问权限、引用的库文件。该文件使用二进制XML格式存储。（译注：该文件还能看到应用的minSdkVersion, targetSdkVersion等信息）
 
    

       

# 使用如下方式减小APK #
## 减少资源个数和尺寸 ##
APK的大小会影响应用加载的速度，使用的内存大小，消耗的电量大小。一个最简单的缩小APK大小的方式是减少资源的个数和大小。特别地，你能移除应用中不再使用的资源，你也能使用可缩放的Drawable对象代替图片文件。这节讨论一些通过减少资源从而减少APK大小的方法。
## 移除不使用的资源 ##
lint是Android Studio中的一个静态代码分析工具，检测在”res/“目录中你的代码没有引用的资源。当lint工具发现了项目中潜在的未使用的资源，它会打印以下类似信息：
    res/layout/preferences.xml: Warning: The resource R.layout.preferences appears to be unused [UnusedResources]
被引用的库中可能会包含没使用的资源。如果你在build.gradle文件中启用shrinkResources，则Gradle能自动移除这些资源。

为了使用shrinkResources，你必须要启用代码混淆。在构建过程中，首先proguard移除了未使用的代码，然后gradle移除未使用的资源。
在Gradle插件0.7或更高版本，你能申明应用支持的配置。Gradle通过传递resConfigs和defaultConfig给构建系统，构建系统会防止不支持的配置出现在APK中，从而减少APK大小。
## 最小化第三方库中资源的使用 ##
 当开发Android应用时，你经常使用第三方库提升应用的可用性和灵活性。比如，你引用Android Support Library提升旧设备的用户体验，或者使用Google Play服务实现文字自动翻译。

如果一个第三方库原本是为服务器或普通电脑设计，会引入许多不需要的对象和方法。为了只引入应用需要的库中的那部分，你可以编辑库文件（如果库的license允许你这么做）。你也能使用另外的针对手机的实现同样功能的库。
注意：代码混淆能清除库中不被使用的代码，但是他不能移除库的大量内部依赖。
## 只支持部分屏幕密度 ##
Android支持很多设备集，其中包含了各种不同的屏幕密度。在Android 4.4及更高版本，框架支持不同的密度：ldpi, mdpi, tvdpi, hdpi, xhdpi, xxhdpi和xxxhdpi。尽管Android支持所有这些屏幕密度，但你不需要为每个密度都配置相应的资源。

如果你知道某种特定屏幕密度已经很少有用户使用了，那么你可以考虑是否需要为这个屏幕密度配置资源。如果你不包含针对特定屏幕密度的资源，那么Android会自动缩放原本针对其他密度的已有资源。

如果你的应用只需要缩放的图片，你甚至可以把图片存放在drawable-nodpi目录，从而节省更多空间。我们推荐每个应用都应该至少包含xxhdpi的图片。
## 减少动画帧数 ##
使用帧动画会大大增加APK的大小。图1显示了目录中构成帧动画的多个PNG文件。每个图片都是动画的一帧。

对于加入动画的每帧，你都增加了APK中图片的个数。图1中，帧动画的帧率是30 FPS。如果帧率降到15 FPS，图片数量将减少一半。
##　使用Drawable对象　##
一些图片不需要静态的图片资源，框架能在运行时动态地绘制图像。Drawable对象（XML的<shape>）只需要占用APK中的一点空间。另外，XML形式的Drawable对象能够产生遵循Material Design设计规范的图像。

## 重用资源 ##
你能包含一张图片的很多变种，比如染色、阴影、旋转的版本。但是，我们推荐在运行时复用一张图片来定制化他们。

Android提供了很多方式改变资源的颜色。对于Android 5.0及以上，使用android:tint和tintMode属性。对于更低版本，使用ColorFilter类。

你也能够删除那些只是对另一个资源做旋转的资源。下面的代码片段提供了对一个箭头旋转180度。
    <?xml version="1.0" encoding="utf-8"?>
<rotate xmlns:android="http://schemas.android.com/apk/res/android"
    android:drawable="@drawable/ic_arrow_expand"
    android:fromDegrees="180"
    android:pivotX="50%"
    android:pivotY="50%"
    android:toDegrees="180" />
## 通过代码绘制 ##
  你也能通过代码绘制图像，从而减少APK大小。代码方式绘制图像不需要任何空间因为你不再需要在APK中存储图像文件。
##  压缩PNG文件 ##
 AAPT工具能够在构建过程中通过无损压缩优化res/drawable/中的图片资源。比如aapt工具能将需要颜色少于256色的PNG变为8位PNG图，这样能够在保证图片质量的同时减少内存使用。

需要注意aapt有以下局限性：
- aapt工具不会压缩asset目录的PNG文件。
- 通过aapt的优化，图片文件会使用少于256色。
- aapt工具可能会影响已经被压缩过的PNG文件。为了防止这种情况，你可以在gradle文件中设置cruncherEnabled为false禁用aapt对PNG的压缩。
    aaptOptions {
    cruncherEnabled = false
}
译注：建议把cruncherEnabled设为false，然后通过tinypng手工压缩PNG图片。
## 压缩PNG和JPEG文件 ##
你能使用一些工具（比如pngcrush, pngquant, zopflipng）在不降低图像质量的前提下减少PNG文件大小。所有这些工具都能保留图像质量的情况下减少PNG文件大小。

pngcrush工具特别有效：这个工具通过迭代png过滤器和zlib参数，使用每种过滤器和参数的组合压缩图像，并选择最小的那个作为最后的输出。

对于JPEG文件，能使用packJPG压缩JPEG文件。
译注：guetzli是Google最近推出的JPEG编码器，官方宣称相同图片质量时，比libjpeg生成的图片小20-30%。
## 使用WebP文件格式 ##
你也能使用WebP文件格式存储图片而不是PNG或者JPEG。WebP格式是有损压缩（像JPEG）且有透明通道（像PNG），且压缩率高于JPEG或PNG。

使用WebP文件格式也有一些缺点。第一，低于Android 3.2的版本不支持WebP，第二，WebP的解码时间比PNG长。

## 使用向量图 ##
 你能使用向量图去创建一个分辨率无关的图标。使用向量图能够显著减少APK大小。在Android中向量图是以VectorDrawable对象形式存在的。使用VectorDrawable对象，一个100B的文件能生成一个屏幕大小的清晰图片。

但是，系统需要很长时间渲染VectorDrawable对象，更大的图片需要更长的时间显示在屏幕上。因此只有小图片才考虑使用向量图。
## 减少Native和Java代码 ##
确保理解任何自动生成的代码。比如，许多protocol buffer工具生成了过多的方法和类，这会让你的应用大小翻倍。
## 移除枚举 ##
 一个枚举能让classes.dex文件增加1-1.4K。枚举的加入会快速增加应用体积。我们可以使用@IntDef注解和Proguard代替枚举，它能提供和枚举一样的类型安全转换。

## 减少Native库的大小 ##
 如果你的应用使用了Native代码和Android NDK，你也能通过优化代码减少应用体积，这里介绍的两个技巧是删除调试符号和避免抽取Native库。

## 移除调试符号 ##
 如果应用在开发中并且仍需要调试，那么我们能理解使用调试符号。使用Android NDK提供的arm-eabi-strip工具，能从Native库中删除不必要的调试符号，之后你再编译release包
## 避免抽取Native库 ##
 在APK中存储未压缩的so文件，并且在Manifest文件的<application>中设置android:extractNativeLibs为false，这会防止在安装时PackageManager将APK中的so文件拷贝到文件系统，避免这种拷贝会让应用在做增量更新时的更新包更小。
## 维持多个小的APK包 ##
 你的APK会包含用户下载了但从未使用的内容，比如地区或语言信息（译注：比如我是中国人，我就不会用到其他语种的资源）。为了给用户创建小的下载包，你能把你的应用拆分成多个APK，这些APK的差别在于一些因素（比如屏幕大小或者GPU纹理支持）。

当一个用户下载了应用，设备根据自身的特性和设置获取正确的APK。这种方式能够让设备不获取设备不需要的资源。比如，如果设备是hdpi的，那么他就不需要xxxhdpi的资源。


[jekyll]:      http://jekyllrb.com
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-help]: https://github.com/jekyll/jekyll-help
