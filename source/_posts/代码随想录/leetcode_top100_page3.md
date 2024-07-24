---
title: leetcode_top100_page3
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

## 240.搜索二维矩阵2

编写一个高效的算法来搜索 `*m* x *n*` 矩阵 `matrix` 中的一个目标值 `target` 。该矩阵具有以下特性：

- 每行的元素从左到右升序排列。
- 每列的元素从上到下升序排列。

**示例 1：**

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2020/11/25/searchgrid2.jpg)

```
输入：matrix = [[1,4,7,11,15],[2,5,8,12,19],[3,6,9,16,22],[10,13,14,17,24],[18,21,23,26,30]], target = 5
输出：true
```

### 思路1-错误

通过target确定所在粗暴确定行列

```java
while (row < m && matrix[row][0] <= target) {
        row++;
    }
    row = row > 0 ? row - 1 : 0;
```

会出现target在row和col里面的情况

### 思路2

就是要选择一个点，使得横向和纵向移动，martix能够有不同的变化

左上角：往右往下移动，martix值都变大，无法区分，不可用

**右上角：往左martix变小，往下martix变大，可区分，可用**

**左下角：往右martix变大，往上martix变小，可区分，可用**

右下角：往左往上移动，martix都变小，不可区分，不可用

### 代码实现

~~~java
public class SearchMatrix {
    public boolean searchMatrix(int[][] matrix, int target) {
        int m = matrix.length;
        int n = matrix[0].length;
        int row = 0;
        int col = n - 1;
        while (col >= 0 && row < m) {
            if (matrix[row][col] > target) col--;
            else if (matrix[row][col] < target) row++;
            else {
                return true;
            }
        }
        return false;
    }
}
~~~

## 138.随机链表的复制

给你一个长度为 `n` 的链表，每个节点包含一个额外增加的随机指针 `random` ，该指针可以指向链表中的任何节点或空节点。

构造这个链表的 **[深拷贝](https://baike.baidu.com/item/深拷贝/22785317?fr=aladdin)**。 深拷贝应该正好由 `n` 个 **全新** 节点组成，其中每个新节点的值都设为其对应的原节点的值。新节点的 `next` 指针和 `random` 指针也都应指向复制链表中的新节点，并使原链表和复制链表中的这些指针能够表示相同的链表状态。**复制链表中的指针都不应指向原链表中的节点** 。

例如，如果原链表中有 `X` 和 `Y` 两个节点，其中 `X.random --> Y` 。那么在复制链表中对应的两个节点 `x` 和 `y` ，同样有 `x.random --> y` 。

返回复制链表的头节点。

用一个由 `n` 个节点组成的链表来表示输入/输出中的链表。每个节点用一个 `[val, random_index]` 表示：

- `val`：一个表示 `Node.val` 的整数。
- `random_index`：随机指针指向的节点索引（范围从 `0` 到 `n-1`）；如果不指向任何节点，则为 `null` 。

你的代码 **只** 接受原链表的头节点 `head` 作为传入参数。

 

**示例 1：**

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2020/01/09/e1.png)

```
输入：head = [[7,null],[13,0],[11,4],[10,2],[1,0]]
输出：[[7,null],[13,0],[11,4],[10,2],[1,0]]
```

### 思路

考略到random指向可前可后，因此先将next全部赋值，在对random一一赋值。

### 代码实现

~~~java
public Node copyRandomList(Node head) {
    Node cur = head;
    Node copyList = new Node(0);
    Node copyCur = copyList;

    //对复制链表next进行赋值
    while (cur != null) {
        Node curNode = new Node(cur.val);
        copyCur.next = curNode;

        //一次节点处理完毕
        cur = cur.next;
        copyCur = copyCur.next;
    }

    //对复制链表random进行赋值
    cur = head;
    copyCur = copyList.next;
    while (cur != null) {
        Node randomP = head;
        Node randomCopyP = copyList.next;
        if (cur.random != null) {
            while (randomP != cur.random) {
                randomP = randomP.next;
                randomCopyP = randomCopyP.next;
            }
            copyCur.random = randomCopyP;
        }
        cur = cur.next;
        copyCur = copyCur.next;
    }
    return copyList.next;
}
~~~

### 思路2

也可以用map来存储新的节点，即使value的值一样，但是node的地址不同。

### 代码实现

~~~java
public Node copyRandomList(Node head) {
    Map<Node,Node> map = new HashMap<>();
    Node cur = head;
    while(cur != null){
        map.put(cur, new Node(cur.val));
        cur = cur.next;
    }
    cur = head;
    while(cur != null){
        map.get(cur).next = map.get(cur.next);
        map.get(cur).random = map.get(cur.random);
        cur = cur.next;
    }
    return map.get(head);
}
~~~

## 148.排序链表

给你链表的头结点head，请将其按升序排列并返回排序后的链表。

**示例 1：**

![img](https://assets.leetcode.com/uploads/2020/09/14/sort_list_1.jpg)

```
输入：head = [4,2,1,3]
输出：[1,2,3,4]
```

### 思路1

优先级队列实现

### 代码实现

~~~java
public ListNode sortList(ListNode head) {
    PriorityQueue<Integer> queue = new PriorityQueue<>();
    while (head != null) {
        queue.add(head.val);
        head = head.next;
    }
    ListNode res = new ListNode(0);
    ListNode cur = res;
    while (!queue.isEmpty()) {
        cur.next = new ListNode(queue.poll());
        cur = cur.next;
    }
    return res.next;

}
~~~

## 543.二叉树的直径

给你一棵二叉树的根节点，返回该树的 **直径** 。

二叉树的 **直径** 是指树中任意两个节点之间最长路径的 **长度** 。这条路径可能经过也可能不经过根节点 `root` 。

两节点之间路径的 **长度** 由它们之间边数表示。

 

**示例 1：**

![img](https://assets.leetcode.com/uploads/2021/03/06/diamtree.jpg)

```
输入：root = [1,2,3,4,5]
输出：3
解释：3 ，取路径 [4,2,1,3] 或 [5,2,1,3] 的长度。
```

### 思路

不能直接求左右两边的深度然后直接相加，因为可能存在左右两边的子树不平衡，左边子树深度1，右边子树深度1000；

正确思路：每个节点下的最长路径都要找出来。

### 代码实现

~~~java
public class DiameterOfBinaryTree {
    int maxPath;

    public int diameterOfBinaryTree(TreeNode root) {
        preOrder(root);
        return maxPath;
    }

    public int getDepth(TreeNode root) {
        if (root == null) return 0;
        else return Math.max(getDepth(root.left), getDepth(root.right)) + 1;
    }

    public void preOrder(TreeNode root) {
        if (root == null) return;
        int left = getDepth(root.left);
        int right = getDepth(root.right);
        maxPath = Math.max(left + right, maxPath);

        preOrder(root.left);
        preOrder(root.right);
    }
}
~~~

### 优化

其实getDepth中恶意直接包含前序遍历的操作，这样不必重复遍历

### 代码实现

~~~java
public class DiameterOfBinaryTree {
    int maxPath;

    public int diameterOfBinaryTree(TreeNode root) {
        preOrder(root);
        return maxPath;
    }


    public int preOrder(TreeNode root) {
        if (root == null) return 0;
        int left = preOrder(root.left);
        int right = preOrder(root.right);
        maxPath = Math.max(left + right, maxPath);

        return Math.max(left, right) + 1;

    }
}
~~~

## 146.LRU缓存

请你设计并实现一个满足 [LRU (最近最少使用) 缓存](https://baike.baidu.com/item/LRU) 约束的数据结构。

实现 `LRUCache` 类：

- `LRUCache(int capacity)` 以 **正整数** 作为容量 `capacity` 初始化 LRU 缓存
- `int get(int key)` 如果关键字 `key` 存在于缓存中，则返回关键字的值，否则返回 `-1` 。
- `void put(int key, int value)` 如果关键字 `key` 已经存在，则变更其数据值 `value` ；如果不存在，则向缓存中插入该组 `key-value` 。如果插入操作导致关键字数量超过 `capacity` ，则应该 **逐出** 最久未使用的关键字。

函数 `get` 和 `put` 必须以 `O(1)` 的平均时间复杂度运行。

### 思路

用两个map来记录value和年龄，每次读取和put都需要增加年龄

但是增加年龄会有一个遍历就十分耗时

### 代码实现（19/22）

~~~java
public class LRUCache {
    Map<Integer, Integer> score;
    int capacity;

    Map<Integer, Integer> cache;

    /**
     * - `LRUCache(int capacity)` 以 **正整数** 作为容量 `capacity` 初始化 LRU 缓存
     *
     * @param capacity
     */
    public LRUCache(int capacity) {
        this.capacity = capacity;
        score = new HashMap<>();
        cache = new HashMap<>();
    }

    /**
     * 如果关键字 `key` 存在于缓存中，则返回关键字的值，否则返回 `-1` 。
     *
     * @param key
     * @return
     */
    public int get(int key) {
        addTime();
        if (cache.containsKey(key)) {
            score.put(key, 0);
            return cache.get(key);
        } else return -1;


    }

    /**
     * 如果关键字 `key` 已经存在，则变更其数据值 `value` ；
     * 如果不存在，则向缓存中插入该组 `key-value` 。
     * 如果插入操作导致关键字数量超过 `capacity` ，则应该 **逐出** 最久未使用的关键字。
     *
     * @param key
     * @param value
     */
    public void put(int key, int value) {
        addTime();
        score.put(key, 0);
        cache.put(key, value);
        if (cache.size() > capacity) {
            removeCache();
        }
    }

    public void removeCache() {
        int maxKey = 0;//无意义，提示报错
        int maxScore = 0;
        for (Map.Entry<Integer, Integer> entry : cache.entrySet()) {
            if (score.get(entry.getKey()) > maxScore) {
                maxScore = score.get(entry.getKey());
                maxKey = entry.getKey();
            }
        }
        cache.remove(maxKey);
        score.remove(maxKey);
    }

    public void addTime() {
        for (Map.Entry<Integer, Integer> entry : cache.entrySet()) {
            score.put(entry.getKey(), score.get(entry.getKey()) + 1);
        }
    }
}
~~~

### 优化方法

用一个双指针链表维护（key，value），节点的结构如下：

~~~java
class DLinkedNode {
    int key;
    int value;
    DLinkedNode prev;
    DLinkedNode next;
    public DLinkedNode() {}
    public DLinkedNode(int _key, int _value) {key = _key; value = _value;}
}
~~~

用map存储（key，DLinkedNode），map的作用是快速定位到某一个节点，方便删除节点，前后指针指向改变就行。

get和put操作将节点放到head处，超过容量通过tail删除节点。

### 代码实现

自己手写代码很长，继承`LinkedHashMap`代码如下：

~~~java
public class LRUCache extends LinkedHashMap<Integer, Integer> {
    private int capacity;

    public LRUCache(int capacity) {
        super(capacity, 0.75f, true);
        this.capacity = capacity;
    }

    public int get(int key) {
        return super.getOrDefault(key, -1);
    }

    public void put(int key, int value) {
        super.put(key, value);
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry eldest) {
        return size() > capacity;
    }
}
~~~

