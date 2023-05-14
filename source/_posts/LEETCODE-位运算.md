---
title: LeetCode-位运算
date: 2020-12-28 19:13:34
tags: LeetCode
---

位运算感觉是个坑...你不会吧，你就完全不会了。你懂了会做相关的题了。

<!--more--> 

常用操作	

~~~python
X & 1 == 1 or ==0 判断奇偶
X = X&(X-1) => 清除最低位的1
X&-X => 得到最低位的1
~~~

LEETCODE191.统计1的个数

~~~python
class Solution:
    def hammingWeight(self, n: int) -> int:
        count=0
        x=n
        while x!=0:
            x=x&(x-1)
            count+=1
        return count
~~~

LEETCODE231 2的幂

2的幂只有0

~~~python
class Solution:
    def isPowerOfTwo(self, n: int) -> bool:
        # 2的幂只要 0 1
        #  -1 
        return n>0 and not(n & n-1)

# 拆开来更好一些    
class Solution:
    def isPowerOfTwo(self, n: int) -> bool:
        # 2的幂只要 0 1
        #  -1 
        if n<=0:return False
        if n&(n-1)==0:return True
        else:
            return False
~~~

LEETCODE338. 比特位计数

> 给定一个非负整数 **num**。对于 **0 ≤ i ≤ num** 范围中的每个数字 **i** ，计算其二进制数中的 1 的数目并将它们作为数组返回。

答案挺有意思的。

~~~python
class Solution:
    def countBits(self, num: int) -> List[int]:
        
        count_list =[0 for i in range(num+1)]

        for i in range(1,num+1):
            count_list[i]=count_list[i&(i-1)]+1
        
        return count_list
~~~

