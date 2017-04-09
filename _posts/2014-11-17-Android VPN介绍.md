---
layout: post
title:  "Android VPN介绍"
date:   2017-03-30 13:31:01 +0800
categories: jekyll
tag: jekyll
---

* content
{:toc}


First POST build by Jekyll.


Android VPN介绍
------------------------
之前老板想做一个VPN软件，为什么呢？因为翻墙不方便呀，现在只有花钱的还算可以，谁叫天朝人民tm就是有钱呢。结果我没钱啊，唉废话少说
## VPN介绍 ##
 VPN：Virtual Private Network（虚拟专用网络）的缩写。能够在不使用专用物理连接的情况下，将一个虚拟的网络扩展到全网，因此所有连接到VPN中的设备可如同物理连接到同一私有网络中一样，发送并接收数据。如果个人设备使用VPN接入目标私有网络，这种方式也叫作远程访问VPN；当VPN用来连接两个远程网络的时候，被称为site-to-site VPN。
远程访问VPN可以将特定设备与一个静态IP连接，设备如同远程办公室中的一台电脑；但是对于移动设备来说，更常用的是可变网络连接和动态地址的配置方法。这样的配置通常被称为road warrior配置，而且是Android VPN中最常用的配置。

     vpn为了保证传输数据的保密性，一般会用安全隧道协议来认证客户端和实现数据保密，由于VPN协议需要同时在多个网络层工作，同时为了兼容不同的网络配置，常常需要进行多层封装。
按照不同的协议封装，vpn分类如下：
1、点到点隧道协议PPTP（Point-to-Point Tunneling Protocol），用于将PPP分组通过IP网络封装传输。
2、第二层转发协议L2F（Level 2 Forwarding Protocol），允许高层协议的链路层隧道技术。
3、第二层隧道协议L2TP（Layer 2 Tunneling Protocol），用来整合多协议拨号服务。
4、多协议标记交换MPLS（Multi-Protocol Label Switching），用于快速数据包交换和路由。
5、IP安全协议IPSec（Internet Protocol Security），保护端对端的对等设备之间的安全性。
6、安全套接层SSL（Secure Sockets Layer），基于WEB应用的安全协议。
      

## Android上的vpn ##
Android系统支持配置VPN service。可以在设置->更多->VPN中添加。但系统只支持部分VPN协议。如果用户想实现自己的VPN协议，该怎么办呢？为了支持这种扩展， 从 Android 4.0 开始，增加了一个VpnService公开API ，允许第三方应用建立vpn连接，实现VPN的解决方案，而且此应用不需要root权限。

VpnService是一个Service的子类。一旦start了该service，它会创建一个类似于应用代理的服务。任何应用外出的包，都会先发给该服务，然后该服务再转发到网络上。于是这个VpnService就成为需要使用网络的应用和网络服务器之间的一个中间人。这就提供了一个机会来控制外出流量。

## Android VPNService ##
Android设备上，用VpnService框架建立起来的一条从设备到远端的VPN链接.
大体过程如下：

1）应用程序使用socket，将相应的数据包发送到真实的网络设备上。

2）Android系统通过iptables，使用NAT，将所有的数据包转发到TUN虚拟网络设备上去，端口是tun0；

3）VPN程序通过打开/dev/tun设备，并读取该设备上的数据，可以获得所有转发到TUN虚拟网络设备上的IP包。因为设备上的所有IP包都会被NAT转成原地址是tun0端口发送的，所以也就是说你的VPN程序可以获得进出该设备的几乎所有的数据（也有例外，不是全部，比如回环数据就无法获得）；

4）VPN数据可以做一些处理，然后将处理过后的数据包，通过真实的网络设备发送出去。为了防止发送的数据包再被转到TUN虚拟网络设备上，VPN程序所使用的socket必须先被明确绑定到真实的网络设备上去。
## VpnService和client交互 ##
 那VpnService是如何跟client应用通信的呢？这里利用了linux的TUN/TAP机制。TUN/TAP提供了一种虚拟的软网络接口（相对于物理网卡而言）。开发人员可以打开这个软接口设备，获取一个文件描述符（设备即文件），然后read就是从其中读数据，write就是向其中写数据。不同于物理网卡接口是把数据发送到网上或者接收自网上（其实是写/读到内核里，然后网卡驱动发/收到网上），该虚拟网络接口是读/写到应用空间。当建立其TUN/TAP后，client应用本来要发送到实际网卡的包，都会发送给这个虚拟的tun/tap，此时vpnservice就可以read到client发送的数据；vpnservice可以写数据到tun/tap中，此时client如果recv或read，那就会读取这个虚拟网口，获取vpnservice写的数据。这个我估计是通过修改路由表实现的，使得所有的网楼包都优先走tun/tap虚拟网口。

tun和tap的区别是前者这有IP头和IP负载（即三层和以上），而tap包括数据链路层头（二层和以上）。在Android中，实际是一个tun，读写的数据是IP原始报文。为了验证vpnservice创建了一个虚拟网口，可以下载noroot filewall，然后通过adb shell netcfg。如果已经打开了vpnservice，则名字是tun0的接口，其状态是UP。

关于tun/tap可以参考

http://backreference.org/2010/03/26/tuntap-interface-tutorial/和https://www.kernel.org/doc/Documentation/networking/tuntap.txt。

这是linux的机制，而不仅仅是android的。
## VpnService和网络服务器交互 ##
那VpnService如果把收到的数据发送到网上呢？不同于普通linux允许用户发送原始ip报文，Android安全机制只允许socket发送普通的tcp/udp报文。这就要求我们把通过tun收到的ip报文进行解包，获取其tcp/udp payload（即应用层数据），然后通过send发送到服务器。在接收的时候，需要把服务器过来的recv的数据，添加tcp/udp头和ip头，然后write到tun中。对于tcp具体而言要复杂很多：

     如果受到的是syn包，则要connect
    如果connect成功，则需要返回给tun一个syn ack。这样子三次握手结束（因为不用处理最后一个          ack）。
   传输的时候要记录双向的seq，以生成seq和ack序号。
   收到fin包的时候需要close，并返回fin ack。有时候是服务器发起连接终止，此时需要向客户端发      fin。
    收到rst同样要close连接。
    某种程度上，这里要重现实现一个tcp协议栈，不过要比完整的简单很多。


## 详细介绍几种主流的VPN协议 ##

###　PPTP　###
Point-to-Point Tunneling Protocol（PPTP）使用TCP的信道来建立连接，并使用Generic Routing Encapsulation（GRE）隧道协议来封装Point-to-Point Protocol（PPP）的数据包。支持的认证方法有密码认证协议（Password Authentication Protocol，PAP）、握手问题认证协议（Challenge-Handshake Authentication Protocol，CHAP）和微软扩展MS-CHAP v1/v2，以及EAP-TLS。其中当前仍然被认为安全的只有EAP-TLS。

PPP的载荷可以使用微软点对点加密（MPPE）协议进行加密，其中使用了RC4流加密方法。因为MPPE并不支持任何密文认证，所以无法防范位翻转（bit-flipping）攻击。除此之外，RC4加密近年来也出现过多次问题，大大减弱了MMPE和PPTP的安全性。

### L2TP/IPSec ###
二层隧道协议（Layer 2 Tunneling Protocol，L2TP）类似于PPTP，工作在数据链路层（OSI模型中的第二层）。因为L2TP本身并不提供任何加密或者保密功能（依赖于隧道协议实现这些特性），L2TP VPN一般使用L2TP和IPSec协议套件的组合实现，由IPSec完成认证，进行机密性和完整性的保证。

在L2TP/IPSec的配置中，首先会使用IPSec建立一个安全信道，然后L2TP隧道将会在这个安全信道之上建立。L2TP的包会被封装到IPSec包中，因此保证了安全。IPSec的连接需要建立一个安全关联（Security Association，SA），这是密钥算法和模式、加密密钥和建立安全信道所需的其他参数的组合。

SA使用网络安全关联和密钥管理协议（ISAKMP）建立。ISAKMP不会定义一个特殊的密钥交换方法，而是使用人工指定的预先共享的密钥或者使用网络密钥交换（IKE和IKEv2）协议。IKE使用X.509证书进行对方身份的验证（与SSL类似），并使用Diffie-Hellman密钥交换创建一个共享密文，并使用其生成实际的会话加密密钥。
### IPSec Xauth ###
  IPSec扩展认证（Xauth）对IKE进行了扩展，包含了额外的用户认证交换。这样就允许使用一个已存在的用户数据库或者RADIUS架构来认证远端请求访问的客户端，并且能够集成双因素认证。

Mode-configuration（Modecfg）是另一个IPSec扩展，经常被用于远程访问场景。Modecfg可以让VPN服务端向客户端推送网络配置信息，比如私有IP地址和DNS服务器地址。如果同时使用Xauth和Modecfg，能够生成一个纯IPSec的VPN解决方案，不需要使用任何额外协议进行认证和隧道操作。
###  基于SSL的VPN ###
基于SSL的VPN使用SSL或者TLS（见第6章）建立安全连接和网络数据传输隧道。然而并不存在一个标准来定义基于SSL的VPN，所以为了建立安全信道并封装数据包，在不同实现中会使用不同的策略。

OpenVPN是一个很流行的开源VPN应用，使用SSL进行认证和密钥交换（同样支持预先配置的共享静态密钥），并使用定制的加密协议 对数据包进行加密和认证。OpenVPN使用多路SSL会话进行认证和密钥交换以及加密包的传输，而且仅仅使用单一的UDP（或者TCP）端口。多路协议为SSL在UDP上提供了一个可靠的传输层，但是基于UDP的加密数据隧道并没有一定的可靠性。可靠性通过隧道协议本身来提供，通常是TCP。

OpenVPN比IPSec的优势在于协议的简单并且可以完全在用户层实现。而IPSec需要内核层的支持和多个相互依赖的协议实现。此外，由于OpenVPN使用常见的TCP和UDP协议，而且利用一个单一的端口完成多路隧道，所以更容易穿过防火墙、NAT和代理。

接下来的几节将会探究Android内置的VPN支持和Android提供给想完成额外VPN解决方案的Android应用的API。本书将会展示Android VPN架构的主要组件并且介绍如何保护VPN的凭据。
## 始终在线的VPN ##
Android 4.2和之后的版本中支持始终在线的VPN配置：在与指定的VPN建立连接前会屏蔽应用发起的所有网络连接。这样就防止了应用使用非加密的信道，例如公开Wi-Fi来发送数据。

设置一个始终在线的VPN需要将VPN的网关设置为IP地址，并且明确指定DNS服务器的IP。这种明确的配置是为了保证DNS流量不会被发送给本地配置的DNS服务器，其在始终在线的VPN状态下是被屏蔽的。用户选择的配置会以“其他VPN设置”的类型被加密保存到文件

LOCKDOWN_VPN中；该文件中仅包含了选择的配置的名字—本例中为144965b85a6。如果存在LOCKDOWN_ VPN文件，系统在启动的时候就会自动连接指定的VPN。如果下层网络连接发生了重连或者改变（比如切换了Wi-Fi热点），VPN也将会自动重启。

始终在线的VPN通过加入防火墙规则屏蔽了所有不经过VPN接口的所有数据包，保证了所有的流量都会通过VPN进行发送。系统使用LockdownVpnTracker类（始终在线VPN在Android源码中被称为lockdown VPN）来添加防火墙规则：监视VPN的状态，并向netd守护进程发送命令来动态调整当前的防火墙状态。而netd守护进程使用iptables工具修改内核的包过滤表。比如，当一个始终在线的L2TP/IPSec VPN以11.22.33.44的IP地址连接到了VPN服务器，并且以10.1.1.1的IP地址创建了一个隧道接口tun0。
## 基于应用的VPN ##
Android 4.0中增加了一个VpnService公开API ，允许第三方应用实现VPN解决方案，而且此应用不需要被植入系统镜像或者拥有系统权限。VpnService和相关的Builder类允许应用指定网络参数，比如接口IP地址和路由，随后系统使用这些参数创建并配置一个虚拟网络接口。应用将会接收到一个该网络接口对应的文件描述符，之后就可以通过写入或者读取该描述符进行网络隧道通信。
每次读取都会获取一个出去的IP包，而且每次写入会注入一个进入的IP包。因为对于网络包有效的直接访问能够允许基于应用的VPN完成对网络包的注入和修改，所以这种VPN不能自动启动，而且总是需要用户操作。此外，当VPN成功连接，将会显示一个运行状态的通告。对基于应用的VPN连接的警告框.
## 多用户支持 ##
正如上文提到的，在多用户设备上，只有设备所有者用户才能够控制legacy VPN。然而，在Android 4.2和之后版本中增加了多用户的支持，允许所有次级用户（除了在受限设置，即restricted profile的情况下，所有次级用户必须共享主用户的VPN连接）启动基于应用的VPN。虽然这样允许每个用户自行启动各自的VPN，但是只能同时启动一个基于应用的VPN，所有设备用户的流量都会通过当前激活的VPN，启动这个VPN的用户是谁并不会造成影响。在Android 4.4中引入了对于多用户VPN的完整支持：增加了per-user VPN，允许每个用户使用各自的VPN连接，因此实现了多用户之间的隔离。




[jekyll]:      http://jekyllrb.com
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-help]: https://github.com/jekyll/jekyll-help
