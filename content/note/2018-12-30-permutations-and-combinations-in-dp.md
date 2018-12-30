---
title: 背包问题中的排列与组合
subtitle: 两层循环的嵌套顺序的区别
date: '2018-12-30'
slug: permutations-and-combinations-in-dp
categories:
  - Algorithms
tags:
  - dynamic programming
---

本文以背包问题中的硬币/零钱问题来阐述在动态规划中存在的排列与组合问题。

## 1 问题描述
[LeetCode 322. Coin Change](https://leetcode.com/problems/coin-change/)

> You are given coins of different denominations and a total amount of money *amount*. Write a function to compute the fewest number of coins that you need to make up that amount. If that amount of money cannot be made up by any combination of the coins, return `-1`.

## 2 基本思路
这是最基本的完全背包问题，根据状态定义的不同，可以衍生出两个不同版本的算法。

### 2.1 状态定义方式一：Use-It-or-Lose-It
**此种方式将状态划分为 2 个维度**，即 dp[i][a] 表示总面值为 a，并且只使用前 i 种硬币的情况下，最少需要用到的硬币个数。其状态转移方程为：

`$${\rm dp} \left [ i \right ] \left [ a \right ] = {\rm min} \left \{ {\rm dp} \left [ i - 1 \right ] \left [ a - k \cdot {\rm coins} \left [ i \right ] \right ] | k \cdot {\rm coins} \left [ i \right ] \leq a , k = 0, 1, 2, ... \right \}$$`

将其转化为 01 背包问题，用 Use-It-or-Lose-It 的思路可以得到以下状态转移方程：

dp[i][a] = min{dp[i-1][a], dp[i][a-coins[i]] + 1}

这种定义方式的特点为：**硬币选择的顺序是提前已经确定了的（即其在数组 coins 中的顺序），因此同一硬币组合方式只被计算了一次，即使其有多种排列方式。**

以一维数组递推法为例，其在具体代码的实现上的特征为：**硬币的面值数组 coins 的循环在外层，表示目前可用的硬币种类，而需要兑换的总面值的循环在内层。**

伪代码如下：

```
for i = 0 ... N - 1
    for a = 0 ... amount
        dp[a] = min{dp[a], dp[a - coins[i]] + 1}
```

C++ 代码如下：

```cpp
int coinChange(vector<int>& coins, int amount) {
    // Use-It-or-Lose-It: Combinations Version

    int MAX = amount + 1; // MAX means no valid solution
    vector<int> dp(amount + 1, MAX);

    // Base case, amount = 0
    dp[0] = 0;

    // Iterately fill in dp table.
    for (size_t i = 0; i < coins.size(); i++) {
        for (int a = 0; a <= amount; a++) {
            if (coins[i] <= a) {
                int useIt = dp[a - coins[i]] + 1;
                int loseIt = dp[a];
                dp[a] = min(useIt, loseIt);
            }
        }
    }

    return dp[amount] == MAX ? -1 : dp[amount];
}
```

### 2.2 状态定义方式二：DAG 上的动态规划
参考上一篇文章：[动态规划的相关细节探讨 -- DAG 上的动态规划](https://kaizhang.me/note/2018/12/more-dynamic-programming/)

**此种方式将状态只划分为 1 个维度**，即 dp[a] 表示总面值为 a，并且使用所有硬币的情况下，最少需要用到的硬币个数。其状态转移方程为：

`$${\rm dp} \left [ a \right ] = {\rm min} \left \{ {\rm dp} \left [ a - {\rm coins} \left [ i \right ] \right ] | {\rm coins} \left [ i \right ] \leq a , i = 0, 1, ..., N - 1 \right \}$$`

这种定义方式的特点为：**每一步中所有的硬币均可选，硬币选择的顺序是不确定的，因此同一硬币组合方式的每一种排列方式均被计算了一次**。如 1 3 5 和 5 3 1 是被视为两种不同的方案。

其在具体代码的实现上的特征为：**需要兑换的总面值的循环在外层，硬币的面值数组 coins 的循环在内层，表示当前考虑的下一枚硬币。**

伪代码如下：

伪代码如下：

```
for a = 0 ... amount
    for i = 0 ... N - 1
        dp[a] = min{dp[a], dp[a - coins[i]] + 1}
```

C++ 代码如下：

```cpp
int coinChange(vector<int>& coins, int amount) {
    // DAG: Permutations Version

    int MAX = amount + 1; // MAX means no valid solution
    vector<int> dp(amount + 1, MAX);

    // Base case, amount = 0
    dp[0] = 0;

    // Iterately fill in dp table.
    for (int a = 1; a <= amount; a++) {
        for (int i = 0; i < coins.size(); i++) {
            if (coins[i] <= a) {
                dp[a] = min(dp[a], dp[a - coins[i]] + 1);
            }
        }
    }
    return dp[amount] == MAX ? -1 : dp[amount];
}
```

### 2.3 两种方式的对比
在形式上二者非常相似 ，唯一的区别在于两层 for 循环的嵌套顺序问题。

在意义上二者区别是很大的，首先前者的 DP table 是二维的，只是在具体的代码实现上为了节约空间通过一些技巧达到了用一维数组来存储这些信息（代价是丢失了重建 solution 的能力）。而后者的 DP table 本身就是一维的。

但是二者的时间复杂度是一致的，因为虽然前者的 DP table 的大小为 O(n * amount)，但是更新每个表项的时间是 O(1)，因此总的时间为 O(n * amount)。而对于后者而言，虽然 DP table 的大小为 O(amount)，但是更新每个表项的时间是 O(n)，因此总的时间仍为 O(n * amount)。

## 3. 对于方案计数问题要分清楚问的组合计数还是排列计数
[LeetCode 518. Coin Change 2](https://leetcode.com/problems/coin-change-2/)

问题描述：

You are given coins of different denominations and a total amount of money. Write a function to compute the number of combinations that make up that amount. You may assume that you have infinite number of each kind of coin.

对于求最优解的问题，采用前述的组合或排列的方式是没有区别的，结果是一致的，时间、空间复杂度也是一致的。但是对于方案计数问题，需要分清楚题目问的是组合还是排列的种类个数。

对于本题而言，对于同一种硬币组成而言，其排列顺序我们是不关心的，因此是一种组合计数问题，需要采用 2.1 节中的 Use-It-or-Lose-It 的状态定义方式。

此种方式将状态划分为 2 个维度，即 dp[i][a] 表示总面值为 a，并且只使用前 i 种硬币的情况下，可能存在的组合总数。“无法到达”的标志值为 0。其状态转移方程为：

dp[i][a] = dp[i][a - coins[i]] + dp[i][a]

其中：

useIt = dp[i][a - coins[i]];  
loseIt = dp[i - 1][a];  
dp[i][a] = useIt + loseIt;

C++ 代码如下：

```cpp
int change(int amount, vector<int>& coins) {
    // Use-It-or-Lose-It: Combinations Version

    vector<int> dp(amount + 1, 0); // 0 means no valid solution

    // Base case: amount = 0.
    dp[0] = 1;

    // Iterately fill in dp table.
    for (size_t i = 0; i < coins.size(); i++) {
        for (int a = 0; a <= amount; a++) {
            if (coins[i] <= a) {
                int useIt = dp[a - coins[i]];
                int loseIt = dp[a];
                dp[a] = useIt + loseIt;
            }
        }
    }

    return dp[amount];
}
```
