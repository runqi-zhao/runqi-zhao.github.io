---
type: knowledge
status: active
moc:
  - "[[计算机基础 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---
# 1312.让字符串成为回文串的最少插入次数

这是 LeetCode 上的 **[1312.让字符串成为回文串的最少插入次数](https://leetcode.cn/problems/minimum-insertion-steps-to-make-a-string-palindrome/)** ，难度为 **困难**。
对应左神训练营第3期第7节第五题。

标签:[[12_动态规划]]

## 题目描述

给你一个字符串 `s` ，每一次操作你都可以在字符串的任意位置插入任意字符。

请你返回让 `s` 成为回文串的 **最少操作次数** 。

「回文串」是正读和反读都相同的字符串。

**示例 1：**

```
输入：s = "zzazz"
输出：0
解释：字符串 "zzazz" 已经是回文串了，所以不需要做任何插入操作。
```

**示例 2：**

```
输入：s = "mbadm"
输出：2
解释：字符串可变为 "mbdadbm" 或者 "mdbabdm" 。
```

**示例 3：**

```
输入：s = "leetcode"
输出：5
解释：插入 5 个字符后字符串变为 "leetcodocteel" 。
```

**提示：**

- `1 <= s.length <= 500`
- `s` 中所有字符都是小写字母。

## 题目解析

这是一个范围上面的尝试，直接上代码，很简单：

```java
public static String getPalindrome1(String s) {
        if (s == null || s.length() < 2) {
            return s;
        }
        int N = s.length();
        char[] str = s.toCharArray();
        int[][] dp = new int[N][N];
        for (int j = 1; j < N; j++) {
            dp[j - 1][j] = str[j - 1] == str[j] ? 0 : 1;
            for (int i = j - 2; i >= 0; i--) {
                if (str[i] == str[j]) {
                    dp[i][j] = dp[i + 1][j - 1];
                } else {
                    dp[i][j] = Math.min(dp[i + 1][j], dp[i][j - 1]) + 1;
                }
            }
        }
        char[] res = new char[str.length + dp[0][str.length - 1]];
        int i = 0;
        int j = str.length - 1;
        int resl = 0;
        int resr = res.length - 1;
        while (i <= j) {
            if (str[i] == str[j]) {
                res[resl++] = str[i++];
                res[resr--] = str[j--];
            } else if (dp[i][j - 1] < dp[i + 1][j]) {
                res[resl++] = str[j];
                res[resr--] = str[j--];
            } else {
                res[resl++] = str[i];
                res[resr--] = str[i++];
            }
        }
        return String.valueOf(res);
    }
```

## 总结

其实动态规划并不难，主要是要掌握对应的模型，就能解出

## 相关题目

