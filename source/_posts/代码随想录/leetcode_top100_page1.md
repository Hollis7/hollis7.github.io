---
title: leetcode_top100_page1
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

## 49.字母异位词分组

给你一个字符串数组，请你将 **字母异位词** 组合在一起。可以按任意顺序返回结果列表。

**字母异位词** 是由重新排列源单词的所有字母得到的一个新单词。

### 自己思路

对每个String字符串进行abc排序，然后用排序后的字符串作为map的key，排序前的字符串作为List<String>中的value，同样的key加入新的str后进行更新，未出现的key或者map为空直接加入当前key和value；

### 代码实现

~~~java
public class GroupAnagrams {
    public List<List<String>> groupAnagrams(String[] strs) {
        List<List<String>> res = new ArrayList<>();
        Map<String, List<String>> map = new HashMap<>();
        for (int i = 0; i < strs.length; i++) {
            String str = strs[i];
            String sortedStr = sortStr(str);
            if (map.isEmpty() || !map.containsKey(sortedStr)) {
                List<String> element = new ArrayList<>();
                element.add(str);
                map.put(sortedStr, element);
            } else {
                List<String> oldListStr = new ArrayList<>(map.get(sortedStr));
                oldListStr.add(str);
                map.put(sortedStr, oldListStr);
            }
        }
        for (String s : map.keySet()) {
            res.add(map.get(s));
        }
        return res;
    }

    public String sortStr(String str) {
        char[] charArray = str.toCharArray();
        Arrays.sort(charArray);
        return Arrays.toString(charArray);
    }

    public static void main(String[] args) {
        GroupAnagrams groupAnagrams = new GroupAnagrams();
        String[] strs = {"eat", "tea", "tan", "ate", "nat", "bat"};
        List<List<String>> lists = groupAnagrams.groupAnagrams(strs);
        for (List<String> list : lists) {
            System.out.println(list);
        }
    }
}

~~~

可以优化

~~~java
for (int i = 0; i < strs.length; i++) {
        String str = strs[i];
        String sortedStr = sortStr(str);
        List<String> list = map.getOrDefault(sortedStr, new ArrayList<>());
        list.add(str);
        map.put(sortedStr, list);
    }
~~~

## 128.最长连续序列

给定一个未排序的整数数组 nums，找出数字连续的最长序列（不要求序列元素在原数组中连续）的长度。
请你设计并实现时间复杂度为0(n）的算法解决此问题。

**示例 1：**

```
输入：nums = [100,4,200,1,3,2]
输出：4
解释：最长数字连续序列是 [1, 2, 3, 4]。它的长度为 4。
```

### 思路1

排序然后计算最长序列长度

### 代码实现1

~~~java
public int longestConsecutive(int[] nums) {
    Arrays.sort(nums);
    int count = 1;
    int maxLen = 1;
    if (nums.length == 0) return 0;
    for (int i = 1; i < nums.length; i++) {
        if (nums[i] - nums[i - 1] == 1) {
            count++;
            maxLen = Math.max(maxLen, count);
        } else if (nums[i] == nums[i - 1]) {
            continue;
        } else {//未能连续
            count = 1;
        }
    }
    return maxLen;
}
~~~

### 思路2

用一个hash集合存储所有元素，判断集合中是否存在当前num的num+1值，如果有，长度加一。当然，连续数字的第一个数字需要确认，只要不存在num-1即可。

### 代码实现

~~~java
class Solution {
    public int longestConsecutive(int[] nums) {
        Set<Integer> num_set = new HashSet<Integer>();
        for (int num : nums) {
            num_set.add(num);
        }

        int longestStreak = 0;

        for (int num : num_set) {
            if (!num_set.contains(num - 1)) {
                int currentNum = num;
                int currentStreak = 1;

                while (num_set.contains(currentNum + 1)) {
                    currentNum += 1;
                    currentStreak += 1;
                }

                longestStreak = Math.max(longestStreak, currentStreak);
            }
        }

        return longestStreak;
    }
}
~~~

## 283.移动零

给定一个数组 `nums`，编写一个函数将所有 `0` 移动到数组的末尾，同时保持非零元素的相对顺序。

**请注意** ，必须在不复制数组的情况下原地对数组进行操作。

```
输入: nums = [0,1,0,3,12]
输出: [1,3,12,0,0]
```

### 思路

双指针，从前往后遍历，快指针负责判断是否为零，慢指针负责赋值，快指针到达数组末尾后，慢指针之后的值填充0

### 代码实现

~~~java
 public void moveZeroes(int[] nums) {
        int size = nums.length;
        int fast, low;
        for (fast = 0, low = 0; fast < size; fast++) {
            if (nums[fast] != 0) {
                nums[low] = nums[fast];
                low++;
            }
        }
        for (; low < size; low++) {
            nums[low] = 0;
        }
    }
~~~

## 11.盛最多水的容器

给定一个长度为 `n` 的整数数组 `height` 。有 `n` 条垂线，第 `i` 条线的两个端点是 `(i, 0)` 和 `(i, height[i])` 。

找出其中的两条线，使得它们与 `x` 轴共同构成的容器可以容纳最多的水。

返回容器可以储存的最大水量。

**说明：**你不能倾斜容器。

**示例 1：**

![img](https://aliyun-lc-upload.oss-cn-hangzhou.aliyuncs.com/aliyun-lc-upload/uploads/2018/07/25/question_11.jpg)

```
输入：[1,8,6,2,5,4,8,3,7]
输出：49 
解释：图中垂直线代表输入数组 [1,8,6,2,5,4,8,3,7]。在此情况下，容器能够容纳水（表示为蓝色部分）的最大值为 49。
```

### 思路1

第一反应是单调栈，但是给出的实例看出不应该用单调栈

暴力法，计算每两根柱子之间的水量

### 代码实现1

~~~java
 public int maxArea(int[] height) {
        int width = 0;
        int tmpHeight = 0;
        int area = 0;
        for (int i = 0; i < height.length - 1; i++) {
            for (int j = i + 1; j < height.length; j++) {
                tmpHeight = Math.min(height[i], height[j]);
                width = j - i;
                area = Math.max(area, tmpHeight * width);
            }
        }
        return area;
    }
~~~

> 会出现超时

### 思路2

双指针

左右指针每次移动较小的那个

### 代码实现2

~~~java
 public int maxArea(int[] height) {
        int l = 0, r = height.length - 1;
        int area = 0;
        int minHeight = 0;
        while (l < r) {
            minHeight = Math.min(height[l], height[r]);
            area = Math.max(area, (r - l) * minHeight);
            if (minHeight == height[l]) l++;
            else r--;
        }
        return area;
    }
~~~

## 3.无重复字符的最长子串

给定一个字符串 `s` ，请你找出其中不含有重复字符的 **最长子串**的长度。

**示例 1:**

```
输入: s = "abcabcbb"
输出: 3 
解释: 因为无重复字符的最长子串是 "abc"，所以其长度为 3。
```

**示例 2:**

```
输入: s = "bbbbb"
输出: 1
解释: 因为无重复字符的最长子串是 "b"，所以其长度为 1。
```

### 思路1

用map存储当前字符串的使用情况和位置，当有重复字符串时，窗口开始位置start变为重复字符加一处，窜口结束位置在start+1，否则存储进map中。

### 代码实现1

~~~java
public int lengthOfLongestSubstring(String s) {
    int maxLen = 0;
    Map<Character, Integer> map = new HashMap();
    int i = 0, j = 0;
    while (j < s.length()) {
        if (map.containsKey(s.charAt(j))) {
            maxLen = Math.max(maxLen, j - i);
            i = map.get(s.charAt(j)) + 1;
            map.clear();
            map.put(s.charAt(i), i);
            j = i + 1;
        } else {
            map.put(s.charAt(j), j);
            j++;
        }
    }
    maxLen = Math.max(maxLen, j - i);
    return maxLen;
}
~~~

### 思路2

使用hashmap太耗时间，因此第二种方法：**采用数组**保存字符串位置

ascii码一共只有128个

### 代码实现2

~~~java
public int lengthOfLongestSubstring(String s) {
        int maxLen = 0;
        int[] postion = new int[128];
        Arrays.fill(postion, -1);
        int l = 0;
        for (int i = 0; i < s.length(); i++) {
            l = Math.max(l, postion[s.charAt(i)] + 1);
            maxLen = Math.max(maxLen, i - l + 1);
            postion[s.charAt(i)] = i;
        }
        return maxLen;
    }
~~~

## 438.找到字符串中所有字母异位词

给定两个字符串 `s` 和 `p`，找到 `s` 中所有 `p` 的 **异位词** 的子串，返回这些子串的起始索引。不考虑答案输出的顺序。

**异位词** 指由相同字母重排列形成的字符串（包括相同的字符串）。

**示例 1:**

```
输入: s = "cbaebabacd", p = "abc"
输出: [0,6]
解释:
起始索引等于 0 的子串是 "cba", 它是 "abc" 的异位词。
起始索引等于 6 的子串是 "bac", 它是 "abc" 的异位词。
```

### 思路1

暴力法，用一个数组记录当前子字符串的各个字符次数，比对p的次数，需要用到两个for循环

### 代码实现1

~~~java
public List<Integer> findAnagrams(String s, String p) {
    List<Integer> res = new ArrayList<>();
    //记录p的次数
    for (int i = 0; i < p.length(); i++) {
        countP[p.charAt(i) - 'a']++;
    }

    for (int i = 0; i < s.length(); i++) {
        int[] curNums = new int[26];
        for (int j = i; j < i + p.length() && j < s.length(); j++) {
            curNums[s.charAt(j) - 'a']++;
            if (curNums[s.charAt(j) - 'a'] > countP[s.charAt(j) - 'a']) {
                break;
            }
        }
        if (isAnagrams(countP, curNums)) res.add(i);

    }
    return res;
}

public boolean isAnagrams(int[] nums, int[] curNums) {
    for (int i = 0; i < nums.length; i++) {
        if (nums[i] != curNums[i]) {
            return false;
        }
    }
    return true;
}
~~~

### 思路2

滑动窗口，滑动窗口的大小的固定为p的长度，每次只需要更新当前窗口起始位置和结束位置后一位的字符的次数就行，比较s和p的字符次数是否相同。

同时可以用`Arrays.equals(countS, countP)`比较两个数组是否相同

不需要每次都更新s的全部次数套两个for循环

### 代码实现2

~~~java
public List<Integer> findAnagrams(String s, String p) {
    int[] countP = new int[26];
    int[] countS = new int[26];
    List<Integer> res = new ArrayList<>();
    if (s.length() < p.length()) return res;
    //记录p的次数
    for (int i = 0; i < p.length(); i++) {
        countP[p.charAt(i) - 'a']++;
        countS[s.charAt(i) - 'a']++;
    }
    if (Arrays.equals(countP, countS)) res.add(0);

    for (int i = 0; i < s.length() - p.length(); i++) {
        countS[s.charAt(i) - 'a']--;
        countS[s.charAt(i + p.length()) - 'a']++;
        if (Arrays.equals(countS, countP)) res.add(i + 1);
    }
    return res;
}
~~~

## 560. 和为 K 的子数组

给你一个整数数组 `nums` 和一个整数 `k` ，请你统计并返回 *该数组中和为 `k` 的子数组的个数* 。

子数组是数组中元素的连续非空序列。

### 思路

前缀表和hash结合

我们定义 pre[i] 为 [0..i] 里所有数的和，则 pre[i] 可以由 pre[i−1] 递推而来，即：

pre[i]=pre[i−1]+nums[i]
那么「[j..i] 这个子数组和为 k 」这个条件我们可以转化为

pre[i]−pre[j−1]==k

简单移项可得符合条件的下标 *j* 需要满足

*pre*[*j*−1]==*pre*[*i*]−k

**所以我们考虑以 *i* 结尾的和为 *k* 的连续子数组个数时只要统计有多少个前缀和为 *pre*[*i*]−*k* 的 *pre*[*j*] 即可。**

### 代码实现

~~~java
public int subarraySum(int[] nums, int k) {
    int count = 0, pre = 0;
    Map<Integer, Integer> map = new HashMap<>();
    map.put(0, 1);
    for (int i = 0; i < nums.length; i++) {
        pre += nums[i];
        if (map.containsKey(pre - k)) {
            count += map.get(pre - k);
        }
        map.put(pre, map.getOrDefault(pre, 0) + 1);
    }
    return count;
}
~~~

## 160.相交链表

给你两个单链表的头节点 `headA` 和 `headB` ，请你找出并返回两个单链表相交的起始节点。如果两个链表不存在相交节点，返回 `null` 。

图示两个链表在节点 `c1` 开始相交**：**

[![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/12/14/160_statement.png)](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/12/14/160_statement.png)

题目数据 **保证** 整个链式结构中不存在环。

**注意**，函数返回结果后，链表必须 **保持其原始结构** 。

### 思路

求取链表长度差后，分别用两个指针指向A和B，长的先走差值步数后开始比较

### 代码实现

~~~java
public class GetIntersectionNode {
    public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
        int aLen = getLength(headA);
        int bLen = getLength(headB);
        int shorterLen = Math.min(aLen, bLen);
        int skip;
        ListNode pa = headA;
        ListNode pb = headB;
        if (aLen == shorterLen) {
            skip = bLen - shorterLen;
            while (skip > 0) {
                pb = pb.next;
                skip--;
            }
        } else {
            skip = aLen - shorterLen;
            while (skip > 0) {
                pa = pa.next;
                skip--;
            }
        }

        while (pa != pb && pa != null && pb != null) {
            pa = pa.next;
            pb = pb.next;
        }
        return pa;
    }

    public int getLength(ListNode list) {
        int len = 0;
        while (list != null) {
            len++;
            list = list.next;
        }
        return len;
    }
}
~~~

### 思路2

指针 pA 指向 A 链表，指针 pB 指向 B 链表，依次往后遍历
如果 pA 到了末尾，则 pA = headB 继续遍历
如果 pB 到了末尾，则 pB = headA 继续遍历
比较长的链表指针指向较短链表head时，长度差就消除了
如此，只需要将最短链表遍历两次即可找到位置

> 通俗来讲，两个链表的长度固定，走到末尾然后走对方的路路程一样，如果有交点，终点前的公共路段一定会相遇，因为他们会同时到达终点。

### 代码实现2

~~~java
public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
    if (headA == null || headB == null) return null;
    ListNode pA = headA, pB = headB;
    while (pA != pB) {
        pA = pA == null ? headB : pA.next;
        pB = pB == null ? headA : pB.next;
    }
    return pA;
}
~~~

