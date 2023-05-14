---
title: LeetCode-DP 问题
date: 2020-12-30 14:55:28
tags:
---

dp问题~

<!--more-->

1.编辑距离问题

~~~python
class Solution:
    def minDistance(self, word1, word2):
        """
        :type word1: str
        :type word2: str
        :rtype: int
        """
        n = len(word1)
        m = len(word2)
        
        # 有一个字符串为空串
        if n * m == 0:
            return n + m
        
        # DP 数组
        D = [ [0] * (m + 1) for _ in range(n + 1)]
        
        # 边界状态初始化
        for i in range(n + 1):
            D[i][0] = i
        for j in range(m + 1):
            D[0][j] = j
        
        # 计算所有 DP 值
        for i in range(1, n + 1):
            for j in range(1, m + 1):
                left = D[i - 1][j] + 1
                down = D[i][j - 1] + 1
                left_down = D[i - 1][j - 1] 
                if word1[i - 1] != word2[j - 1]:
                    left_down += 1
                D[i][j] = min(left, down, left_down)
        
        return D[n][m]
~~~

---

2.最长公共子序列

这是两个序列 (反而跟编辑距离类似)

~~~python
class Solution:
    def longestCommonSubsequence(self,str1, str2) -> int:
        m, n = len(str1), len(str2)
        # 构建 DP table 和 base case
        dp = [[0] * (n + 1) for _ in range(m + 1)]
        # 进行状态转移
        for i in range(1, m + 1):
            for j in range(1, n + 1):
                if str1[i - 1] == str2[j - 1]:
                    # 找到一个 lcs 中的字符
                    dp[i][j] = 1 + dp[i-1][j-1]
                else:
                  	# 最大的
                    dp[i][j] = max(dp[i-1][j], dp[i][j-1])
            
        return dp[-1][-1]
~~~

3.最长上升子序列

名字听着差不多 其实差好多。

~~~python
class Solution:
    def lengthOfLIS(self, nums: List[int]) -> int:
        n = len(nums)
        dp = [1 for i in range(n)]

        for i in range(n):
            for j in range(i):
                if nums[j]<nums[i]:
                    dp[i]=max(dp[i],dp[j]+1)

        return max(dp)
~~~



4.零钱兑换一