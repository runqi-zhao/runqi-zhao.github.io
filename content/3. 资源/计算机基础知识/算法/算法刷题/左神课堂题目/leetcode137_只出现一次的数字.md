---
type: knowledge
status: active
moc:
  - "[[计算机基础 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---
这是 LeetCode上的 **[137.只出现一次的数字](https://leetcode.cn/problems/distinct-subsequences/)**的**拓展** ，难度为 **困难**。

对应左神训练营第4期第3节第四题。

标签：[[位运算]]

## 题目描述

给定一个数组arr，已知除了一种数只出现1次之外，剩下所有的数都出现了k次，如何使用O(1)的额外空间，找到这个数。

## 题目解析

以`k==3`为例

### 方法一：哈希表

一个朴素的做法是使用「哈希表」进行计数，然后将计数为 111 的数字进行输出。

哈希表以「数值 : 数值出现次数」形式进行存储。

代码：

```java
class Solution {
    public int singleNumber(int[] nums) {
        Map<Integer, Integer> map = new HashMap<>();
        for (int x : nums) {
            map.put(x, map.getOrDefault(x, 0) + 1);
        }
        for (int x : map.keySet()) {
            if (map.get(x) == 1) return x;
        }
        return -1;
    }
}
```

时间复杂度：$O(n)$
空间复杂度：$O(n)$

### 方法二：位数统计

哈希表解法的空间复杂度是 $O(n)$的，而题目的【进阶】部分提到应当使用常数空间来做。

其中一个比较容易想到的做法，是利用 $int$类型固定为 $32$位。

使用一个长度为 32的数组$ cnt[]$c记录下所有数值的每一位共出现了多少次 111，再对 $cnt[]$ 数组的每一位进行$mod$$3$操作，重新拼凑出只出现一次的数值。

举个 🌰，考虑样例 [1,1,1,3]，1和 3对应的二进制表示分别是 00..001 和 00..011，存入 $cnt[]$数组后得到 [0,0,...,0,1,4]。进行 $mod$$3$ 操作后得到 [0,0,...,0,1,1]，再转为十进制数字即可得「只出现一次」的答案 $3$。

代码：

```java
class Solution {
    public int singleNumber(int[] nums) {
        int[] cnt = new int[32];
        for (int x : nums) {
            for (int i = 0; i < 32; i++) {
                if (((x >> i) & 1) == 1) {
                    cnt[i]++;
                }
            }
        }
        int ans = 0;
        for (int i = 0; i < 32; i++) {
            if ((cnt[i] % 3 & 1) == 1) {
                ans += (1 << i);
            }
        }
        return ans;
    }
}
```

时间复杂度：$O(n)$
空间复杂度：$O(1)$

## 总结



## 相关题目