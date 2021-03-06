---
layout: post
title:  "Flutter模块集成到一个现有的项目!"
date:   2018-10-09 13:31:01 +0800
categories: jekyll
tag: jekyll
---

* content
{:toc}


First POST build by Jekyll.
github:https://github.com/nishibaiyang


Flutter模块集成到一个现有的项目
------------------------

Flutter模块集成到一个现有的Android项目中（前提是已经安装好flutter环境）

1. new flutter module
2. 选择菜单栏的 File -> New Module -> Import Flutter Module

导入成功后，项目中的两个gradle文件会被修改。一个是settings.gradle 和 app/build.gradle , 修改的内容分别如下：
```
setBinding(new Binding([gradle: this]))
evaluate(new File(
  settingsDir.parentFile,
  '../flutter/each_family/.android/include_flutter.groovy'//这是导入的Flutter的东西))
```


app/build.gradle中的依赖会在第一排添加：
```
dependencies {
    implementation project(':flutter')  //这是导入的Flutter的东西
    implementation fileTree(include: ['*.jar'], dir: 'libs')
    implementation 'com.android.support:appcompat-v7:27.+'
    .... //此处省略N多
}
```

*由于 libflutter.so 的问题，或者说是安卓编译架构的问题，在编译运行后打开flutter显示的那个页面可能会出现 couldn't find libflutter.so 的问题，解决办法是只使用 armeabi-v7a, 具体配置是修改 app/build.gradle 文档，如下：*

```
android{
    defaultConfig{
        ndk {
            abiFilters "armeabi-v7a", "x86"
       }
    }
    buildTypes {
          debug {
              ndk {
                abiFilters "armeabi-v7a", "x86"
              }
          }
          release {
              ndk {
                 abiFilters "armeabi-v7a"
              }   
         }
    }
}
```

## 二、native跳转到flutter模块组件

1. 现有项目application必须初始化flutter的实现``` FlutterMain.startInitialization(this); ```
2. 对应activity继承FlutterActivity并注册
``` 
public class JumpFlutterActivity extends FlutterActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        GeneratedPluginRegistrant.registerWith(this);
    }
} 
```
这样就可以实现native跳转flutter模块组件




[jekyll]:      http://jekyllrb.com
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-help]: https://github.com/jekyll/jekyll-help
