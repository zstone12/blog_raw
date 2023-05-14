---
title: LeetCode-LRU 探究
date: 2020-12-20 19:49:07
tags: LeetCode
---

Leetcode上看到这题，感觉是个很有意思的题目，好好整一整

<!--more--> 

*LRU*(Least recently used,最近最少使用)

一个经典的使用场景：**CPU的高速缓存**

---

### 1.基于哈希表和双向链表实现LRU

整体的设计思路是，可以使用 HashMap 存储 key，这样可以做到 save 和 get key的时间都是 O(1)，而 HashMap 的 Value 指向双向链表实现的 LRU 的 Node 节点。

> 有点不太能理解,

貌似我想的并不是很清楚 头有点大

双向链表的好处在哪呢

冷静一下

先上我这个很挫的代码 。复杂度应该都是O(n)



~~~python
class LRUCache:

# 我这个思路 数据反而实际存在hash表里
# 然后维护一个last_used列表

# 把用到的放在队首
    def __init__(self, capacity: int):
        self.max_capacity=capacity
        self.dict_struct=dict()
        self.last_used=list()

    def get(self, key: int) -> int:
        # 如果已经在LRU中
        if key in self.dict_struct:
            if key in self.last_used:
                if self.last_used[-1]==key:
                    return self.dict_struct[key]
                
                to_pop=-1
                for index,value in enumerate(self.last_used):
                    if value==key:
                        to_pop=index
                        break
                
                #删一个插一个很合理啊
                self.last_used.pop(to_pop)
                self.last_used.append(key)
            #不在缓存队列中
            else:
                if len(self.last_used)==self.max_capacity:
                    self.last_used.pop(0)

                self.last_used.append(key)
            return self.dict_struct[key]
        else:
            return -1
            

    def put(self, key: int, value: int) -> None:
        # 有足够的空间加入
        if key in self.dict_struct:
            self.dict_struct[key]=value
            to_pop=-1
            for index,value in enumerate(self.last_used):
                if value==key:
                    to_pop=index
                    break
                
            self.last_used.pop(to_pop)
            self.last_used.append(key)
            return
        if len(self.last_used)+1<=self.max_capacity:
            pass
        else:
            del self.dict_struct[self.last_used[0]]
            self.last_used.pop(0)

        self.dict_struct[key]=value        
        self.last_used.append(key)
~~~

---

麻烦点就在于:

- 随机访问
- 需要将数据插到头部尾部

犯困 搞不明白啊。

二战LRU

先把设计思路搞清晰，一切就都明朗了。

~~~c
读
​		-------数据不存在: return -1
​		-------数据存在: 将数据移动到头(这边设定越靠近头数据越新) 返回数据


 写 数据是否存在->
              	--存在key:定位，修改值，移动到头
                |
                | 不存在key:
								|--------容量达到上限:那就把尾巴的删了 同时对应hashtable里的数据也得删再往头那里写。
  							|	-------容量未达上限:未达上限就只管往头那里写就完了
      
 
  hashtable doubleLinkedlist
  key: 对应值
  
还有一个麻烦点就在于双向链表的实现
注意一下细节.只有当缓存没满时，size在变化
还有就是注意添加的是cache在添加和删除的时候变化

~~~

~~~python
class DLinkListNode:
    def __init__(self,key=0,value=0):
        self.key=key
        self.value=value
        self.prev=None
        self.next=None


class LRUCache:

    def __init__(self, capacity: int):
        self.cache=dict()
        # 伪头伪尾
        self.tail=DLinkListNode()
        self.head=DLinkListNode()
        self.head.next=self.tail
        self.tail.prev=self.head
        self.capacity=capacity
        self.size=0


    def get(self, key: int) -> int:
        if key not in self.cache:
            return -1
        node = self.cache[key]
        self.moveToHead(node)
        return node.value

    def put(self, key: int, value: int) -> None:
        if key not in self.cache:
            # 初始化
            node = DLinkListNode(key,value)
            self.cache[key]=node
            self.addToHead(node)
            self.size+=1

            if self.size>self.capacity:
                removed = self.removeTail()
                #pop
                self.cache.pop(removed.key)
                self.size-=1
        else:
            node=self.cache[key]
            node.value=value
            self.moveToHead(node)

    def addToHead(self,node):
        # o->o o->N->o
        head_next = self.head.next
        node.prev=self.head
        node.next=head_next
        head_next.prev=node
        self.head.next=node

    def removeNode(self,node):
        node.prev.next=node.next
        node.next.prev=node.prev
    

    def moveToHead(self,node):
        self.removeNode(node)
        self.addToHead(node)

    def removeTail(self):
        node = self.tail.prev
        self.removeNode(node)
        return node
~~~

操作还是封装在LRU里的

LRU果然没有经过测试的东西就是不可靠的。

有机会应该走一走单元测试，找点感觉。

---

### Redis的LRU实现

---



### 参考

- https://zhuanlan.zhihu.com/p/34133067
  - https://leetcode-cn.com/problems/lru-cache/solution/lruhuan-cun-ji-zhi-by-leetcode-solution/	