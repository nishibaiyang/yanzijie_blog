---
layout: post
title:  "Android内存泄漏及LeakCanary原理介绍"
date:   2017-05-24 13:31:01 +0800
categories: jekyll
tag: jekyll
---

* content
{:toc}


First POST build by Jekyll.
github:https://github.com/nishibaiyang


Android内存泄漏及LeakCanary原理介绍
------------------------
## 比较常见的几种Android内存泄漏场景 ##
 
- 资源对象没关闭造成的内存泄漏:
#### 描述 ####
资源对象没关闭造成的内存泄漏
资源性对象比如（Cursor，File文件等）往往都用了一些缓冲，我们在不使用的时候，应该及时关闭它们，以便它们的缓冲及时回收内存。它们的缓冲不仅存在于java虚拟机内，还存在于java虚拟机外。如果我们仅仅是把它的引用设置为null,而不关闭它们，往往会造成内存泄漏。因为有些资源性对象，比如SQLiteCursor（在析构函数finalize(),如果我们没有关闭它，它自己会调close()关闭），如果我们没有关闭它，系统在回收它时也会关闭它，但是这样的效率太低了。因此对于资源性对象在不使用的时候，应该调用它的close()函数，将其关闭掉，然后才置为null.在我们的程序退出时一定要确保我们的资源性对象已经关闭。

- 构造Adapter时，没有使用缓存的convertView:
#### 描述 ####
以构造ListView的BaseAdapter为例，在BaseAdapter中提供了方法：
public View getView(int position, ViewconvertView, ViewGroup parent)
来向ListView提供每一个item所需要的view对象。初始时ListView会从BaseAdapter中根据当前的屏幕布局实例化一定数量的view对象，同时ListView会将这些view对象缓存起来。当向上滚动ListView时，原先位于最上面的list item的view对象会被回收，然后被用来构造新出现的最下面的list item。这个构造过程就是由getView()方法完成的，getView()的第二个形参View convertView就是被缓存起来的list item的view对象(初始化时缓存中没有view对象则convertView是null)。由此可以看出，如果我们不去使用convertView，而是每次都在getView()中重新实例化一个View对象的话，即浪费资源也浪费时间，也会使得内存占用越来越大

- Bitmap对象不在使用时调用recycle()释放内存:
#### 描述 ####
 有时我们会手工的操作Bitmap对象，如果一个Bitmap对象比较占内存，当它不在被使用的时候，可以调用Bitmap.recycle()方法回收此对象的像素所占用的内存
如何解决bitmap带给我们的内存问题
1. 及时销毁，在用完Bitmap时，要及时的recycle掉。
2. 设置一定的采样率，有时候，我们要显示的区域很小，没有必要将整个图片都加载出来，而只需要记载一个缩小过的图片，这时候可以设置一定的采样率，那么就可以大大减小占用的内存
3. 巧妙的运用软引用，有些时候，我们使用Bitmap后没有保留对它的引用，因此就无法调用Recycle函数。这时候巧妙的运用软引用，可以使Bitmap在内存快不足时得到有效的释放。

- 试着使用关于application的context来替代和activity相关的 context：
#### 描述 ####
不要对activity 的context 长期引用( 一个activity 的引用的生存周期应该和activity 的生命周期相同)
· 试着使用关于application的 context 来替代和activity相关的context
· 如果一个acitivity 的非静态内部类的生命周期不受控制，那么避免使用它；使用一个静态的内部类并且对其中的activity 使用一个弱引用。解决这个问题的方法是使用一个静态的内部类，并且对它的外部类有一WeakReference，就像在ViewRoot中内部类W所做的就是这么个例子。
· 垃圾回收器不能处理内存泄漏的保障。
- 注册没取消造成的内存泄漏
所以建议：register放到onCreate，且onDestroy里无需做判断，直接unregister。目的是注册/反注册在相反的生命周期中处理（onCreate->onDestroy、onStart->onStop），同时，严格保证一定能反注册。

- 集合中对象没清理造成的内存泄漏
#### 描述 ####
我们通常把一些对象的引用加入到了集合中，当我们不需要该对象时，并没有把它的引用从集合中清理掉，这样这个集合就会越来越大。如果这个集合是static的话，那情况就更严重了。
- static的不合理应用导致的内存泄漏
    public class ClassName {      
    private static Context mContext;       //省略 
} 
以上的代码是很危险的，如果将Activity赋值到么mContext的话。那么即使该Activity已经onDestroy，但是由于仍有对象保存它的引用，因此该Activity依然不会被释放. 
如何才能有效的避免这种引用的发生呢？ 
    第一，应该尽量避免static成员变量引用资源耗费过多的实例，比如Context。 
    第二、Context尽量使用Application Context，因为Application的Context的生命周期比较长，引用它不会出现内存泄露的问题。 
    第三、使用WeakReference代替强引用。比如可以使用WeakReference<Context> mContextRef; 


- 线程惹的祸
#### 描述 ####
线程也是造成内存泄露的一个重要的源头。线程产生内存泄露的主要原因在于线程生命周期的不可控

 


      

## LeakCanary原理 ##
1. 注意只支持android4.0以上

- ActivityRefWatcher中，注册Activity生命周期监听接口，当Activity onDestroy()被调用时，将当前Activity加入内存泄漏监听队列；
- WeakReference和ReferenceQueue<Object>配合使用，如果弱引用所引用的对象被垃圾回收，Java虚拟机就会把这个弱引用加入到与之关联的引用队列
- 检测方法就很简单了，主动GC，触发WeakReference被GC，同时检测GC前后，ReferenceQueue是否包含被监听对象；如果不包含，则说明该对象没有被GC，一定存在到GC Roots的强引用链，也就是发生了泄漏







[jekyll]:      http://jekyllrb.com
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-help]: https://github.com/jekyll/jekyll-help
