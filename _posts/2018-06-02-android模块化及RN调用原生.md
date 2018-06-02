---
layout: post
title:  "Android模块化及RN调用原生!"
date:   2018-06-02 13:31:01 +0800
categories: jekyll
tag: jekyll
---

* content
{:toc}


First POST build by Jekyll.github:https://github.com/nishibaiyang


Android模块化及RN调用原生
------------------------
先老实交代下背景，这篇讲的是两个问题。这两个问题是自己被问到的，但是讲的很含糊，想必问我的人也不满意吧。究其原因：语言组织、表达、紧张。都是泪，下面说说这两个问题吧。

### Q1：android模块化遇到的问题？
按照原始MVP的架构，随着业务越来越复杂、基础组件越来越多。业务与业务之间还有很强的耦合。就变成这样

![复杂业务](https://raw.githubusercontent.com/LiushuiXiaoxia/AndroidModular/master/doc/3.png)

使用模块化之后，变成这样

![模块化](https://raw.githubusercontent.com/LiushuiXiaoxia/AndroidModular/master/doc/4.png)

很明显清晰了不少。那么模块化又存在哪些问题呢？

1. 最常见的R文件问题。
我当时说的时候：整体与个体之间冲突，findbyid常用的ButterKnife不能用，因为会有冲突等。module的application和主app的application冲突。

总之没有很系统的讲出来，估计对方也不耐烦了。尴了个噶。接下来正规系统的讲下：

Android开发中，各个资源文件都是放在res目录中，在编译过程中，会生成R.java文件。R文件中包含有各个资源文件对应的id，这个id是静态常量，但是在Library Module中，这个id不是静态常量，那么就会出现找不到id或者id冲突这样的问题。因此常用的ButterKnite也不能使用

解决方案：
1. 重新一个Gradle插件，生成一个R2.java文件，这个文件中各个id都是静态常量，这样ButterKnite就可以正常使用了，通过R2获取到id。
2. 使用Android系统提供的最原始的方式，直接用findViewById以及setOnClickListener方式。
3. 设置项目支持Databinding，然后使用Binding中的对象，但是会增加不少方法数，同时Databinding也会有编译问题和学习成本。目前项目中没有使用Databinging，这里也没有太多的发言权


2. xml冲突，资源冲突问题

解决办法：

1. 对各个模块的资源名字添加前缀，比如user模块中的登录界面布局为activity_login.xml，那么可以写成这样us_activity_login.xml。
2. application的xml只能查查哪些属性冲突，借助网上资料了。


##总结
模块化架构主要思路就是分而治之，把依赖整理清楚，减少代码冗余和耦合，在把代码抽取到各自的模块后，了解各个模块的通信方式，以及可能发生的问题，规避问题或者解决问题。最后为了开发和调试方便，开发一些周边工具，帮助开发更好的完成任务。


###Q2 ReactNative如何调用Android原生模块
先来说说我的回答：android原生模块实现ReactContext，通过它的回调使用原生模块的方法。现在回想太通俗了。下面正规说下：

1. 创建一个原生模块
首先我们需要创建一个原生模块，这个原生模块是一个继承ReactContextBaseJavaModule的Java类,它可以实现一些JavaScript所调用的原生功能.

```
public class RnTest extends ReactContextBaseJavaModule {
  public RnTest(ReactApplicationContext reactContext) {
    super(reactContext);
  }
  // ReactContextBaseJavaModule要求派生类实现getName方法。这个函数用于返回一个字符串
  // 这个字符串用于在JavaScript端标记这个原生模块
  @Override
  public String getName() {
    return "ToastByAndroid";
  }
  // 获取应用包名
  // 要导出一个方法给JavaScript使用，Java方法需要使用注解@ReactMethod
   @ReactMethod
   public void getPackageName() {
     String name = getReactApplicationContext().getPackageName();
     Toast.makeText(getReactApplicationContext(),name,Toast.LENGTH_LONG).show();
    }
}
```
2. 注册模块
要使JavaScript端调用到原生模块还需注册这个原生模块。需实现一个类实现ReactPackage接口
```
public class ExampleReactPackage implements ReactPackage {
    @Override
    public List<NativeModule> createNativeModules(ReactApplicationContext reactContext) {
      List<NativeModule> modules = new ArrayList<>();
      modules.add(new RnTest(reactContext));
      return modules;
    }

    @Override
    public List<Class<? extends JavaScriptModule>> createJSModules() {
      return Collections.emptyList();
    }

    @Override
    public List<ViewManager> createViewManagers(ReactApplicationContext reactContext) {
      return Collections.emptyList();
    }
}
```
还需在MainApplication.java文件中的getPackages方法中，实例化上面的注册类

```
@Override
  protected List<ReactPackage> getPackages() {
    return Arrays.<ReactPackage>asList(
      new MainReactPackage(),
      // 实例化注册类
      new ExampleReactPackage());
    }
  };
  ```
  
3. js调用android原生

	1. 引入模块
`import { NativeModules } from 'react-native';`
	2. 使用方法
	
```
//  这里的ToastByAndroid即为1.创建一个原生模块中getName()方法返回的字符串
var rnToastAndroid = NativeModules.ToastByAndroid;
rnToastAndroid.getPackageName();
```
	3. 回调函数
提供给js调用的原生android方法的返回类型必须是void，React Native的跨语言访问是异步进行的，所以想要给JavaScript返回一个值的唯一办法是使用回调函数或者发送事件

###android主动向rn发送消息
既然都说道js向原生调用了，那么也说说原生向js调用。

根据上面，可以想到的思路就是：通过reactContext获取到jsModule，然后像eventbus一样发生事件，js方获取事件。
1. 创建一个原生模块，发送消息方法

```
public  static void sendEvent(ReactContext reactContext, String eventName, int status)
    {
        System.out.println("reactContext="+reactContext);

        reactContext
                .getJSModule(DeviceEventManagerModule.RCTDeviceEventEmitter.class)
                .emit(eventName,status);
    }
    ```


2. RN端代码

```
DeviceEventEmitter.addListener(eventName, (reminder) => {
      console.log(reminder):
    });
    ```
3. rn调用android模版

```
const RNBridgeModule = NativeModules.RNBridgeModule;
nativeLanuchApp(message) {
    RNBridgeModule.nativePlayVideo(message);
  }

  <TouchableOpacity onPress={() => {
                            this.nativeLanuchApp("111");
                        }} >
      <Text >
        try
      </Text>
    </TouchableOpacity>
    ```


















[jekyll]:      http://jekyllrb.com
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-help]: https://github.com/jekyll/jekyll-help
