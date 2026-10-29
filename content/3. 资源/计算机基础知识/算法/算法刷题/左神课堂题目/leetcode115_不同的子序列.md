---
type: knowledge
status: active
moc:
  - "[[计算机基础 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---
# 115.不同的子序列

这是 LeetCode上的 **[115.不同的子序列](https://leetcode.cn/problems/distinct-subsequences/)** ，难度为 **困难**。

对应左神训练营第4期第2节第四题。

标签：[[12_动态规划]],[[11_暴力递归]]

## 题目描述

给定一个字符串 `s` 和一个字符串 `t` ，计算在 `s` 的子序列中 `t` 出现的个数。

字符串的一个 **子序列** 是指，通过删除一些（也可以不删除）字符且不干扰剩余字符相对位置所组成的新字符串。（例如，`"ACE"` 是 `"ABCDE"` 的一个子序列，而 `"AEC"` 不是）

题目数据保证答案符合 32 位带符号整数范围。

**示例 1：**

**输入：**s = "rabbbit", t = "rabbit"
**输出** **：**3
**解释：**
如下图所示, 有 3 种可以从 s 中得到 "rabbit" 的方案。
**rabb**b**it**
**ra**b**bbit**
**rab**b**bit**

**示例 2：**

**输入：**s = "babgbag", t = "bag"
**输出** **：**5
**解释：**
如下图所示, 有 5 种可以从 s 中得到 "bag" 的方案。 
**ba**b**g**bag
**ba**bgba**g**
**b**abgb**ag**
ba**b**gb**ag**
babg**bag**

**提示：**

-   0 <= s.length, t.length <= 1000
-   s 和 t 由英文字母组成

## 题目解析

使用字符串s作为行，使用字符串t作为列，表示在字符串 s 的 0-i 中选出字符串 t 的 0-j 的情况总数，则存在两种情况：

1. 子序列不考虑i位置的字符。`dp[i][j] = dp[i - 1][j]`
2. 子序列一定要含有i位置的字符
   1. [i]位置字符跟j位置字符不相等，那么可能性2就一定不成立
   2. [i]位置字符跟j位置字符相等，可能性2）才有可能成立。`dp[i][j] = dp[i - 1][j - 1]`

```java
 // S[...i]的所有子序列中，包含多少个字面值等于T[...j]这个字符串的子序列
    // 记为dp[i][j]
    // 可能性1）S[...i]的所有子序列中，都不以s[i]结尾，则dp[i][j]肯定包含dp[i-1][j]
    // 可能性2）S[...i]的所有子序列中，都必须以s[i]结尾，
    // 这要求S[i] == T[j]，则dp[i][j]包含dp[i-1][j-1]
    public static int numDistinct2(String S, String T) {
        char[] s = S.toCharArray();
        char[] t = T.toCharArray();
        int[][] dp = new int[s.length + 1][t.length + 1];
        for (int j = 0; j <= t.length; j++) {
            dp[0][j] = 0;
        }
        for (int i = 0; i <= s.length; i++) {
            dp[i][0] = 1;
        }
        for (int i = 1; i <= s.length; i++) {
            for (int j = 1; j <= t.length; j++) {
                dp[i][j] = dp[i - 1][j] + (s[i - 1] == t[j - 1] ? dp[i - 1][j - 1] : 0);
            }
        }
        return dp[s.length][t.length];
    }
```

时间复杂度为$O(M \times N)$，空间复杂度为$O(M \times N)$
## 总结
关于字符串匹配，通常有两种（你也可以理解为一种）通用的状态定义：

- $s[i][j]$ 表示「第一个字符 S中 $[0,i]$ 个字符」与「第二个字符 T 中 $[0,j]$个字符」的匹配结果
- $f[i][j]$表示「第一个字符S中$ [0,i]$个字符」与「第二个字符 T中 $[0,j]$个字符」且 「最后一个字符为 $t[j]$」的匹配结果
- 往两个字符串的头部追加「不存在」的字符，目的是为了能够构造出可以滚动（被累加）下去的初始化值
