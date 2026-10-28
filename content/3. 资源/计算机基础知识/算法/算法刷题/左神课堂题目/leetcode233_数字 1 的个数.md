---
type: knowledge
status: active
moc:
  - "[[计算机基础 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---
这是 LeetCode 上的 **[233.数字 1 的个数](https://leetcode.cn/problems/word-search-ii/description/)** ，难度为 **困难**。
对应左神训练营第3期第8节第三题。
标签：[[12_动态规划]]

## 题目描述

给定一个正数N，表示你在纸上写下1~N所有的数字

返回在书写的过程中，一共写下了多少个1

## 题目解析

这是一种极为特殊的动态规划，`数位dp`。建议记住算了。

 假设一个数是N，一共有K位，则有下面两种情况：

1. 最高位是1 :  $(n\% 10^{(k-1)} + 1) +  (10^{(k-2)}\times(k-1)\times1)$     
2.  最高位不是1: $10^{(k  -1)} + 10^{(k-2)} \times(k-1) \times first$

现在使用非递归的进行

> 举个例子，13625，现在这种情况下最高位是1，那么可以划分成以下几段：
>
> 1. 3626-13625上面，万位上出现1是10000-13625($(n\% 10^{(k-1)} + 1)$)，出现3626，千位上面是1，百位，十位，个位随意变，此时都会存在一个数字，确保千位上数字为1，那么千位上面就是$10^3$，百位上面也相同，也是$10^3$（2105），各位也相同。
>
> 2. 626-3625，此时最高位不是1，千位上面1000-1999是1，即$10^{(k-1)}$，百位上，还要继续分段：
>
>    1. 626-1625
>    2. 1626-2625
>    3. 2626-3625
>
>    十位与个位随意遍，此时一定每一段里面都存在一个($10^{k-2}\times 3$)，同理，十位与个位也行同
>
> 3. 26-625
>
> 4. 1-25
>
>    接下来就是遍历。

因此，可以写出下面代码

```java
 public static int solution2(int num) {
        if (num < 1) {
            return 0;
        }
        //12345
        //5位
        int len = getLenOfNum(num);
        if (len == 1) {
            return 1;
        }
        // 12345
        // 10000
        int temp1 = powerBaseOf10(len - 1);
        // num最高位 num / temp1
        int first = num / temp1;

        // 最高位1 N % temp1 + 1
        //最高位first temp1
        int firstNum = first == 1 ? num % temp1 + 1 : temp1;

        // 除去最高位之外，剩下1的数量
        // 最高位1 $10^(k-2) \times (k - 1) \times 1$
        // 最高位first $10^(k - 2) \times (k - 1) * first$
        int otherNum = first * (len - 1) * (temp1 / 10);
        return firstNum + otherNum + solution2(num % temp1);
   }

   public static int getLenOfNum(int num) {
        int len  = 0;
        while (num != 0) {
            len++;
            num /= 10;
        }
        return len;
   }

   public static int powerBaseOf10(int base) {
        return (int)Math.pow(10, base);
   }
```

时间复杂度为$O(log_{10}n) \times O(log_{10}n)$

## 总结

这是一道经典的数位dp，看例子吧，是在不用理解其中的原理。

## 相关题目
