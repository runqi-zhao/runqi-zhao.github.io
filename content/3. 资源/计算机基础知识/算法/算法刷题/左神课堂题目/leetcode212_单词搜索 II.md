---
type: knowledge
status: active
moc:
  - "[[计算机基础 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---
# 212. 单词搜索 II
这是 LeetCode 上的 **[212. 单词搜索 II](https://leetcode.cn/problems/word-search-ii/description/)** ，难度为 **困难**。
对应左神训练营第4期第1节第四题。

标签：[[05_前缀树]],[[11_暴力递归]]，[[12_动态规划]]

## 题目描述

给定一个 `m x n` 二维字符网格 `board` 和一个单词（字符串）列表 `words`， *返回所有二维网格上的单词* 。

单词必须按照字母顺序，通过 **相邻的单元格** 内的字母构成，其中“相邻”单元格是那些水平相邻或垂直相邻的单元格。同一个单元格内的字母在一个单词中不允许被重复使用。

**示例 1：**

![img](https://img-1312072469.cos.ap-nanjing.myqcloud.com/search1.jpg)

```ini
输入：board = [["o","a","a","n"],["e","t","a","e"],["i","h","k","r"],["i","f","l","v"]], words = ["oath","pea","eat","rain"]
输出：["eat","oath"]
```

**示例 2：**

![img](https://img-1312072469.cos.ap-nanjing.myqcloud.com/search2.jpg)

```ini
输入：board = [["a","b"],["c","d"]], words = ["abcb"]
输出：[]
```

**提示：**

- `m == board.length`
- `n == board[i].length`
- `1 <= m, n <= 12`
- `board[i][j]` 是一个小写英文字母
- `1 <= words.length <= 3 * 104`
- `1 <= words[i].length <= 10`
- `words[i]` 由小写英文字母组成
- `words` 中的所有字符串互不相同

## 题目解析

这是一道前缀树题目，回顾[[05_前缀树]]的规则

1. 单个字符串中，字符从前到后的加到一棵多叉树上
2. 字符放在路上，节点上有专属的数据项（常见的是pass和end值）
3. 所有样本都这样添加，如果没有路就新建，如有路就复用
4. 沿途节点的pass值增加1，每个字符串结束时来到的节点end值增加1

基于前缀树的思路，此题目可以这么解：

1. 以word数组构建前缀树；
2. 当构建好前缀树之后，遍历数组；
   1. 从某个位置的字符出发，之前的路径上，走过的字符，记录在一个集合里；
   2. 当前节点还没有登上，有待检查能不能登上去的前缀树的节点；
   3. 如果找到单词中的某个字符，记录在集合中；
   4. 返回找到了几个字符，当我来到某个位置，通过end值判断是否搞定了一个位置的值。
3. 将其中每次最终位置的字符串保存在一个集合中，进行返回

```java
public class WordSearch {
    public static class TrieNode {
        public TrieNode[] nexts;
        public int pass;
        public int end;

        public TrieNode() {
            nexts = new TrieNode[26];
            pass = 0;
            end = 0;
        }

    }

    public static void fillWord(TrieNode head, String word) {
        head.pass++;
        char[] chs = word.toCharArray();
        int index = 0;
        TrieNode node = head;
        for (int i = 0; i < chs.length; i++) {
            index = chs[i] - 'a';
            if (node.nexts[index] == null) {
                node.nexts[index] = new TrieNode();
            }
            node = node.nexts[index];
            node.pass++;
        }
        node.end++;
    }

    public static String generatePath(LinkedList<Character> path) {
        char[] str = new char[path.size()];
        int index = 0;
        for (Character cha : path) {
            str[index++] = cha;
        }
        return String.valueOf(str);
    }

    public static List<String> findWords(char[][] board, String[] words) {
        TrieNode head = new TrieNode(); // 前缀树最顶端的头
        HashSet<String> set = new HashSet<>();
        for (String word : words) {
            if (!set.contains(word)) {
                fillWord(head, word);
                set.add(word);
            }
        }
        // 答案
        List<String> ans = new ArrayList<>();
        // 沿途走过的字符，收集起来，存在path里
        LinkedList<Character> path = new LinkedList<>();
        for (int row = 0; row < board.length; row++) {
            for (int col = 0; col < board[0].length; col++) {
                // 枚举在board中的所有位置
                // 每一个位置出发的情况下，答案都收集
                process(board, row, col, path, head, ans);
            }
        }
        return ans;
    }

    // 从board[row][col]位置的字符出发，
    // 之前的路径上，走过的字符，记录在path里
    // cur还没有登上，有待检查能不能登上去的前缀树的节点
    // 如果找到words中的某个str，就记录在 res里
    // 返回值，从row,col 出发，一共找到了多少个str
    public static int process(
            char[][] board,
            int row, int col,
            LinkedList<Character> path,
            TrieNode cur,
            List<String> res) {
        char cha = board[row][col];
        if (cha == 0) { // 这个row col位置是之前走过的位置
            return 0;
        }
        // (row,col) 不是回头路   cha 有效

        int index = cha - 'a';
        // 如果没路，或者这条路上最终的字符串之前加入过结果里
        if (cur.nexts[index] == null || cur.nexts[index].pass == 0) {
            return 0;
        }
        // 没有走回头路且能登上去
        cur = cur.nexts[index];
        path.addLast(cha);// 当前位置的字符加到路径里去
        int fix = 0; // 从row和col位置出发，后续一共搞定了多少答案
        // 当我来到row col位置，如果决定不往后走了。是不是已经搞定了某个字符串了
        if (cur.end > 0) {
            res.add(generatePath(path));
            cur.end--;
            fix++;
        }
        // 往上、下、左、右，四个方向尝试
        board[row][col] = 0;
        if (row > 0) {
            fix += process(board, row - 1, col, path, cur, res);
        }
        if (row < board.length - 1) {
            fix += process(board, row + 1, col, path, cur, res);
        }
        if (col > 0) {
            fix += process(board, row, col - 1, path, cur, res);
        }
        if (col < board[0].length - 1) {
            fix += process(board, row, col + 1, path, cur, res);
        }
        board[row][col] = cha;
        path.pollLast();
        cur.pass -= fix;
        return fix;
    }
}
```

