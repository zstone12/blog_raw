---
title: HTTP 报文结构及TCP 报文结构
date: 2020-12-22 16:52:05
tags:
- HTTP
- TCP
categories: 面试
---

同上，是之前没回答上来的问题。

<!--more--> 

先整一整HTTP报文结构，TCP报文结构过一遍大致能想起来。

HTTP报文是真的没啥印象了。

##### HTTP报文结构

**HTTP 协议的请求报文和响应报文的结构基本相同，由三大部分组成**

- 起始行（start line）：描述请求或响应的基本信息；
- 头部字段集合（header）：使用 `key-value` 形式更详细地说明报文；
- 消息正文（entity）：实际传输的数据，它不一定是纯文本，可以是图片、视频等二进制数据

![img](http://blog.poetries.top/img-repo/2019/12/6.png)

---

![img](http://blog.poetries.top/img-repo/2019/12/7.png)

---

##### startline

起始行 GET / 版本号

`GET / HTTP/1.1`

##### header

HOST:

User-AGent:

Accept:接受的报文类型

Accept-Encoding:接受的编码格式？

Accept-language:zh-CNS

空格

数据

---



实在是无趣啊...不搞HTTP报文结构了。

---

![img](https://img-blog.csdn.net/20140609125220296?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE5ODgxMDI5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

捋一下TCP报文段还是有点意思的。

首先最重要的源端口号 目的端口号

然后想一下他为了保证可靠的一些结构

拥塞控制 ->窗口大小

校验和

序列号

确认号 ->ACK=1 SYN=1 ack=m seq=n

SYN ACK FIN  RST 

URG PSH是干啥的呢

　URG（紧急位）：设置为1时，首部中的紧急指针有效；为0时，紧急指针没有意义。

　　PSH（推位）：当设置为1时，要求把数据尽快的交给应用层，不做处理

急数据：URG标志设置为1时，紧急指针才有效，紧急方式是向对方发送紧急数据的一种方式，表示数据要优先处理。他是一个正的偏移量，与TCP收不中序号字段的值相加表示紧急数据后面的字节，即紧急指针是指向紧急数据最后一个字节的下一个字节。这是协议编写上的错误，RFC1122中对此给出了更正说明，紧急指针是数据最后一个字节，不是最后字节的下一位置，TCP首部中只有紧急指针指出紧急数据的位置，他所指的字节为紧急数据，但没有办法指定紧急数据的长度。

　　URG=1，表示紧急指针指向包内数据段的某个字节（数据从第一字节到指针所指向字节就是紧急数据）不进入缓冲区（一般不都是待发送的数据要先进入发送缓存吗？就直接交个上层进程，余下的数据都是要进入接收缓冲的；一般来说TCP是要等到整个缓存都填满了后在向上交付，但是如果PSH=1的话，就不用等到整个缓存都填满，直接交付，但是这里的交付仍然是从缓冲区交付的，URG是不要经过缓冲区的。



>  剩下的不去记了没啥意思

#### 参考：

- https://blog.poetries.top/http-protocol/

- https://www.cnblogs.com/qingjiaowoxiaoxioashou/p/6506157.html