---
type: knowledge
status: active
moc:
  - "[[计算机基础 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---
# 174.地下城游戏

这是 LeetCode上的 **[174.地下城游戏](https://leetcode.cn/problems/dungeon-game/description/)** ，难度为 **困难**。

对应左神训练营第4期第2节第四题。

标签：[[12_动态规划]],[[11_暴力递归]]

## 题目描述

一些恶魔抓住了公主（**P**）并将她关在了地下城的右下角。地下城是由 M x N 个房间组成的二维网格。我们英勇的骑士（**K**）最初被安置在左上角的房间里，他必须穿过地下城并通过对抗恶魔来拯救公主。

骑士的初始健康点数为一个正整数。如果他的健康点数在某一时刻降至 0 或以下，他会立即死亡。

有些房间由恶魔守卫，因此骑士在进入这些房间时会失去健康点数（若房间里的值为*负整数*，则表示骑士将损失健康点数）；其他房间要么是空的（房间里的值为 *0*），要么包含增加骑士健康点数的魔法球（若房间里的值为*正整数*，则表示骑士将增加健康点数）。

为了尽快到达公主，骑士决定每次只向右或向下移动一步。

**编写一个函数来计算确保骑士能够拯救到公主所需的最低初始健康点数。**

例如，考虑到如下布局的地下城，如果骑士遵循最佳路径 `右 -> 右 -> 下 -> 下`，则骑士的初始健康点数至少为 **7**。

| -2 (K) | -3   | 3      |
| ------ | ---- | ------ |
| -5     | -10  | 1      |
| 10     | 30   | -5 (P) |

**说明:**

- 骑士的健康点数没有上限。
- 任何房间都可能对骑士的健康点数造成威胁，也可能增加骑士的健康点数，包括骑士进入的左上角房间以及公主被监禁的右下角房间。

## 思路解析

### 思路一

采用深度递归，从每个位置开始，判断向右走或者向下走所需要的血量，分为以下三种情况：

1. 来到最后一行：
   1. 此时`map[i][j] < 0`，那么就需要将向右走所需血量最小值与已经走的值进行相加，进行返回；
   2. 此时`map[i][j] >= 0 && map[i][j] > 向右走所需血量最小值`，直接返回1，代表不需要的血量，进行返回；
   3. 其余情况，直接将`向右走所需血量最小值-map[i][j]`，进行返回；
2. 来到最后一列
   1. 此时`map[i][j] < 0`，那么就需要将这个值与已经走的值进行相加，进行返回；
   2. 此时`map[i][j] >= 0 && map[i][j] > 向下走所需血量最小值`，直接返回1，代表不需要的血量，进行返回；
   3. 其余情况，直接将`向下走所需血量最小值-map[i][j]`，进行返回；
3. 其余情况
   1. 选择出向右或者向下走的最小值；
   2. 此时`map[i][j] < 0`，那么就需要将最小值值进行相加，进行返回；
   3. 此时`map[i][j] >= 0 && map[i][j] > 最小值`，直接返回1，代表不需要的血量，进行返回；
   4. 其余情况，直接将`最小值-map[i][j]`，进行返回；

```java
public static int needMin(int[][] matrix) {
		return process(matrix, matrix.length, matrix[0].length, 0, 0);
	}

	// 来到了matrix[row][col]，还没登上去，到达右下角，返回至少的初始血量
	public static int process(int[][] matrix, int N, int M, int row, int col) {
		if(row == N - 1 && col == M - 1) { // 已经达到右下角了
			return matrix[N-1][M-1] < 0 ? (-matrix[N-1][M-1] + 1) : 1;
		}
		if(row == N - 1) {
			int rightNeed = process(matrix, N, M, row, col+1);
			if(matrix[row][col] < 0) { // 3    -7    10
				return -matrix[row][col] + rightNeed;
			}else if(matrix[row][col] >= rightNeed) {  // 3    3    1
				return 1;
			}else { //  3   1    2
				return rightNeed - matrix[row][col];
			}
		}
		if(col == M - 1) {
			int downNeed = process(matrix, N, M, row+1, col);
			if(matrix[row][col] < 0) { // 3    -7    10
				return -matrix[row][col] + downNeed;
			}else if(matrix[row][col] >= downNeed) {  // 3    3    1
				return 1;
			}else { //  3   1    2
				return downNeed - matrix[row][col];
			}
		}
		int minNextNeed = Math.min(process(matrix, N, M, row, col+1), process(matrix, N, M, row+1, col));
		if(matrix[row][col] < 0) { // 3    -7    10
			return -matrix[row][col] + minNextNeed;
		}else if(matrix[row][col] >= minNextNeed) {  // 3    3    1
			return 1;
		}else { //  3   1    2
			return minNextNeed - matrix[row][col];
		}
	}
```

### 思路二

首先生成和map一样大的矩阵dp，`dp[i][j]`表示如果骑士要走上位置（i，j），并且从该位置选择一条最优的路径，所需要的血量最少是多少。本题相当于是求`dp[0][0]`的值，所以，采用倒着填表的方式便可以得到最终结果。`dp[i][j]`的计算方式如下：

1. 矩阵的右下角的位置是骑士到达的最后位置，骑士经过这里的时候只要血量不少于1即可，如果该位置是血瓶，即`map[2][2] >= 0`，则`dp[2][2] == 1`即可，否则`dp[2][2] == -map[2][2] + 1`。
2. 矩阵的最后一列，表示骑士只能向下走，所以此时只要满足在当前位置（i，col-1） (col表示矩阵的列数)加上血或者扣完血之后的血量等于`dp[i+1][col-1]`即可。
3. 矩阵的最后一行，表示骑士只能向右走，只要满足在当前位置（row-1，j）(row表示矩阵的行数)加上血或者扣完血之后的血量等于`dp[row-1][j+1]` 即可。
4. 骑士在矩阵的其他位置，都有向右或者向下两种选择，只要选择需要血量最少的一个即可。

```java
public static int minHP1(int[][] matrix) {
        if (matrix == null || matrix.length == 0 || matrix[0] == null || matrix[0].length == 0) {
            return 1;
        }
        int row = matrix.length;
        int col = matrix[0].length;
        int[][] dp = new int[row--][col--];
        dp[row][col] = matrix[row][col] < 0 ? (-matrix[row][col] + 1) : 1;
        for (int j = col - 1; j >= 0; j--) {
            dp[row][j] = Math.max(dp[row][j + 1] - matrix[row][j], 1);
        }
        int right = 0;
        int down = 0;
        for (int i = row - 1; i >= 0; i--) {
            dp[i][col] = Math.max(dp[i + 1][col] - matrix[i][col], 1);
            for (int j = col - 1; j >= 0; j--) {
                right = Math.max(dp[i][j + 1] - matrix[i][j], 1);
                down = Math.max(dp[i + 1][j] - matrix[i][j], 1);
                dp[i][j] = Math.min(right, down);
            }
        }
        return dp[0][0];
    }
```

此时时间复杂度为$O(M \times N)$，空间复杂度为$O(M \times N)$

## 总结
此题目对应的模型是多样本位置全对应的尝试模型，特点如下：

- 此题目无法从初始位置进行判断，因为你并不知道向右走与向下走的之后，下一个位置向右走或者向下走的路径和，因此，采用从结束位置进行出发，选择出所需要血量最少的位置；
- 需要考虑来到最后一行或者最后一列的情况。

## 相关题目

 [[leetcod64_最小路径和]]

[[leetcode62_不同路径]]
[[牛客_最大路径和]]