---
title: LeetCode-打家劫舍问题
date: 2020-12-28 19:13:10
tags: LeetCode
---

打家劫舍问题~

<!--more-->

LEECODE198.打家劫舍1

> 你是一个专业的小偷，计划偷窃沿街的房屋。每间房内都藏有一定的现金，影响你偷窃的唯一制约因素就是相邻的房屋装有相互连通的防盗系统，如果两间相邻的房屋在同一晚上被小偷闯入，系统会自动报警。
>
> 给定一个代表每个房屋存放金额的非负整数数组，计算你 不触动警报装置的情况下 ，一夜之内能够偷窃到的最高金额。

大体思路:这个问题描述很好玩，但核心就是不能搞相邻的。

所以核心状态转移方程的话就是:`max(dp[i-2]+nums[i],dp[i-1])`

~~~python
class Solution:
    def rob(self, nums: List[int]) -> int:
        # dp数组的定义很重要就是打劫前n家的最大收入
        if not nums:return 0
        if len(nums)==1:return nums[0]
        n=len(nums)
        dp = [0 for _ in range(n)]
        dp[0] = nums[0]
        dp[1] = max(nums[0], nums[1])
        for i in range(2, n):
            dp[i] = max(dp[i - 2] + nums[i], dp[i - 1])
        return dp[n - 1]

~~~

LEECODE213.打家劫舍2

> 你是一个专业的小偷，计划偷窃沿街的房屋，每间房内都藏有一定的现金。这个地方所有的房屋都 围成一圈 ，这意味着第一个房屋和最后一个房屋是紧挨着的。同时，相邻的房屋装有相互连通的防盗系统，如果两间相邻的房屋在同一晚上被小偷闯入，系统会自动报警 。
>
> 给定一个代表每个房屋存放金额的非负整数数组，计算你 在不触动警报装置的情况下 ，能够偷窃到的最高金额。

和上一题相比 ，多了一个头尾相连 就仅仅多了一个条件

如果等于2 那就抢不得 但是显然没有那么简单

太妙了！！ 把他拆解成 不包含头 和不包含尾 然后取一个最大值 鬼才啊

~~~python
class Solution:
    def rob(self, nums: List[int]) -> int:
        if not nums:return 0
        if len(nums)==1:return nums[0]
        if len(nums)==0:return 0

        return max(self.rob1(nums[1:]),self.rob1(nums[:-1]))

    def rob1(self, nums: List[int]) -> int:
        # dp数组的定义很重要就是打劫前n家的最大收入
        if not nums:return 0
        if len(nums)==1:return nums[0]
        n=len(nums)
        dp = [0 for _ in range(n)]
        dp[0] = nums[0]
        dp[1] = max(nums[0], nums[1])
        for i in range(2, n):
            dp[i] = max(dp[i - 2] + nums[i], dp[i - 1])
        return dp[n - 1]

~~~

LEETCODE.337 打家劫舍3

> 在上次打劫完一条街道之后和一圈房屋后，小偷又发现了一个新的可行窃的地区。这个地区只有一个入口，我们称之为“根”。 除了“根”之外，每栋房子有且只有一个“父“房子与之相连。一番侦察之后，聪明的小偷意识到“这个地方的所有房屋的排列类似于一棵二叉树”。 如果两个直接相连的房子在同一天晚上被打劫，房屋将自动报警。

打劫二叉树是我万万没有想到的。好家伙

我们可以用 f(o)f(o) 表示选择 oo 节点的情况下，oo 节点的子树上被选择的节点的最大权值和；g(o)g(o) 表示不选择 oo 节点的情况下，oo 节点的子树上被选择的节点的最大权值和；ll 和 rr 代表 oo 的左右孩子。



~~~c++
class Solution {
public:
    unordered_map <TreeNode*, int> f, g;

    void dfs(TreeNode* o) {
        if (!o) {
            return;
        }
        dfs(o->left);
        dfs(o->right);
      # 如果选了root 就不能选左右孩子。
      # 如果没选root 就是选左 或者选右 的最大值。
      # 看着还挺麻烦的。。
      
        f[o] = o->val + g[o->left] + g[o->right];
        g[o] = max(f[o->left], g[o->left]) + max(f[o->right], g[o->right]);
    }

    int rob(TreeNode* o) {
        dfs(o);
        return max(f[o], g[o]);
    }
};
~~~

