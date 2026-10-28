---
type: knowledge
status: active
moc:
  - "[[计算机基础 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---
未找到出处，难度认为中等。
对应左神训练营第4期第5节第1题。
标签：[[12_动态规划]]
## 题目描述
给定一个字符串str，求最长回文子序列长度。
## 题目解析
使用的是范围尝试模型，多次使用过，这里不再详细赘述。
```java
public static int lcse(char[] str1, char[] str2) {  
    int[][] dp = new int[str1.length][str2.length];  
    dp[0][0] = str1[0] == str2[0] ? 1 : 0;  
    for (int i = 1; i < str1.length; i++) {  
        dp[i][0] = Math.max(dp[i - 1][0], str1[i] == str2[0] ? 1 : 0);  
    }  
    for (int j = 1; j < str2.length; j++) {  
        dp[0][j] = Math.max(dp[0][j - 1], str1[0] == str2[j] ? 1 : 0);  
    }  
    for (int i = 1; i < str1.length; i++) {  
        for (int j = 1; j < str2.length; j++) {  
            dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);  
            if (str1[i] == str2[j]) {  
                dp[i][j] = Math.max(dp[i][j], dp[i - 1][j - 1] + 1);  
            }  
        }  
    }  
    return dp[str1.length - 1][str2.length - 1];  
}
```
## 总结
动态规划的分类，需要搞清楚
## 相关题目
[[马拉车算法]]