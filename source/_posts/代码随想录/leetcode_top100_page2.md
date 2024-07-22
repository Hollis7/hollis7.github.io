---
title: leetcode_top100_page2
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

## 189.轮转数组

给定一个整数数组 `nums`，将数组中的元素向右轮转 `k` 个位置，其中 `k` 是非负数。

**示例 1:**

```
输入: nums = [1,2,3,4,5,6,7], k = 3
输出: [5,6,7,1,2,3,4]
解释:
向右轮转 1 步: [7,1,2,3,4,5,6]
向右轮转 2 步: [6,7,1,2,3,4,5]
向右轮转 3 步: [5,6,7,1,2,3,4]
```

### 思路

典型的右移处理，三次反转即可，先整体反转，再局部反转两次

### 代码实现

~~~java
public class Rotate {
    public void rotate(int[] nums, int k) {
        k = k % nums.length;
        reverseArrays(nums, 0, nums.length - 1);
        reverseArrays(nums, 0, k - 1);
        reverseArrays(nums, k, nums.length - 1);
    }

    public void reverseArrays(int[] nums, int start, int end) {
        int tmp;
        while (start < end) {
            tmp = nums[start];
            nums[start] = nums[end];
            nums[end] = tmp;
            start++;
            end--;
        }
    }
}
~~~

## 238.除自身以外数组的乘积

给你一个整数数组 `nums`，返回 数组 `answer` ，其中 `answer[i]` 等于 `nums` 中除 `nums[i]` 之外其余各元素的乘积 。

题目数据 **保证** 数组 `nums`之中任意元素的全部前缀元素和后缀的乘积都在 **32 位** 整数范围内。

请 **不要使用除法，**且在 `O(n)` 时间复杂度内完成此题。

**示例 1:**

```
输入: nums = [1,2,3,4]
输出: [24,12,8,6]
```

### 思路

~~~java
for (int i = 1; i < nums.length; i++) {
        ans[i] = ans[i - 1] / nums[i] * nums[i - 1];
    }
~~~

这种做法是错误的，没有考虑到`nums[i]`为0的情况

**正确思路**

分别求取i左边的乘积和i右边的乘积，那么ans[i]应该是`l[i]·r[i]`，同时`l[i]`和`r[i]`

可以在`O（n）`的时间复杂度计算。

还有注意 `l[0] = 1;`，`r[nums.length - 1] = 1;`，这是为了方便递推下去。

### 代码实现

~~~java
public int[] productExceptSelf(int[] nums) {
    int[] ans = new int[nums.length];
    int[] l = new int[nums.length];//i左边的乘积
    int[] r = new int[nums.length];//i右边的乘积
    l[0] = 1;
    for (int i = 1; i < nums.length; i++) {
        l[i] = l[i - 1] * nums[i - 1];
    }
    r[nums.length - 1] = 1;
    for (int i = nums.length - 2; i >= 0; i--) {
        r[i] = r[i + 1] * nums[i + 1];
    }
    for (int i = 0; i < nums.length; i++) {
        ans[i] = l[i] * r[i];
    }
    return ans;
}
~~~

## 73.矩阵置零

给定一个 `*m* x *n*` 的矩阵，如果一个元素为 **0** ，则将其所在行和列的所有元素都设为 **0** 。请使用 **[原地](http://baike.baidu.com/item/原地算法)** 算法。

### 思路

用一个函数来将`matrix[i][j] == 0`的位置行和列进行置零，但是要考虑到可能会导致之后遇到0但是不是矩阵原本0的情况，因此用一个数组`figures`来记录是不是原生0，是原生0，那么才执行操作，否则跳过；

注意，当前位置的行和列可能包含原生0，遇到了直接不做处理

### 代码实现

~~~java
public class SetZeroes {
    public void setZeroes(int[][] matrix) {
        int m = matrix.length;
        int n = matrix[0].length;
        int[][] figures = new int[m][n];//figures元素是1表示是被其他0值修改为0，不做更改
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                if (matrix[i][j] == 0 && figures[i][j] == 0) {
                    firstSetZero(matrix, figures, i, j, m, n);
                }
            }
        }
    }

    public void firstSetZero(int[][] matrix, int[][] figures, int i, int j, int m, int n) {
        for (int k = 0; k < m; k++) {
            if (matrix[k][j] != 0) {
                matrix[k][j] = 0;
                figures[k][j] = 1;
            }
        }
        for (int k = 0; k < n; k++) {
            if (matrix[i][k] != 0) {
                matrix[i][k] = 0;
                figures[i][k] = 1;
            }
        }
    }
}
~~~

### 优化思路2

可以对figures进行空间优化，可以先记录那些行和列可以被置零，难后才置零，这样只需要`（m+n）`的空间大小。

### 优化代码2

~~~java
public void setZeroes(int[][] matrix) {
    int m = matrix.length; //m行
    int n = matrix[0].length; //n列
    boolean[] row = new boolean[m];
    boolean[] col = new boolean[n];
    for(int i = 0 ; i < m ; i++){
        for(int j = 0 ; j < n ;j++){
            if(matrix[i][j]==0){
                row[i]=col[j]=true;
            }
        }
    }
     for(int i = 0 ; i < m ; i++){
        for(int j = 0 ; j < n ;j++){
            if(row[i] || col[j]){
                matrix[i][j]=0;
            }
        }
    }
}
~~~

## 54.螺旋矩阵

给你一个 `m` 行 `n` 列的矩阵 `matrix` ，请按照 **顺时针螺旋顺序** ，返回矩阵中的所有元素。

### 思路

**遍历的同时缩放边界**

根据题目示例 matrix = [[1,2,3],[4,5,6],[7,8,9]] 的对应输出 [1,2,3,6,9,8,7,4,5] 可以发现，顺时针打印矩阵的顺序是 “从左向右、从上向下、从右向左、从下向上” 循环。

因此，考虑设定矩阵的 “左、上、右、下” 四个边界，模拟以上矩阵遍历顺序。

==算法流程：==
空值处理： 当 matrix 为空时，直接返回空列表 [] 即可。
初始化： 矩阵 左、右、上、下 四个边界 l , r , t , b ，用于打印的结果列表 res 。
循环打印： “从左向右、从上向下、从右向左、从下向上” 四个方向循环打印。
	根据边界打印，即将元素按顺序添加至列表 res 尾部。
	边界向内收缩 1 （代表已被打印）。
判断边界是否相遇（是否打印完毕），若打印完毕则跳出。

### 代码实现

~~~java
public List<Integer> spiralOrder(int[][] matrix) {
    List<Integer> res = new ArrayList<>();
    int t = matrix.length - 1;
    int r = matrix[0].length - 1;
    int l = 0, b = 0;
    while (true) {
        for (int i = l; i <= r; i++) {
            res.add(matrix[b][i]);
        }
        if (++b > t) break;
        for (int i = b; i <= t; i++) {
            res.add(matrix[i][r]);
        }
        if (--r < l) break;
        for (int i = r; i >= l; i--) {
            res.add(matrix[t][i]);
        }
        if (--t < b) break;
        for (int i = t; i >= b; i--) {
            res.add(matrix[i][l]);
        }
        if (++l > r) break;
    }
    return res;

}
~~~

## 21.合并两个有序链表

将两个升序链表合并为一个新的 **升序** 链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。 

### 思路

就是一个比较和拼接的问题

但是要注意`mergeList`有个初始点的值是0，可以把他看作指向头节点的虚拟节点。

### 代码实现

~~~java
public ListNode mergeTwoLists(ListNode list1, ListNode list2) {
    ListNode mergeList = new ListNode();
    ListNode p = mergeList;
    while (list1 != null && list2 != null) {
        if (list1.val > list2.val) {
            p.next = new ListNode(list2.val);
            list2 = list2.next;
            p = p.next;
        } else {
            p.next = new ListNode(list1.val);
            list1 = list1.next;
            p = p.next;
        }
    }
    if (list1 != null) p.next = list1;
    else p.next = list2;
    return mergeList.next;
}
~~~

## 48.旋转图像

给定一个 *n* × *n* 的二维矩阵 `matrix` 表示一个图像。请你将图像顺时针旋转 90 度。

你必须在**[ 原地](https://baike.baidu.com/item/原地算法)** 旋转图像，这意味着你需要直接修改输入的二维矩阵。**请不要** 使用另一个矩阵来旋转图像。

**示例 1：**

![img](https://assets.leetcode.com/uploads/2020/08/28/mat1.jpg)

```
输入：matrix = [[1,2,3],[4,5,6],[7,8,9]]
输出：[[7,4,1],[8,5,2],[9,6,3]]
```

### 思路

这道题十分考验找规律，先将矩阵对角线反转，在按照行进行反转。

### 代码实现

~~~java
public class RotateMatrix {
    public void rotate(int[][] matrix) {
        diagonalReverse(matrix);
        rowReverse(matrix);
    }

    public void diagonalReverse(int[][] matrix) {
        int n = matrix.length;
        int tmp;
        for (int i = 0; i < n; i++) {
            for (int j = i; j < n; j++) {
                tmp = matrix[i][j];
                matrix[i][j] = matrix[j][i];
                matrix[j][i] = tmp;
            }
        }
    }

    public void rowReverse(int[][] matrix) {
        int n = matrix.length;
        int tmp;
        for (int i = 0; i < n; i++) {
            int l = 0;
            int r = n - 1;
            while (l < r) {
                tmp = matrix[i][r];
                matrix[i][r] = matrix[i][l];
                matrix[i][l] = tmp;
                l++;
                r--;
            }
        }
    }

}
~~~

## 2.两数相加

给你两个 **非空** 的链表，表示两个非负的整数。它们每位数字都是按照 **逆序** 的方式存储的，并且每个节点只能存储 **一位** 数字。

请你将两个数相加，并以相同形式返回一个表示和的链表。

你可以假设除了数字 0 之外，这两个数都不会以 0 开头。

**示例 1：**

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2021/01/02/addtwonumber1.jpg)

```
输入：l1 = [2,4,3], l2 = [5,6,4]
输出：[7,0,8]
解释：342 + 465 = 807.
```

### 思路1

先转化为数字，在相加，最后转化为列表

先由int型，再改为long型，还是溢出了，所以不能直接用这种思路

### 代码实现

~~~java
public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
    long num1 = listToNum(l1);
    long num2 = listToNum(l2);
    long res = num1 + num2;

    ListNode resList = new ListNode();
    numToList(res, resList);
    return resList.next;
}

public long listToNum(ListNode node) {
    long res = 0;
    long weight = 1;
    for (; node != null; node = node.next, weight *= 10) {
        res += node.val * weight;
    }
    return res;
}

public void numToList(long num, ListNode res) {
    if (num == 0) {
        res.next = new ListNode(0);
    }
    ListNode p = res;
    long tmp;
    while (num > 0) {
        tmp = num % 10;
        p.next = new ListNode((int) tmp);
        p = p.next;
        num = num / 10;
    }
}
~~~

### 思路2

将链表位置一一对应相加，有进位直接保存到下一次的计算结果中。

还要注意最后一个位置是否有进位，要单独判断。

### 代码实现

~~~java
public class AddTwoNumbers {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        ListNode resList = new ListNode();
        ListNode p = resList;
        int round = 0;
        while (l1 != null || l2 != null) {
            int num1 = l1 == null ? 0 : l1.val;
            int num2 = l2 == null ? 0 : l2.val;
            int curSum = num1 + num2 + round;
            round = curSum >= 10 ? 1 : 0;
            p.next = new ListNode(curSum % 10);
            p = p.next;
            l1 = l1 == null ? null : l1.next;
            l2 = l2 == null ? null : l2.next;
        }
        if (round > 0) {
            p.next = new ListNode(1);
        }
        return resList.next;
    }
}
~~~

