---
title: LeetCode-反转链表相关
date: 2020-12-17 20:36:42
tags: LeetCode
---

反转链表的题还挺多的。也算是做出点感觉来了。

<!--more--> 

##### LEETCODE206.反转链表

~~~python
# class ListNode:
#     def __init__(self, x):
#         self.val = x
#         self.next = None
class Solution:
    def reverseList(self, head: ListNode) -> ListNode:
      	if not head:return None
        prev= None
        while head:
          
          #交代后事
          next_p = head.next
          head.next=prev
          prev=head
          head=next_p
				
        return prev
~~~

##### LEETCODE92 反转链表II

>  反转从位置 *m* 到 *n* 的链表。请使用一趟扫描完成反转。

大体思路: 移动m格 找到prev 移动m-n 格找到后续 然后反转

~~~python
class Solution:
    def reverseBetween(self, head: ListNode, m: int, n: int) -> ListNode:
      # 主要还是reverse这部分会写迷糊了
      # 思路清晰一下 prev cur 在动
      # head tail 不动
        def reverse(tail,head):
            prev=tail.next
            cur=head
            while prev!=tail:
                next_p = cur.next
                cur.next=prev
                prev=cur
                cur=next_p          
            return head,tail

        # 惯例传统艺能 dummy
        dummy = ListNode(0)
        dummy.next =head
        
        prev=dummy
       # 先不考虑 模拟一下假设m=2 那么prev走到1 cur_head走到2
        for i in range(m-1):
            prev=prev.next
       
        cur_head =prev.next
        tail=cur_head
        for i in range(n-m):
       	    tail=tail.next
        next_p = tail.next
       
       # 现在前面节点和后面节点都交代完了，可以反转
        tail,cur_head=reverse(tail,cur_head)
      # 现在tail就是实际的tail
        tail.next=next_p
        prev.next=cur_head
      
        return dummy.next
~~~

##### LEETCODE25 K个一组翻转链表

> 给你一个链表，每 k 个节点一组进行翻转，请你返回翻转后的链表。
>
> 如果节点总数不是 k 的整数倍，那么请将最后剩余的节点保持原有顺序。

大体思路:和上面一题还是蛮像的。就是不用提前走m，直接走k

>总是能踩到莫名的坑 昨天是参数位置错了。今天是被prev和pre坑了

~~~python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution:
    def reverse(self,head,tail):
        prev=tail.next
        cur=head
        while prev!=tail:
            next_p=cur.next
            cur.next=prev
            prev=cur
            cur=next_p
        return tail,head
    def reverseKGroup(self, head: ListNode, k: int) -> ListNode:
		    # 惯例传统艺能 dummy
        dummy = ListNode(0)
        dummy.next =head
        
        prev=dummy
        
        while head:
            tail =prev
            for i in range(k):
                tail=tail.next
                if not tail:return dummy.next
          
          # 保留后一节点指针
            nex = tail.next
            head,tail=self.reverse(head,tail)
					
          #续上
            prev.next=head
            tail.next=nex
          
          # 迭代
            prev=tail
            head=tail.next

        return dummy.next
~~~

##### LEETCODE24.两两交换列表中的节点

那这不就是K=2吗

> 太麻烦了 不如正常解~

~~~python
class Solution:
    def swapPairs(self, head: ListNode) -> ListNode:
        return self.reverseKGroup(head,2)
    def reverse(self,head,tail):
        prev=tail.next
        cur=head
        while prev!=tail:
            next_p=cur.next
            cur.next=prev
            prev=cur
            cur=next_p
        return tail,head
    def reverseKGroup(self, head: ListNode, k: int) -> ListNode:
		    # 惯例传统艺能 dummy
        dummy = ListNode(0)
        dummy.next =head
        
        prev=dummy
        
        while head:
            tail =prev
            for i in range(k):
                tail=tail.next
                if not tail:return dummy.next
          
          # 保留后一节点指针
            nex = tail.next
            head,tail=self.reverse(head,tail)
					
          #续上
            prev.next=head
            tail.next=nex
          
          # 迭代
            prev=tail
            head=tail.next

        return dummy.next
~~~



嘿嘿 再有其他的翻转链表我也不至于不会了