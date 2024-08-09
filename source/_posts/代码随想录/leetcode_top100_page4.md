---
title: leetcode_top100_page4
categories:
- leetcode
tags:
- top100
---
<meta name="referrer" content="no-referrer"/>

# 内容

**本节包括：**

- leetcode的top一百的题

<!--more-->

## 208. 实现 Trie (前缀树)

**[Trie](https://baike.baidu.com/item/字典树/9825209?fr=aladdin)**（发音类似 "try"）或者说 **前缀树** 是一种树形数据结构，用于高效地存储和检索字符串数据集中的键。这一数据结构有相当多的应用情景，例如自动补完和拼写检查。

### 思路1

暴力法，将word都存入列表中，然后进行匹配和搜索，速度很慢

> contains（）：用于检查字符串是否包含指定的子字符串。它返回一个布尔值，表示子字符串是否在字符串中出现过。

### 代码实现1

略

### 思路2

定义前缀树节点，每个节点包含26个子节点，和结尾标志

~~~java
class TrieNode {
    TrieNode[] subNode;
    boolean isEnd;

    public TrieNode() {
        subNode = new TrieNode[26];
        isEnd = false;
    }
}
~~~

插入时不断取char字符，计算对应索引，并将对应索引的子节点分情况初始话

> 通用的可以用map存储节点

### 代码实现2

~~~java
public class Trie {
    TrieNode rootNode;

    public Trie() {
        rootNode = new TrieNode();
    }

    public void insert(String word) {
        TrieNode node = rootNode;
        for (int i = 0; i < word.length(); i++) {
            char c = word.charAt(i);
            int idx = c - 'a';
            if (node.subNode[idx] == null) {
                node.subNode[idx] = new TrieNode();
            }
            node = node.subNode[idx];
        }
        node.isEnd = true;

    }

    public boolean search(String word) {
        TrieNode node = searchPrefix(word);
        return node != null && node.isEnd;
    }

    public boolean startsWith(String prefix) {
        TrieNode node = searchPrefix(prefix);
        return node != null;
    }

    private TrieNode searchPrefix(String word) {
        TrieNode node = rootNode;
        for (int i = 0; i < word.length(); i++) {
            char c = word.charAt(i);
            int idx = c - 'a';
            node = node.subNode[idx];
            if (node == null) return null;
        }
        return node;
    }


}

class TrieNode {
    TrieNode[] subNode;
    boolean isEnd;

    public TrieNode() {
        subNode = new TrieNode[26];
        isEnd = false;
    }
}
~~~

## 79.单词搜索

给定一个 `m x n` 二维字符网格 `board` 和一个字符串单词 `word` 。如果 `word` 存在于网格中，返回 `true` ；否则，返回 `false` 。

单词必须按照字母顺序，通过相邻的单元格内的字母构成，其中“相邻”单元格是那些水平相邻或垂直相邻的单元格。同一个单元格内的字母不允许被重复使用。

 

**示例 1：**

![img](https://assets.leetcode.com/uploads/2020/11/04/word2.jpg)

```
输入：board = [["A","B","C","E"],["S","F","C","S"],["A","D","E","E"]], word = "ABCCED"
输出：true
```

### 思路

用一个二维数组记录已经使用的字母，当前的字母与word对应索引的字符相同时，就要使用这个字母，遍历完当前字母周围的字母后才将当前字母标记为未使用。结束条件是当前idx等于单词word的长度减一，否则越界了。

### 代码实现

~~~java
int[][] directions = {{1, 0}, {0, 1}, {-1, 0}, {0, -1}};
boolean res = false;

public boolean exist(char[][] board, String word) {
    int m = board.length;
    int n = board[0].length;
    boolean[][] used = new boolean[m][n];
    for (int i = 0; i < m; i++) {
        for (int j = 0; j < n; j++) {
            backTracing(board, used, i, j, 0, word);
        }
    }
    return res;

}

public void backTracing(char[][] board, boolean[][] used, int x, int y, int idx, String word) {
    if (idx == word.length() - 1) {
        if (board[x][y] == word.charAt(idx)) {
            res = true;
        }
    }
    if (res) {
        return;
    }
    if (board[x][y] == word.charAt(idx)) {
        used[x][y] = true;
        for (int k = 0; k < 4; k++) {
            int newX = x + directions[k][0];
            int newY = y + directions[k][1];
            if (newX >= 0 && newX < board.length && newY >= 0 && newY < board[0].length && !used[newX][newY]) {
                backTracing(board, used, newX, newY, idx + 1, word);
            }
        }
        used[x][y] = false;
    }
}
~~~

## 437.路径总和3

给定一个二叉树的根节点 `root` ，和一个整数 `targetSum` ，求该二叉树里节点值之和等于 `targetSum` 的 **路径** 的数目。

**路径** 不需要从根节点开始，也不需要在叶子节点结束，但是路径方向必须是向下的（只能从父节点到子节点）。

 

**示例 1：**

![img](https://assets.leetcode.com/uploads/2021/04/09/pathsum3-1-tree.jpg)

```
输入：root = [10,5,-3,3,2,null,11,3,-2,null,1], targetSum = 8
输出：3
解释：和等于 8 的路径有 3 条，如图所示。
```

### 思路

前序遍历+dfs

### 代码实现

~~~java
public class PathSum {

    public int pathSum(TreeNode root, int targetSum) {
        int count = 0;
        if (root == null) return 0;
        count += dfs(root, targetSum, 0);
        count += pathSum(root.left, targetSum);
        count += pathSum(root.right, targetSum);
        return count;
    }

    /**
     * 寻找以当前节点为起始节点的路径和为target的结果数
     */
    public int dfs(TreeNode root, long targetSum, long curSum) {
        if (root == null) {
            return 0;
        }
        int res = 0;
        if (curSum + (long) root.val == targetSum) res++;

        res += dfs(root.left, targetSum, curSum + root.val);
        res += dfs(root.right, targetSum, curSum + root.val);
        return res;
    }
}
~~~

## 代码2-前缀和

~~~java
public int pathSum(TreeNode root, int targetSum) {
    Map<Long, Integer> prefix = new HashMap<Long, Integer>();
    prefix.put(0L, 1);
    return dfs(root, prefix, 0, targetSum);
}

public int dfs(TreeNode root, Map<Long, Integer> prefix, long curr, int targetSum) {
    if (root == null) {
        return 0;
    }

    int ret = 0;
    curr += root.val;

    ret = prefix.getOrDefault(curr - targetSum, 0);
    prefix.put(curr, prefix.getOrDefault(curr, 0) + 1);
    ret += dfs(root.left, prefix, curr, targetSum);
    ret += dfs(root.right, prefix, curr, targetSum);
    prefix.put(curr, prefix.getOrDefault(curr, 0) - 1);

    return ret;
}
~~~

## 22.括号生成

数字 `n` 代表生成括号的对数，请你设计一个函数，用于能够生成所有可能的并且 **有效的** 括号组合。

**示例 1：**

```
输入：n = 3
输出：["((()))","(()())","(())()","()(())","()()()"]
```

### 思路

用栈来验证产生的括号的有效性，用回溯来产生各种括号组合，可以采用剪枝的方式来减少没有必要的运算。

~~~java
//剪枝
if (sb.length() > n) {
    if (leftParenthesisNUms > n || sb.length() - leftParenthesisNUms > n) return;
}
~~~

### 代码实现

~~~java
public class GenerateParenthesis {
    List<String> res;

    public List<String> generateParenthesis(int n) {
        res = new ArrayList<>();
        StringBuilder stringBuilder = new StringBuilder();
        backTracing(stringBuilder, 0, n);
        return res;
    }

    public boolean validParenthesis(StringBuilder sb) {
        Deque<Character> stack = new LinkedList<>();
        char[] array = sb.toString().toCharArray();
        for (int i = 0; i < array.length; i++) {
            if (array[i] == '(') stack.push('(');
            else {
                if (!stack.isEmpty() && stack.peek() == '(') {
                    stack.pop();
                } else return false;
            }
        }
        return true;
    }

    public void backTracing(StringBuilder sb, int leftParenthesisNUms, int n) {
        if (sb.length() == n * 2 && leftParenthesisNUms == n) {
            if (validParenthesis(sb)) {
                res.add(new String(sb));
            }
            return;
        }
        //剪枝
        if (sb.length() > n) {
            if (leftParenthesisNUms > n || sb.length() - leftParenthesisNUms > n) return;
        }

        sb.append('(');
        backTracing(sb, leftParenthesisNUms + 1, n);
        sb.deleteCharAt(sb.length() - 1);


        sb.append(')');
        backTracing(sb, leftParenthesisNUms, n);
        sb.deleteCharAt(sb.length() - 1);
    }

    public static void main(String[] args) {
        GenerateParenthesis generate = new GenerateParenthesis();
        System.out.println(generate.generateParenthesis(3));
    }
}
~~~

### 优化剪枝

左括号不能超过3，当右括号比左括号少时添加右括号

### 代码

~~~java
List<String> res;

public List<String> generateParenthesis(int n) {
    res = new ArrayList<>();
    StringBuilder stringBuilder = new StringBuilder();
    backTracing(stringBuilder, 0, n);
    return res;
}

public void backTracing(StringBuilder sb, int leftParenthesisNUms, int n) {
    if (sb.length() == n * 2) {
        res.add(new String(sb));
        return;
    }
    if (leftParenthesisNUms < n) {
        sb.append('(');
        backTracing(sb, leftParenthesisNUms + 1, n);
        sb.deleteCharAt(sb.length() - 1);
    }
    if (sb.length() - leftParenthesisNUms < leftParenthesisNUms) {
        sb.append(')');
        backTracing(sb, leftParenthesisNUms, n);
        sb.deleteCharAt(sb.length() - 1);
    }
}
~~~

## 35.搜索插入位置

给定一个排序数组和一个目标值，在数组中找到目标值，并返回其索引。如果目标值不存在于数组中，返回它将会被按顺序插入的位置。

请必须使用时间复杂度为 `O(log n)` 的算法。

**示例 1:**

```
输入: nums = [1,3,5,6], target = 5
输出: 2
```

### 思路

有序数组的二分查找

### 代码实现

~~~java
public int searchInsert(int[] nums, int target) {
    int left = 0;
    int right = nums.length - 1;
    int mid = (left + right) / 2;
    while (left <= right && nums[mid] != target) {
        if (nums[mid] > target) {
            right = mid - 1;
        } else {
            left = mid + 1;
        }
        mid = (left + right) / 2;
    }
    if (nums[mid] == target) return mid;
    else return left;
}
~~~

## 74.搜素二维矩阵

给你一个满足下述两条属性的 `m x n` 整数矩阵：

- 每行中的整数从左到右按非严格递增顺序排列。
- 每行的第一个整数大于前一行的最后一个整数。

给你一个整数 `target` ，如果 `target` 在矩阵中，返回 `true` ；否则，返回 `false` 。

 

**示例 1：**

![img](https://assets.leetcode.com/uploads/2020/10/05/mat.jpg)

```
输入：matrix = [[1,3,5,7],[10,11,16,20],[23,30,34,60]], target = 3
输出：true
```

### 思路

限定为行的位置，如果行的位置小于0，那么直接返回false

再定位列的位置，返回`matrix[rowPos][colPos] == target`

### 代码实现

~~~java
public boolean searchMatrix(int[][] matrix, int target) {
    int m = matrix.length;
    int n = matrix[0].length;
    int[] colNums = new int[m];
    for (int i = 0; i < m; i++) {
        colNums[i] = matrix[i][0];
    }
    //定位target可能出现的行位置
    int rowPos = rowSearch(colNums, target);
    if (rowPos < 0) return false;
    if (matrix[rowPos][0] == target) return true;

    int colPos = rowSearch(matrix[rowPos], target);
    return matrix[rowPos][colPos] == target;

}

public int rowSearch(int[] nums, int target) {
    int left = 0;
    int right = nums.length - 1;
    int mid = (left + right) / 2;
    while (left <= right && nums[mid] != target) {
        if (nums[mid] > target) {
            right = mid - 1;
        } else {
            left = mid + 1;
        }
        mid = (left + right) / 2;
    }
    if (nums[mid] == target) return mid;
    else return left - 1;
}
~~~

## 34.在排序数组中查找元素的第一个和最后一个位置

给你一个按照非递减顺序排列的整数数组 `nums`，和一个目标值 `target`。请你找出给定目标值在数组中的开始位置和结束位置。

如果数组中不存在目标值 `target`，返回 `[-1, -1]`。

你必须设计并实现时间复杂度为 `O(log n)` 的算法解决此问题。

### 思路

二分搜素找到mid位置，如果等于target，那么向左右两边找到边界

注意边界的值的获取，left要判断前一个位置相等时，在减一

```
while (left > 0 && nums[left - 1] == target) {
    left--;
}
```

### 代码实现

~~~java
package binarysearch;

public class SearchRange {

    public int[] searchRange(int[] nums, int target) {
        int[] posRange = new int[2];
        int pos = rowSearch(nums, target);
        if (pos < 0) return new int[]{-1, -1};
        if (nums[pos] == target) {
            int left = pos, right = pos;
            while (left > 0 && nums[left - 1] == target) {
                left--;
            }
            while (right < nums.length - 1 && nums[right + 1] == target) {
                right++;
            }
            return new int[]{left, right};
        } else return new int[]{-1, -1};
    }

    public int rowSearch(int[] nums, int target) {
        if (nums.length == 0) return -1;
        int left = 0;
        int right = nums.length - 1;
        int mid = (left + right) / 2;
        while (left <= right && nums[mid] != target) {
            if (nums[mid] > target) {
                right = mid - 1;
            } else {
                left = mid + 1;
            }
            mid = (left + right) / 2;
        }
        if (nums[mid] == target) return mid;
        else return left - 1;
    }
}
~~~

