---
title: 如何实现 Zset
date: 2020-12-08 21:01:51
tags:
---

搞懂skiplist,再搞一搞ziplist

<!--more--> 

# redis有序集合zset的底层实现——跳跃表skiplist

skiplist

搞懂skiplist 再搞一搞ziplist

---

跳跃表是有序单链表的一种改进，其查询、插入、删除也是O(logN)的时间复杂度。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190705174140317.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RhX2thb19sYQ==,size_16,color_FFFFFF,t_70)

---

问题就在于是如何实现有序的？

---

## skiplist原理

普通有序链表的插入需要一个一个向前查找是否可以插入，所以时间复杂度为O(N)，比如下面这个链表插入23，就需要一直查找到22和26之间。

![img](https://user-gold-cdn.xitu.io/2019/12/28/16f4cdf8de59fcc7?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

如果节点能够跳过一些节点，连接到更靠后的节点就可以优化插入速度：

![img](https://user-gold-cdn.xitu.io/2019/12/28/16f4ce042d683d4e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

在上面这个结构中，插入23的过程是

- 先使用第2层链接head->7->19->26，发现26比23大，就回到19
- 再用第1层连接19->22->26，发现比23大，那么就插入到26之前，22之后

上面这张图就是跳表的初步原理，但一个元素插入链表后，应该拥有几层连接呢？跳表在这块的实现方式是随机的，也就是23这个元素插入后，随机出一个数，比如这个数是3，那么23就会有如下连接：

- 第3层head->23->end
- 第2层19->23->26
- 第1层22->23->26



所以通过有序和和多指针来提高速度。有点那种二分的意思。

---

结一下跳表原理：

- 每个跳表都必须设定一个最大的连接层数MaxLevel
- 第一层连接会连接到表中的每个元素
- 插入一个元素会随机生成一个**连接层数值[1, MaxLevel]之间，根据这个值跳表会给这元素建立N个连接**
- **插入某个元素的时候先从最高层开始，当跳到比目标值大的元素后，回退到上一个元素，用该元素的下一层连接进行遍历，周而复始直到第一层连接，最终在第一层连接中找到合适的位置**



这太骚了

---

redis中skiplist的MaxLevel设定为32层

skiplist原理中提到skiplist一个元素插入后，会随机分配一个层数，而redis的实现，这个随机的规则是：

- 一个元素拥有第1层连接的概率为100%
- 一个元素拥有第2层连接的概率为50%
- 一个元素拥有第3层连接的概率为25%
- 以此类推...

为了提高搜索效率，redis会缓存MaxLevel的值，在每次插入/删除节点后都会去更新这个值，这样每次搜索的时候不需要从32层开始搜索，而是从MaxLevel指定的层数开始搜索

---

### 查找过程

对于zrangebyscore命令：score作为查找的对象，在跳表中跳跃查询，就和上面skiplist的查询一样

---

### 插入过程

zadd [zset name] [score] [value]：

- 在map中查找value是否已存在，如果存在现需要在skiplist中找到对应的元素删除，再在skiplist做插入
- 插入过程也是用score来作为查询位置的依据，和skiplist插入元素方法一样。并需要更新value->score的map

如果score一样怎么办？根据value再排序，按照顺序插入

---

### 删除过程

zrem [zset name] [value]：从**map**中找到value所对应的score，然后再在跳表中搜索这个score,value对应的节点，并删除

---

### 排名是怎么算出来的

zrank [zset name] [value]的实现依赖与一些附加在跳表上的属性：

- 跳表的每个元素的Next指针都记录了这个指针能够跨越多少元素，redis在插入和删除元素的时候，都会更新这个值
- 然后在搜索的过程中按经过的路径将路径中的span值相加得到rank

---

太骚了 redis