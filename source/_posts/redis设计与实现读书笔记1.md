---
title: Redis 设计与实现读书笔记(一)
date: 2020-12-16 16:58:36
tags: redis
categories: 读书笔记
---

先将redis设计与实现这一本读完。了解redis的设计思路，再待有时间将Redis源码读一读。看到小一届的学弟早已就Redis源码读完，甚是羞愧。

<!--more--> 

#### 数据结构相关

##### 简单动态字符串(simple dynamic string, SDS)

​	C字符串只会作为字符串字面量用在一些无须对字符串值进行修改的地方，比如打印日志:

​	当redis需要一个可修改的字符串，都会使用SDS来表示字符串值。

举个例子,如果客户端执行命令:

~~~
SET msg "hello world"
~~~

那么Redis将在数据库中创建一个新的键值对，其中，

- 键是一个字符串对象，底层是一个保存着字符串”msg“的SDS.
- 值也是个字符串对象，底层使用一个保存"hello world"的SDS.

再比如,如果客户端执行命令:

~~~
RPUSH fruits "apple" "banana" "cherry"
~~~

- 键是一个字符串对象，底层是一个保存”fruits“的SDS.
- 值是一个列表对象，列表对象包含了三个字符串对象。



除了用来保存数据库中的字符串值之外，SDS还被用作**缓冲区**(buffer),AOF模块中的AOF缓冲区，以及客户端状态的输入缓冲区，都是由SDS实现的。

---

#### SDS的结构

~~~c
struct sdshdr{
  int len; //已使用长度
  int free; //未使用长度
  char buf[]; //真实存储
}
~~~

SDS遵循C字符串以空字符结尾的惯例，”\0“,好处在于SDS可以直接复用一部分C字符串函数库的函数。

~~~c
print("%s",s->buf); //直接打印出来
~~~

##### SDS与C字符串的区别

###### 1.常数复杂度获取字符串长度

只需要访问len属性即可，至于创建SDS时len是怎么来的，就要探究具体实现了。

###### 2.杜绝缓冲区溢出

举个例子:

~~~c
char *strcat(char *dest,const,char *src);
~~~

因为C字符串不记录自身的长度，所以strcat假定用户在执行这个函数时，已经为dest分配足够的内存，当这假定不成立，就会造成缓冲区溢出。

SDS的空间分配策略完全杜绝了发送缓冲区溢出的可能性。当SDSAPI需要对SDS进行修改时，API会先检查空间是否足够，如果不够，会进行扩充。

3.减少修改字符串时带来的内存重分配次数

两种情况，一种是拼接，也就是扩充，第二种是截断，也就是缩小。

策略也是两种，空间预分配，惰性空间释放。

空间预分配的逻辑是这样:

1.修改后的空间小于1MB,也就是len的值，1：1预分配。用一半剩一半。

2.如果大于1M,那就固定多给1m.

惰性空间释放:

1.当缩短的时候，并不马上内存重分配，而是用free记录起来。

也有对应的API ，有需要的时候回真正释放SDS未使用空间，所以不用当心内存浪费。

###### 4.二进制安全.

主要还是因为'\0' 导致C语言字符串只能保存文本数据，而不能保存二进制数据。SDS使用len属性的值来判断字符串结束，而不是'\0'.

5.兼容部分C语言函数

不赘述

----

#### 链表

列表键的底层实现之一就是链表，元素比较多，或者元素都是长字符串时，redis就会用链表作为列表键的底层实现。

除了链表键之外，发布与订阅、慢查询、监视器等功能也用到了链表，

-----

##### 链表及链表节点的实现

~~~c
typedef struct listNode {
  struct listNode * prev;
  struct listNode * next;
  void * value;
}listNode;
~~~

~~~c
typedef struct list {
  listNode * head;
  listNode * tail;
  
  unsigned long len;
  
 	// 节点值复制函数
  void *(*dup)(void *ptr);
  // 节点值释放函数
  void (*free)(void *ptr);
  
  int (*match)(void *ptr,void *key);
}list;
~~~

特点:

- 双端
- 无环
- 带头尾指针
- 带长度计数器
- 多态。 可以用于保存各种不同类型的值

----

#### 字典

Redis数据库使用字典作为底层实现

字典还是哈希键的底层实现之一，元素多或者元素长，则使用字典。

##### 字典的实现

redis字典所使用的哈希表由dict.h/dictht结构定义:

~~~c
typedef struct dictht {
  // 哈希表数组
  dictEntry **table;
  // 哈希表大小
  unsigned long size;
  // 哈希表大小掩码，用于计算索引值
  unsigned long sizemask;
  // 该哈希表已有节点的数量
  unsigned long used;
}dictht;
~~~

table 属性是一个数组，数组中的每一个元素都是一个指向dict.h/dictEntry结构的指针，**每个dictEntry结构保存着一个键值对**。size属性记录了哈希表的大小。

##### 哈希表节点

~~~c
typedef struct dictEntry{
  //
  void *key;
  union{
    void *val;
    uint64_t u64;
    int64_t s64;
  }
  //指向下个哈希表节点，形成链表
  struct dictEntry *next;
}dictEntry;
~~~

key属性保存着键值对中的键，V属性保存键值对中的值，值可以是一个指针，或者是uint64_t整数，又或者是int64_t 整数。

​	next属性是指向另个一哈希表节点的指针，这个指针可以将多个哈希值相同的键值对连接在一下，以此解决键冲突。

Redis中的字典由dict.h/dict结构表示:

~~~c
typedef struct dict{
  // 类型特定函数
  dictType *type;
  void *privdata;
  
  //哈希表
  dicthx ht[2];
  
  //rehash索引
  // 当rehash不在进行时，值为-1
  int rehashidx;
}dict
~~~

一般情况下，只用ht[0]哈希表，ht[1]哈希表只会在对ht[0]哈希表进行rehash时使用。rehashidx记录了rehash目前的进度。

##### rehash

随着操作的不断执行，为了让哈希表的负载因子维持在一个合理的范围内，

程序需要对哈希表的大小进行相应的拓展或者收缩。

Redis对字典的哈希表进行rehash的步骤如下：

1)为字典的ht[1]哈希表分配空间

- 如果是拓展操作，ht[1]的大小为一个大于等于ht[0].used*2的 2的n次幂(假设used是3,那么大小就应该是8)
- 如果是收缩操作，ht[1]的大小为第一个大于等于ht[0].used的2的n次幂(继续假设是3,那么大小就应该是4)

2)将保存在ht[0]中的所有键值对**rehash**到ht[1]上面；rehash指的是重新计算键的哈希值和索引值，然后将键值对放置到ht[1]哈希表的指定位置上。

3)当ht[0]包含的所有键值对都迁移到ht[1]之后，(ht[0]变为空表),释放ht[0]，

将ht[1]设置为ht[0],并在ht[1]新创建一个空表哈希表，为下一次rehash做准备。

---

当下面条件任意一个满足时，进行拓展:

1.没有执行BGSAVE或者BGREWRITEAOF,且负载因子>=1.

2.在进行，且负载因子>=5

load_factor = ht[0].used/ht[0].size

已保存节点数量/哈希表大小

例如: 大小为4，包含4个键值对的哈希表来说,

load_factor = 4/4=1 

当哈希表的负载因子小于0.1时,程序进行收缩。

##### 4.5 渐进式rehash

迁移动作是分多次、渐进式地完成的。

###### 渐进式rehash的步骤：

1)为ht[1]分配空间，同时持有两个哈希表。

2)rehashidx置为0,表示rehash工作开始。

3)在rehash进行期间，每次对字典增删改查，还会顺带将ht[0]哈希表在**rehashidx索引上的所有键值对**rehash到ht[1].rehash完成后，rehashidx属性值+1.

4)所有都迁移完后，rehashidx置为1

----

#### 跳表

个人觉得是一个很有意思的数据结构

Redis中的跳跃表由redis.h/zskiplistNode 和redis.h/zskiplist 两个结构定义，其中zskiplistNode 结构用于表示跳跃表节点，而zskiplist结构用户保存跳跃表节点的相关信息。

~~~c
zskiplist 
 header
 tail
 level 记录目前跳跃表中，层数最大的那个节点的层数。
 length
~~~

~~~c
zskiplistNode 
  level
  backward 指向前一节点
  score 分值
  obj 成员对象
  forward 前进指针
~~~

是zset的实现方式

---

#### 整数集合

intset是集合键的底层实现之一。当一个集合只包含整数值元素，且元素数量不多，就用整数集合。

~~~
> SADD numbers 1 3 5 7 9
> OBJECT ENCODING numbers
"INTSET"
~~~

~~~c
typedef struct intset{
  // 编码方式
  uint32_t encoding;
  
  // 集合包含的元素数量
  uint32_t length;
  
  // 保存元素的数组
  int8_t contents[];
}intset;
~~~

整数集合的每个元素都是contents数组的一个数组项，各个项在数组中按值的大小有序地排序。

虽然intset结构将contents属性声明为int8_t 类型的数组，但实际上contents数组并不包租任何int8_t类型的值，contents数组的真正类型取决于encoding属性的值:

- encoding = INTSET_ENC_INT16，那么contents就是一个int16_t类型的数组，数组里的每个项都是一个int16_t类型的整数值。

---

##### 升级:

每当一个新元素并现有所有元素的类型都长时，整数集合需要先进行升级，然后才将新元素添加到整数集合里面。

过程如下：

1)根据新元素类型，拓展数组的空间大小，并为新元素分配空间。

2)所有元素都转成新元素的相同类型，且保证有序。

3)将新元素添加到底层数组里面。

##### 升级的好处：

1.提升灵活性。

为了避免类型错误，我们通常不会将两种不同类型的值放在同一个数据结构里面。

2.节约内存。

有需要的时候才升级。

##### 6.4降级 

不支持降级 嘿嘿

---

压缩列表

ziplist是列表键和哈希键的底层实现之一。每个列表项要么是小整数值，要么是长度较短的字符串，redis就会用ziplist来做列表键的底层实现。

~~~c
> RPUSH lst 1 3 5 10086 "hello" "world"
> OBJECT ENCODING lst
"ziplist"
~~~

另外，当一个哈希键只包含少量键值对，且每个键值对的键和值要么是小整数值，要么是长度较短的字符串，就会使用ziplist作为哈希键的底层实现。

~~~c
> HMSET profile “name” “Jack" "age" 28 "job" "Programmer"
> OBJECT ENCODING profile
  "ziplist"
~~~

#### ziplist构成

ziplist是Redis为了节约内存而开发的，是由一系列特殊编码的**连续内存块**组成的顺序型数据结构，一个ziplist可以保持任意多个节点(entry),**每一个节点可以保存一个字节数组或者一个整数值**。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201216164130889.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pzdG9uZTE=,size_16,color_FFFFFF,t_70#pic_center)



![在这里插入图片描述](https://img-blog.csdnimg.cn/20201216164336845.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pzdG9uZTE=,size_16,color_FFFFFF,t_70#pic_center)

---

##### 对象

Redis并没有直接使用这些数据结构来实现键值数据库，而是基于这些数据结构创建了一个对象系统。

通过这五种不同类型的对象，Redis可以根据对象的类型来判断一个对象是否可以执行给定的命令。另一个好处是，可以针对不同的场景，为对象设置多种不同的数据结构实现，从而优化对象使用效率。

Redis采用了引用计数的内存回收机制。还用过引用计数实现了对象共享机制，可以让多个数据库键共享同一个对象来节约内存。

~~~c
typedef struct redisObject{
  unsigned type:4;
  unsigned encoding:4;
  void *ptr;
}robj;
~~~

##### 字符串对象

字符串对象的编码可以是int、raw或者embstr

列表对象的编码可以是ziplist或者linkedlist

哈希对象的编码可以是ziplist或者是hashtable

集合对象的编码可以使intset或者是hashtable

有序集合对象的编码可以使ziplist或者是skiplist