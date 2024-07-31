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

## 994.腐烂的橘子

在给定的 `m x n` 网格 `grid` 中，每个单元格可以有以下三个值之一：

- 值 `0` 代表空单元格；
- 值 `1` 代表新鲜橘子；
- 值 `2` 代表腐烂的橘子。

每分钟，腐烂的橘子 **周围 4 个方向上相邻** 的新鲜橘子都会腐烂。

返回 *直到单元格中没有新鲜橘子为止所必须经过的最小分钟数。如果不可能，返回 `-1`* 。

 

**示例 1：**

**![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2019/02/16/oranges.png)**

```
输入：grid = [[2,1,1],[1,1,0],[0,1,1]]
输出：4
```

### 思路

1. 值 4 代表新腐烂的橘子
2. 先将腐烂橘子的周围进行感染，赋值4，当前2-橘子变为0-橘子，避免反复感染
3. 遍历一次，找到剩余的新鲜橘子，并将4-橘子变为2-橘子
4. 当新鲜橘子数量不在改变，停止感染，分情况返回结果

### 代码实现

~~~java
public class OrangesRotting {
    int[][] around = {{1, 0}, {0, 1}, {-1, 0}, {0, -1}};

    public int orangesRotting(int[][] grid) {
        //值 0 代表空单元格；
        //值 1 代表新鲜橘子；
        //值 2 代表腐烂的橘子。
        //值 4 代表新腐烂的橘子。

        int m = grid.length;
        int n = grid[0].length;
        int minutes = 1;
        int goodOrangeOld = getGoodOranges(grid, m, n);
        int goodOrangeNew = rotting(grid, m, n);
        while (goodOrangeNew != goodOrangeOld) {
            goodOrangeOld = goodOrangeNew;
            goodOrangeNew = rotting(grid, m, n);
            minutes++;
        }
        minutes--;
        if (goodOrangeNew == 0) return minutes;
        else return -1;

    }

    public int rotting(int[][] grid, int m, int n) {
        int goodOrange = 0;
        int x;
        int y;
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                if (grid[i][j] == 2) {
                    for (int k = 0; k < 4; k++) {
                        x = around[k][0] + i;
                        y = around[k][1] + j;
                        //腐烂橘子感染周围新鲜句子
                        if (x >= 0 && x < m && y >= 0 && y < n && grid[x][y] == 1) {
                            grid[x][y] = 4;
                        }
                    }
                    grid[i][j] = 0;
                }
            }
        }
        goodOrange = getGoodOranges(grid, m, n);

        return goodOrange;
    }

    public int getGoodOranges(int[][] grid, int m, int n) {
        int goodOrange = 0;
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                if (grid[i][j] == 1) {
                    goodOrange++;
                }
                if (grid[i][j] == 4) {//将新腐烂的橘子转化为腐烂橘子
                    grid[i][j] = 2;
                }
            }
        }
        return goodOrange;
    }
}
~~~

## 230.二叉搜索树中第K小的元素

给定一个二叉搜索树的根节点 `root` ，和一个整数 `k` ，请你设计一个算法查找其中第 `k` 小的元素（从 1 开始计数）。

**示例 1：**

![img](https://assets.leetcode.com/uploads/2021/01/28/kthtree1.jpg)

```
输入：root = [3,1,4,null,2], k = 1
输出：1
```

### 思路

中序遍历，用list存储值，中序遍历完毕后再次获取第k个值。

### 代码实现

~~~java
public class KthSmallest {
    List<Integer> list;

    public int kthSmallest(TreeNode root, int k) {
        list = new ArrayList<>();
        midOrder(root);
        return list.get(k - 1);
    }

    public void midOrder(TreeNode root) {
        if (root == null) return;
        midOrder(root.left);
        list.add(root.val);
        midOrder(root.right);
    }
}
~~~

## 114.二叉树展开为链表

给你二叉树的根结点 `root` ，请你将它展开为一个单链表：

- 展开后的单链表应该同样使用 `TreeNode` ，其中 `right` 子指针指向链表中下一个结点，而左子指针始终为 `null` 。
- 展开后的单链表应该与二叉树 [**先序遍历**](https://baike.baidu.com/item/先序遍历/6442839?fr=aladdin) 顺序相同。

 

**示例 1：**

![img](https://assets.leetcode.com/uploads/2021/01/14/flaten.jpg)

```
输入：root = [1,2,5,3,4,null,6]
输出：[1,null,2,null,3,null,4,null,5,null,6]
```

### 思路

先序遍历，然后将节点存储到队列中，遍历队列，前一个节点的右指针指向队列的开头位置，左指针指向null，直到队列为空

### 代码实现

~~~java
public class FlattenTrees {
    Deque<TreeNode> queue;

    public void flatten(TreeNode root) {
        if (root == null) return;
        queue = new LinkedList<>();
        preOrder(root);
        TreeNode cur;
        while (!queue.isEmpty()) {
            cur = queue.poll();
            if (!queue.isEmpty()) {
                cur.right = queue.peek();
                cur.left = null;
            }
        }
    }

    public void preOrder(TreeNode root) {
        if (root == null) return;
        queue.offer(root);
        preOrder(root.left);
        preOrder(root.right);
    }
}
~~~

## 207.课程表

你这个学期必须选修 `numCourses` 门课程，记为 `0` 到 `numCourses - 1` 。

在选修某些课程之前需要一些先修课程。 先修课程按数组 `prerequisites` 给出，其中 `prerequisites[i] = [ai, bi]` ，表示如果要学习课程 `ai` 则 **必须** 先学习课程 `bi` 。

- 例如，先修课程对 `[0, 1]` 表示：想要学习课程 `0` ，你需要先完成课程 `1` 。

请你判断是否可能完成所有课程的学习？如果可以，返回 `true` ；否则，返回 `false` 。

**示例 2：**

```
输入：numCourses = 2, prerequisites = [[1,0],[0,1]]
输出：false
解释：总共有 2 门课程。学习课程 1 之前，你需要先完成课程 0 ；并且学习课程 0 之前，你还应先完成课程 1 。这是不可能的。
```

### 思路

拓朴排序+广度优先算法

找出入度为0的点，作为前置课程，如果当前节点的邻接点入度减去1后变为0，将邻接点作为放入队列中，删除当前节点，课程数量减一。

### 代码实现

~~~java
public class CanFinishCourses {
    public boolean canFinish(int numCourses, int[][] prerequisites) {
        List<List<Integer>> adj = new ArrayList<>();//邻接表
        int[] inDegrees = new int[numCourses];
        Deque<Integer> queue = new LinkedList<>();
        for (int i = 0; i < numCourses; i++) {//邻接表初始化
            adj.add(new ArrayList<>());
        }
        for (int[] pq : prerequisites) {
            inDegrees[pq[0]]++;
            adj.get(pq[1]).add(pq[0]);
        }

        for (int i = 0; i < numCourses; i++) {
            if (inDegrees[i] == 0) queue.offer(i);//初始入度为零的进入队列中
        }

        while (!queue.isEmpty()) {
            int cur = queue.poll();
            numCourses--;
            for (int next : adj.get(cur)) {
                if (--inDegrees[next] == 0) queue.offer(next);
            }
        }
        return numCourses == 0;
    }
}
~~~

