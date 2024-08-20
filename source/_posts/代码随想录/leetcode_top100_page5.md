---
title: leetcode_top100_page5
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

## 155.最小栈

设计一个支持 `push` ，`pop` ，`top` 操作，并能在常数时间内检索到最小元素的栈。

实现 `MinStack` 类:

- `MinStack()` 初始化堆栈对象。
- `void push(int val)` 将元素val推入堆栈。
- `void pop()` 删除堆栈顶部的元素。
- `int top()` 获取堆栈顶部的元素。
- `int getMin()` 获取堆栈中的最小元素。

 

**示例 1:**

```
输入：
["MinStack","push","push","push","getMin","pop","top","getMin"]
[[],[-2],[0],[-3],[],[],[],[]]

输出：
[null,null,null,null,-3,null,0,-2]
```

### 思路

- 定义两个栈，一个存储元素，一个存储最小值，两个栈的空间一直一样大

- 用一个最小栈记录每次压入一个元素时，当前的最小值，初始为空时，直接压入当前元素

### 代码实现

~~~java
class MinStack {
    Deque<Integer> stack;
    Deque<Integer> minStack;

    public MinStack() {
        stack = new LinkedList<>();
        minStack = new LinkedList<>();
    }

    public void push(int val) {
        stack.push(val);
        if (minStack.isEmpty()) minStack.push(val);
        else if (val > minStack.peek()) {
            minStack.push(minStack.peek());
        } else {
            minStack.push(val);
        }
    }

    public void pop() {
        stack.pop();
        minStack.pop();
    }

    public int top() {
        return stack.peek();
    }

    public int getMin() {
        return minStack.peek();
    }
}
~~~

## 394.字符串解码

给定一个经过编码的字符串，返回它解码后的字符串。

编码规则为: `k[encoded_string]`，表示其中方括号内部的 `encoded_string` 正好重复 `k` 次。注意 `k` 保证为正整数。

你可以认为输入字符串总是有效的；输入字符串中没有额外的空格，且输入的方括号总是符合格式要求的。

此外，你可以认为原始数据不包含数字，所有的数字只表示重复的次数 `k` ，例如不会出现像 `3a` 或 `2[4]` 的输入。

**示例 1：**

```
输入：s = "3[a]2[bc]"
输出："aaabcbc"
```

### 思路

算法流程：

```
构建辅助栈 stack， 遍历字符串 s 中每个字符 c；
    当 c 为数字时，将数字字符转化为数字 multi，用于后续倍数计算；
    当 c 为字母时，在 res 尾部添加 c；
    当 c 为 [ 时，将当前 multi 和 res 入栈，并分别置空置 0：
        记录此 [ 前的临时结果 res 至栈，用于发现对应 ] 后的拼接操作；
        记录此 [ 前的倍数 multi 至栈，用于发现对应 ] 后，获取 multi × [...] 字符串。
        进入到新 [ 后，res 和 multi 重新记录。
    当 c 为 ] 时，stack 出栈，拼接字符串 res = last_res + cur_multi * res，其中:
        last_res是上个 [ 到当前 [ 的字符串，例如 "3[a2[c]]" 中的 a；
        cur_multi是当前 [ 到 ] 内字符串的重复倍数，例如 "3[a2[c]]" 中的 2。
返回字符串 res。
```

**==感谢作者：Krahets==**:

### 代码实现

~~~java
public String decodeString(String s) {
    LinkedList<Integer> multiStack = new LinkedList<>();
    LinkedList<String> resStack = new LinkedList<>();
    StringBuilder res = new StringBuilder();
    int multi = 0;
    for (char c : s.toCharArray()) {
        if (c == '[') {
            multiStack.add(multi);
            resStack.add(res.toString());
            multi = 0;
            res = new StringBuilder();
        } else if (c == ']') {
            StringBuilder tmp = new StringBuilder();
            int curMulti = multiStack.removeLast();
            for (int i = 0; i < curMulti; i++) {
                tmp.append(res);
            }
            res = new StringBuilder(resStack.removeLast() + tmp);
        } else if (c >= '0' && c <= '9') {
            multi = multi * 10 + Integer.parseInt(String.valueOf(c));
        } else {
            res.append(c);
        }
    }
    return res.toString();
}
~~~

## 215.数组中的第K个最大元素

给定整数数组 `nums` 和整数 `k`，请返回数组中第 `k` 个最大的元素。

请注意，你需要找的是数组排序后的第 `k` 个最大的元素，而不是第 `k` 个不同的元素。

你必须设计并实现时间复杂度为 `O(n)` 的算法解决此问题。

 

**示例 1:**

```
输入: [3,2,1,5,6,4], k = 2
输出: 5
```

### 思路1-2

1. 排序不符合O（n），但是速度最快？
2. 构建大小固定为k的小根堆，只有当前元素大于小根堆的最小元素才更新小根堆
   用到了优先级队列

### 代码实现

~~~java
public int findKthLargest(int[] nums, int k) {
    PriorityQueue<Integer> queue = new PriorityQueue<>();
    for (int i = 0; i < nums.length; i++) {
        if (queue.size() < k) {
            queue.offer(nums[i]);
        } else if (queue.peek() >= nums[i]) {
            continue;
        } else {
            queue.poll();
            queue.offer(nums[i]);
        }
    }
    return queue.poll();
}
~~~

### 优化

~~~java
public int findKthLargest(int[] nums, int k) {
    PriorityQueue<Integer> heap = new PriorityQueue<>();
    for (int num : nums) {
        heap.add(num);
        if (heap.size() > k) {
            heap.poll();
        }
    }
    return heap.peek();
}
~~~

## 152.乘积最大子数组

给你一个整数数组 `nums` ，请你找出数组中乘积最大的非空连续

子数组

（该子数组中至少包含一个数字），并返回该子数组所对应的乘积。

测试用例的答案是一个 **32-位** 整数。

### 思路

```
标签：动态规划
遍历数组时计算当前最大值，不断更新
令imax为当前最大值，则当前最大值为 imax = max(imax * nums[i], nums[i])
由于存在负数，那么会导致最大的变最小的，最小的变最大的。因此还需要维护当前最小值imin，imin = min(imin * nums[i], nums[i])
当负数出现时则imax与imin进行交换再进行下一步计算
时间复杂度：O(n)
```

==作者：画手大鹏==

### 代码实现

~~~java
public int maxProduct(int[] nums) {
    int max = Integer.MIN_VALUE;
    int imax = 1;
    int imin = 1;
    for (int i = 0; i < nums.length; i++) {
        if (nums[i] < 0) {
            int tmp = imax;
            imax = imin;
            imin = tmp;
        }
        imax = Math.max(imax * nums[i], nums[i]);
        imin = Math.min(imin * nums[i], nums[i]);
        max = Math.max(imax, max);
    }
    return max;
}
~~~

## 118.杨辉三角

给定一个非负整数 *`numRows`，*生成「杨辉三角」的前 *`numRows`* 行。

在「杨辉三角」中，每个数是它左上方和右上方的数的和。

![img](https://pic.leetcode-cn.com/1626927345-DZmfxB-PascalTriangleAnimated2.gif)

 

**示例 1:**

```
输入: numRows = 5
输出: [[1],[1,1],[1,2,1],[1,3,3,1],[1,4,6,4,1]]
```

### 思路

按照规律取值相加即可

### 代码实现

~~~java
public List<List<Integer>> generate(int numRows) {
    List<List<Integer>> resList = new ArrayList<>();
    List<Integer> list0 = new ArrayList<>();
    list0.add(1);

    if (numRows == 1) {
        resList.add(list0);
    } else {
        resList.add(list0);
        for (int i = 1; i < numRows; i++) {
            List<Integer> curList = new ArrayList<>();
            curList.add(1);
            int curValue;
            for (int j = 1; j < i; j++) {
                curValue = resList.get(i - 1).get(j - 1) + resList.get(i - 1).get(j);
                curList.add(curValue);
            }
            curList.add(1);
            resList.add(curList);
        }
    }
    return resList;
}
~~~

## 64.最小路径和

给定一个包含非负整数的 `*m* x *n*` 网格 `grid` ，请找出一条从左上角到右下角的路径，使得路径上的数字总和为最小。

**说明：**每次只能向下或者向右移动一步。

**示例 1：**

![img](https://assets.leetcode.com/uploads/2020/11/05/minpath.jpg)

```
输入：grid = [[1,3,1],[1,5,1],[4,2,1]]
输出：7
解释：因为路径 1→3→1→1→1 的总和最小。
```

### 思路

定义一个二维动态规划数组，记录最短路径

初始化第一行和第一列，之后按照状态转移方程逐一填充

### 代码实现

~~~java
public int minPathSum(int[][] grid) {
    int m = grid.length;
    int n = grid[0].length;
    int[][] dp = new int[m + 1][n + 1];
    dp[0][0] = grid[0][0];
    for (int i = 1; i < m; i++) {//初始化列
        dp[i][0] = dp[i - 1][0] + grid[i][0];
    }
    for (int i = 1; i < n; i++) {//初始化行
        dp[0][i] = dp[0][i - 1] + grid[0][i];
    }
    for (int i = 1; i < m; i++) {
        for (int j = 1; j < n; j++) {
            dp[i][j] = Math.min(dp[i][j - 1], dp[i - 1][j]) + grid[i][j];
        }
    }
    return dp[m - 1][n - 1];
}
~~~

## 5.最长回文子串

给你一个字符串 `s`，找到 `s` 中最长的回文子串。

**示例 1：**

```
输入：s = "babad"
输出："bab"
解释："aba" 同样是符合题意的答案。
```

### 思路

二维数组动态规划

~~~java
 int[][] dp = new int[s.length()][s.length()];//dp[i][j]表示以[i,j]子字符串是回文的最长长度
~~~

状态转移方程，分三种情况：

1. `j - i >= 2`

```java
dp[i][j] = dp[i + 1][j - 1] + 2;
```

2. `j - i == 1`

~~~java
dp[i][j] = 2
~~~

3. `j==i`

~~~java
dp[i][i] = 1;
~~~

每次赋值后，记录最大长度和最长字符串

### 注意

> 由于有一个状态转移方程是`dp[i][j] = dp[i + 1][j - 1] + 2;`，因此`i`因该从大到小，`j`应该从小到大

### 代码实现

~~~java
public String longestPalindrome(String s) {
    if (s.length() == 1) return s;
    int[][] dp = new int[s.length()][s.length()];//dp[i][j]表示以[i,j]子字符串是回文的最长长度
    String resStr = s.substring(0, 1);
    int maxLen = 1;

    for (int i = s.length() - 1; i >= 0; i--) {
        for (int j = i; j < s.length(); j++) {
            boolean isEqual = (s.charAt(i) == s.charAt(j));
            if (j - i >= 2) {
                if (isEqual && dp[i + 1][j - 1] > 0) {
                    dp[i][j] = dp[i + 1][j - 1] + 2;
                }
            } else if (j - i == 1) {
                if (isEqual) dp[i][j] = 2;
            } else {
                dp[i][i] = 1;
            }

            if (dp[i][j] > 0 && j - i + 1 > maxLen) {
                maxLen = j - i + 1;
                resStr = s.substring(i, j + 1);
            }
        }
    }
    return resStr;
}
~~~

