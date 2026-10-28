---
type: knowledge
status: active
moc:
  - "[[计算机基础 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---
## 题目描述

给定一个整型数组arr，再给定一个整数k，打印所有出现次数大于 N/K 的数。如果没有这样的数，打印提示信息。
【**要求**】
时间复杂度为`O(N*K)`,额外空间复杂度为`O(K)`。
标签：[[01_评价算法常用方法]]


## 题目解析

每次从数组中删除 K 个不同的数，如果某个数的次数大于 N/K ，这个数最后肯定会剩下来；

数学证明：假设 X 的次数为 $(N/K+1) > N/K$ ,如果每次删除 K个不同的数，如果最后剩余的数里面没有 X ，那么这个数删除的次数大于等于$（N/K+1）>N/K $，

那么：$K*（N/k+1）>N$,不成立因为数组中只有N个数不可能删除掉 比N还多的数。

注意：**删除后剩余的数不一定全是次数大于 N/K 的数**，例如：{1，2，3} k=2, 删除后数组中还剩余 3，但是 3 的次数为 1 ，因此**还要看剩余的数的次数是不是大于 N/K**.

算法过程：大小为 K 的空间，来保存不同的数，当有K个不同的数时，将每个数的次数减 1，将次数为1的数删除，如果空间中有这个数时，则将这个数的次数加 1 。

下面是数组arr[]={1 ,2 ,3 ,3 ,5 ,2 ,2 ,3 ,3 ,3 ,5 ,6 ,2 ,2 ,2 ,3 , 3}处理的简单过程图：

![这里写图片描述](https://img-1312072469.cos.ap-nanjing.myqcloud.com/20170422230107108)



```java
public static void printKMajor(int[] arr, int K) {
        if (K < 2) {
            System.out.println("the value of K is invalid.");
            return;
        }
        HashMap<Integer, Integer> cands = new HashMap<>();
        for (int i = 0; i < arr.length; i++) {
            //hashmap由于保存k个不同的数，当hashmap中有k个不同的数时，则每个数的次数减1，
            //如果某个数的次数为0，则将其从hashmap中删除
            if (cands.containsKey(arr[i])) {
                cands.put(arr[i], cands.get(arr[i]) + 1);
            } else {
                if (cands.size() == K - 1) {
                    //如果hashmap中不存在这个数，如此时hashmap中存在k-1个不同的数则将
                    //hashmap中的每个数的次数减1
                    allCandsMinusOne(cands);
                } else {
                    //hashmap中的不同数个数小于k-1，时将该数加入到hashmap中。
                    cands.put(arr[i], 1);
                }
            }
        }

        HashMap<Integer, Integer> reals = getDeals(arr, cands);
        boolean hasPrint = false;
        for (Map.Entry<Integer, Integer> set : cands.entrySet()) {
            Integer key = set.getKey();
            //如果某个数的次数大于K/N则将其输出
            if (reals.get(key) > arr.length / K) {
                hasPrint = true;
                System.out.print(key + " ");
            }
        }
        System.out.println(hasPrint ? "" : "no such number.");
    }

    public static void allCandsMinusOne(HashMap<Integer, Integer> map) {
        //将hashmap中的每个数的次数减1，将减 1 后次数为0 的从hashmap中删除。
        List<Integer> removeList = new LinkedList<Integer>();
        for (Map.Entry<Integer, Integer> set : map.entrySet()) {
            Integer key = set.getKey();
            Integer value = set.getValue();
            if (value == 1) {
                removeList.add(key);
            }
            map.put(key, value - 1);
            for (Integer removeKey : removeList) {
                map.remove(removeKey);
            }
        }
    }

    public static HashMap<Integer, Integer> getDeals(int[] arr, HashMap<Integer, Integer> cands) {
        HashMap<Integer, Integer> reals = new HashMap<>();
        for (int i = 0; i < arr.length; i++) {
            int curNum = arr[i];
            if (cands.containsKey(curNum)) {
                if (reals.containsKey(curNum)) {
                    reals.put(curNum, reals.get(curNum) + 1);
                } else {
                    reals.put(curNum, 1);
                }
            }
        }
        return reals;
    }
```

## 总结

这道题目还是得看视频。

## 相关题目
[[牛客_在数组中找到出现次数大于一半的数]]