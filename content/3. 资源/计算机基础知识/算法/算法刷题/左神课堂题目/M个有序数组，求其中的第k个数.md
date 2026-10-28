---
type: knowledge
status: active
moc:
  - "[[计算机基础 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---
这道题目疑似来源于谷歌面试题，难度个人判断困难。

对应左神训练营第4期第4节第2题。

标签：[[01_评价算法常用方法]]

## 题目描述

给定两个整数数组A和B

A是长度为m、元素从小到大排好序了

B是长度为n、元素从小到大排好序了

希望从A和B数组中，找出最大的k个数字

## 题目解析

### 上中位数

上中位数：等长、有序，两个合起来的中位数

进行二分法，选择A的中位数与B的中位数进行比较，相等返回，A<B则在A的右边进行分割，A>B则在A的左边进行分割，如果发现A中无论怎么划分都不存在，则证明在B

```java
public static int getUpMedian(int[] A, int s1, int e1, int[] B, int s2, int e2, int kth) {
        int mid1 = 0;
        int mid2 = 0;
        while (s1 < e1) {
            mid1 = (s1 + e1) / 2;
            mid2 = (s2 + e2) / 2;
//            int offset = ((e1 - s1 + 1) & 1) ^ 1;
//            if (A[mid1] == B[mid2]) {
//                return A[mid1];
//            } else if (A[mid1] > B[mid2]) {
//                return A[mid1];
//            } else {
//                s1 = mid1 + offset;
//                e2 = mid2;
//            }
            if (A[mid1] == B[mid2]) {
                return A[mid1];
            }
            if (((e1 - s1 + 1) & 1) == 1) {
                if (A[mid1] > B[mid2]) {
                    if (B[mid2] >= A[mid1 -1]) {
                        return B[mid2];
                    }
                    e1 = mid1 - 1;
                    s2 = mid2 + 1;
                } else { //A[mid1] < B[mid2]
                    if (A[mid1] >= B[mid2 - 1]) {
                        return A[mid1];
                    }
                    e2 = mid2 - 1;
                    s1 = mid1 + 1;
                }
            } else {
                if (A[mid1] > B[mid2]) {
                    e1 = mid1;
                    s2 = mid2 + 1;
                } else {
                    e2 = mid2;
                    s1 = mid1 + 1;
                }
            }
        }
        return Math.min(A[s1],B[s2]);
    }
```

下面开始解题

考虑边界问题

举例:A:10;B:17

1. 1<=k<=10 如第7小，求出A的前7个与B的前7个进行求解，就能求出第7小；
2. 11<k <=17，如第15小，
   1. A中都有可能；
   2. B中前4个没有可能，同时16 17没有可能；
   3. A的候选区是1-10，B的候选区是5-15，但是此时的长度不一样，因此我们需要人为的判断5的位置；
      1. 如果是>=A中的10，则此时B中的5就是第15小；
      2. 如果不是，那么B从6-15进行选择。此时长度为一样的。
3. 18 <K <=27, 如23小，
   1. A从6开始有可能；
   2. B从13开始有可能；
   3. 此时我们淘汰的数是5+12=17个数；
   4. 但是我们的后选串中只能选出第5小，此时17+5=22，不是我们需要的数，因此是得不到23小的数；
   5. 解决办法：
      1. 验证A中的6是不是第23小，如果是返回，如果不是淘汰；
      2. 再次验证B中的13是不是第23小，如果是返回，如果不是，那么此时淘汰的数是6+13=19个，后面选出第4小，就是我们所求。

```java
 public static int findKthNum(int[] arr1, int[] arr2, int kth) {
        if (arr1 == null || arr2 == null) {
            return -1;
        }
        if (kth < 1 || kth > arr1.length + arr2.length) {
            return -1;
        }
        int[] longest = arr1.length > arr2.length ? arr1 : arr2;
        int[] smallest = longest == arr1 ? arr2 : arr1;
        int l = longest.length;
        int s = smallest.length;
        if (kth <= s) {
            return getUpMedian(longest, 0, kth - 1, smallest, 0, kth - 1);
        } else if (kth > l) {
            if (smallest[kth - l - 1] > longest[l - 1]) {
                return smallest[kth - l - 1];
            }
            if (longest[kth - s - 1] > smallest[s - 1]) {
                return longest[kth - l - 1];
            }
            return getUpMedian(smallest, kth - l, s - 1, longest, kth - s, l - 1);
        } else {
            if (longest[kth - s - 1] >= smallest[s - 1]) {
                return longest[kth - s - 1];
            }
            return getUpMedian(smallest, 0, s - 1, longest, kth - s, kth - 1);
        }
    }
```

## 总结

这道题目其实就是求取中文书的变形，首先我们要想到二分法求取两个等长的中位数，然后再想到分情况讨论。

## 相关题目