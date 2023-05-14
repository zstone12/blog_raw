---
title: LeetCode-K 个问题
date: 2020-12-29 15:51:13
tags: LeetCode
---

这是个不能称作问题集合的问题集合

<!--more-->

----

##### LEETCODE25: K个一组反转链表

~~~python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
# 先写一写思路吧 
class Solution: 
    def reverse(self,head,tail):
        prev=tail.next
        cur =head
        # 反转这边就写的有问题
        # 应该是prev!=tail
        while cur!=prev.next:
        	next_p=cur.next
          cur.next=prev
          prev=cur
          cur=next_p
        
        return tail,head
        
    def reverseKGroup(self, head: ListNode, k: int) -> ListNode:
        # 写了两三遍了 希望这次能bug free
        dummy = ListNode(0)
        dummy.next = head
        
        prev=dummy
        cur=head
        
        while cur:
          
          tail=prev # 保留prev
          for i in range(k):
            tail=tail.next
            if not tail:
              return dummy.next
            
          next_p =tail.next# 保留next
          cur,tail =self.reverse(cur,tail)
          
          prev.next=cur
          tail.next=next_p
          
          prev=tail
          cur=next_p
          
        # 先只管写吧
        
        
        #记是记不住的只能靠自己写
        # k=2
        
        return dummy.next
        
~~~

这次写的倒是还行 。就是反转这边出了一点问题。肌肉记忆了都快

---

##### LEETCODE23.合并K个升序链表

> 不会写

这个归并妙啊 三目表达式用的好。这边为什么要那啥呢

归并还是很漂亮的~

思路还是清晰的~

okok股票完就下班

~~~c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
    //剩下的就是mergeTwoList
    ListNode* mergeTwoList(ListNode *l1,ListNode* l2){
        if (l1 == nullptr)return l2;
        if (l2 == nullptr)return l1;
        ListNode *cur = new ListNode;
        ListNode *dummy = new ListNode;
        cur =dummy;
        while (l1 && l2){
            if (l1->val<l2->val){
                dummy->next=l1;
                l1=l1->next;
            }else{
                dummy->next=l2;
                l2=l2->next;
            }
            dummy=dummy->next;
        }
        if (l1 != nullptr)dummy->next=l1;
        if (l2 != nullptr)dummy->next=l2;
        return cur->next;
    }
    ListNode* merge(vector<ListNode*>& lists,int left,int right){
        if(left==right){
            return lists[left];
        }
        if(left>right){
            return nullptr;
        }   
        int mid = (left+right)>>1;
        //这边还是容易忘
        return mergeTwoList(merge(lists,left,mid),merge(lists,mid+1,right));

    }
    ListNode* mergeKLists(vector<ListNode*>& lists) {
        return merge(lists,0,lists.size()-1);
    }
};
~~~

