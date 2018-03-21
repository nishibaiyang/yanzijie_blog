---
layout: post
title:  "Android进程通信Binder!"
date:   2018-03-21 13:31:01 +0800
categories: jekyll
tag: jekyll
---

* content
{:toc}


First POST build by Jekyll.


Android进程通信Binder
------------------------
### IPC是个啥
IPC 即 Inter-Process Communication (进程间通信)。

Android 基于 Linux，而 Linux 出于安全考虑，不同进程间不能之间操作对方的数据，这叫做“进程隔离”。

	在 Linux 系统中，虚拟内存机制为每个进程分配了线性连续的内存空间，操作系统将这种虚拟内存空间映射到物理内存空间，每个进程有自己的虚拟内存空间，进而不能操作其他进程的内存空间，只有操作系统才有权限操作物理内存空间。 进程隔离保证了每个进程的内存安全。
### Android中的进程通信方式
#### 叮叮叮
我们先来思考一个问题，Linux系统本身有许多IPC手段，为什么Android要重新设计一套Binder机制呢？
Android也是基于Linux内核，Linux现有的进程通信手段有以下几种:
* 管道：在创建时分配一个page大小的内存，缓存区大小比较有限；
* 消息队列：信息复制两次，额外的CPU消耗；不合适频繁或信息量大的通信；
* 共享内存：无须复制，共享缓冲区直接付附加到进程虚拟地址空间，速度快；但进程间的同步问题操作系统无法实现，必须各进程利用同步工具解决；
* 套接字：作为更通用的接口，传输效率低，主要用于不通机器或跨网络的通信；
* 信号量：常作为一种锁机制，防止某进程正在访问共享资源时，其他进程也访问该资源。因此，主要作为进程间以及同一进程内不同线程之间的同步手段。
* 信号: 不适用于信息交换，更适用于进程中断控制，比如非法内存访问，杀死某个进程等；
既然有现有的IPC方式，为什么重新设计一套Binder机制呢?
1. 高性能：从数据拷贝次数来看Binder只需要进行一次内存拷贝，而管道、消息队列、Socket都需要两次，共享内存不需要拷贝，Binder的性能仅次于共享内存。
2. 稳定性：上面说到共享内存的性能优于Binder，那为什么不适用共享内存呢，因为共享内存需要处理并发同步问题，控制负责，容易出现死锁和资源竞争，稳定性较差。而Binder基于C/S架构，客户端与服务端彼此独立，稳定性较好。
3. 安全性：我们知道Android为每个应用分配了UID，用来作为鉴别进程的重要标志，Android内部也依赖这个UID进行权限管理，包括6.0以前的固定权限和6.0以后的动态权限，传荣IPC只能由用户在数据包里填入UID/PID，这个标记完全 是在用户空间控制的，没有放在内核空间，因此有被恶意篡改的可能，因此Binder的安全性更高。

关于在android下的进程通信的方式，Android开发艺术探索这本书上已经总结了，相信大家都有看过。
![android_art.jpeg](https://upload-images.jianshu.io/upload_images/6879516-4386af730421439f.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这里再对比总结一下：

* 只有允许不同应用的客户端用 IPC 方式调用远程方法，并且想要在服务中处理多线程时，才有必要使用 AIDL
* 如果需要调用远程方法，但不需要处理并发 IPC，就应该通过实现一个 Binder 创建接口
* 如果您想执行 IPC，但只是传递数据，不涉及方法调用，也不需要高并发，就使用 Messenger 来实现接口
* 如果需要处理一对多的进程间数据共享（主要是数据的 CRUD），就使用 ContentProvider
* 如果要实现一对多的并发实时通信，就使用 Socket

bundle、messenger、contentprovider这些都是基于Binder实现的。
### 理解Binder
我们知道每一个Android应用都是一个独立的Android进程，它们拥有自己独立的虚拟地址空间，应用进程处于用户空间之中，彼此之间相互独立，不能共享。但是内核空间却是可以共享的，Client 进程向Server进程通信，就是利用进程间可以共享的内核地址空间来完成底层的通信的工作的。Client进程与Server端进程往往采用ioctl等方法跟内核空间的驱动进行交互。
![binder.png](https://upload-images.jianshu.io/upload_images/6879516-291ca722920e6c3b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
整个流程也如下所示：
1. Server进程将服务注册到ServiceManager。
2. Client进程向ServiceManager获取服务。
3. Client进程得到的Service信息后，建立与Server进程的通信通道，然后就可以与Server进程进行交互了。

Binder是客户端和服务端进行通信的桥梁，当 bindService的时候，服务端就会返回一个包含了服务端业务调用的Binder对象，通过这个Binder对象，客户端就可以获取服务端提供的服务（包含普通服务和基于AIDL的服务）或者数据。
到这里为止，我们对binder已经有了一个大体的认识。接下去也不会很深入的分析，为什么呢？发育的资源不够呀，还要在野区发育下托后期啊。
至于使用网上很多。



















[jekyll]:      http://jekyllrb.com
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-help]: https://github.com/jekyll/jekyll-help
