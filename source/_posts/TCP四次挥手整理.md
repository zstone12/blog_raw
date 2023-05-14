---
title: TCP 四次挥手整理
date: 2020-12-22 15:55:37
tags:
categories: 面试
---

昨天面试计算机网络相关的问题都没答上。甚是羞愧。故而整理一下文章。

<!--more--> 

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222164929647.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pzdG9uZTE=,size_16,color_FFFFFF,t_70)

被问到的问题就是TCP四次挥手的状态。想了半天，的确没有这个印象。

只记下来他的过程而已。

---

捋一下它的状态变化。

首先主动和被动关闭方之前都是ESTABLISHED状态。

ESTABLISHED的意思是**建立连接。表示两台机器正在通信**。

然后主动方发送第一个`FIN=1，seq=u`自己进入**FIN-WAIT-1**，被动方收到后进入**CLOSE-WAIT**阶段。

在被动关闭连接情况下，在已经接收到FIN，但是还没有发送自己的FIN的时刻，连接处于CLOSE_WAIT状态。

FIN-WAIT-1就是等待对面回复ACK时候的状态。

(这两个状态有点通过过程推状态的意思了，那么我只要记住这个过程就行了，然后再把名字记下。也就是状态名没有什么特殊的意义...)

FIN-WAIT-2 就是收到对方ACK后进入状态。

然后把被动方数据发完~

被动方发出FIN报文，进入LAST_ACK状态。等待最后一个ACK?

然后主动方收到FIN报文后，进入TIME_WAIT状态。返回ACK报文。

被动方收到后关闭连接。

主动方等待2MSL 后关闭连接。

这用嘴说容易糊。

~~~ 
A(主动关闭)      B(被动关闭)
established   established
fin-wait-1		->FIN=1   |
 |                      |
 |  <-ACK=1				close_wait
 | 											|
 fin-wait-2             |
 |       数据传送        |
 |		<-ACK=1 FIN=1  LASK_ACK
 |											|
 TIME_WAIT ->ACK=1      |
 |                      |
 | 											CLOSED
 | 2msl									|
 CLOSED
~~~

昨天要是能把这个图给他一画不是起飞？

顺势整理下握手阶段的吧。	

---

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222164929645.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pzdG9uZTE=,size_16,color_FFFFFF,t_70)

~~~C
A(主动打开方)       B(被动打开方)
CLOSED                CLOSED
 	| 									 	|
  |											LISTEN
SYN-SENT  SYN=1->        |
  |											 |
  |   <- SYN=1 ACK=1    SYN-RECD
  |											 |
  ESTABLISHED ACK=1->    |
  |										ESTABLISED
  
~~~

为什么有这么一个listen状态...

- LISTEN：侦听来自客户端的TCP端口的**连接请求**



BTW：

顺带推荐下:

[为什么 TCP 建立连接需要三次握手](https://draveness.me/whys-the-design-tcp-three-way-handshake/)

[为什么 TCP 协议有 TIME_WAIT 状态](https://draveness.me/whys-the-design-tcp-time-wait/)



---

参考：

https://www.jianshu.com/p/9968b16b607e