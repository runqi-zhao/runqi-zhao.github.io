---
type: knowledge
status: active
moc:
  - "[[计算机基础 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---
# 246 · 二叉树的路径和 II

这是 LintCode 上的 **[246·二叉树的路径和 II](https://www.lintcode.com/problem/246/description)** ，难度为 **中等**。
对应左神训练营第4期第3节第3题。

标签：[[07_二叉树]],[[08_二叉树的递归套路]],[[11_暴力递归]]

## 题目描述

给一棵二叉树和一个目标值，设计一个算法找到二叉树上的和为该目标值的所有路径。路径可以从任何节点出发和结束，但是需要是一条一直往下走的路线。也就是说，路径上的节点的层级是逐个递增的。

样例

**样例1：**

```plain
输入:
{1,2,3,4,#,2}
6
输出:
[[2, 4],[1, 3, 2]]
解释:
这棵二叉树如图所示：
    1
   / \
  2   3
 /   /
4   2
对于给定目标值6, 很显然有 2 + 4 = 6 和 1 + 3 + 2 = 6 两条路径。
```

**样例2：**

```plain
输入:
{1,2,3,4}
10
输出:
[]
解释:
这棵二叉树如图所示：
    1
   / \
  2   3
 /   
4   
对于给定目标值10, 没有符合条件的答案。
```

## 思路解析

这道题目是典型的[[08_二叉树的递归套路]]的题目，思路如下：

1. 假设以X节点为头，假设可以向X左树和X右树要任何信息
2. **在上一步的假设下，讨论以X为头节点的树，得到答案的可能性**（最重要,常见的分类是与X有关，与X无关）
3. 列出所有可能性后，确定到底需要向左树和右树要什么样的信息
4. 把左树信息和右树信息求全集，就是任何一棵子树都需要返回的信息S
5. 递归函数都返回S，每一棵子树都这么要求
6. 写代码，在代码中考虑如何把左树的信息和右树信息整合出整棵树的信息

结合本题目，我们可以知道：

1. 与X有关，从X出发，向下走，判断是否等于要求的target
2. 与X无关，不要X的值，向下走，判断是否等于要求的target
3. 注意：存在可能重复的值，因此在走完左右子树后，要删除已经走完的路径。

```JAVA
public class Solution {
    /**
     * @param root: the root of binary tree
     * @param target: An integer
     * @return: all valid paths
     *          we will sort your return value in output
     */
    public List<List<Integer>> binaryTreePathSum2(TreeNode root, int target) {
        // write your code here
        List<List<Integer>> ans = new ArrayList<>();
        process(root, target, new ArrayList<Integer>(), ans);
        return ans;
    }

      public void process(TreeNode node, int target, List<Integer> path, List<List<Integer>> ans) {
        if (node != null) {
            path.add(node.val);
            int sum = 0;
            for (int i = path.size() - 1; i >= 0; i--) {
                sum += path.get(i);
                if(sum == target) {
                    ans.add(new ArrayList<Integer>(path.subList(i, path.size())));
                }
            }
            process(node.left, target, path, ans);
            process(node.right, target, path, ans);
            path.remove(path.size() - 1);
        }
    }
}
```

## 总结

一定要回顾二叉树的递归套路，理解之后，很多都是触类旁通。
