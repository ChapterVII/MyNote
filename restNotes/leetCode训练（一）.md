---
title: leetCode训练（一）
date: 2019-01-10 09:45:34
tags: hashTable 
---

## [Two Sum](https://leetcode.com/problems/two-sum/)

### 需求

输入一个不定项数字数组和一个值；输出数组中相加得这个值的两项的索引

### 例

twoSum([2, 7, 11, 15], 9)  => [0, 1]

### 答案

#### 方法一：最慢，内存占用最小

```
const twoSum = function(nums, target) {
    for (let i = 0; i < nums.length; i++) {
        const diff = target - nums[i];
        const index = nums.indexOf(diff, i + 1);
        if (index !== -1) {
            return [i, index];
        }
    }
    return [0, 0];
}
```

#### 方法二：较快，占用内存最多，使用hash table

```js
const twoSum = function(nums, target) {
    if (nums.length === 2 && nums[0] + nums[1] === target) return [0, 1];
    const len = nums.length;
    let hashTable = {};
    for (let i = 0; i < len; i++) {
        hashTable[nums[i]] = i;
    }
    for (let i = 0; i < len; i++) {
        let complement = target - nums[i];
        let found = hashTable[complement];
        if (found !== undefined && found !== i) return [i, found];
    }
    return [0, 0];
}
```

#### 方法三：最快，内存占用介于一二之间，one-pass-hash table

```js
const twoSum = function (nums, target) {
    if (nums.length === 2 && nums[0] + nums[1] === target) return [0, 1];
    const len = nums.length;
    let map = {};
    for (let i = 0; i < len; i++) {
        let n = target - nums[i];
        let found = map[n];
        if (found !== undefined) {
            return [found, i];
        } else {
            map[nums[i]] = i;
        }
    }
}
```

## [Add Two Numbers](https://leetcode.com/problems/add-two-numbers/)

### 需求

将两个非空链表（表示两个非负整数，数字以相反的顺序存储每个节点包含一个数字），将这两个数字相加并以链表的形式返回

### 例

addTwoNumbers((2 --> 4 --> 3), (5 --> 6 --> 4)) => 7 --> 0 --> 8  (因为342+465=807)

### 答案

```js
/**
 * Definition for singly-linked list.
 * function ListNode(val) {
 *     this.val = val;
 *     this.next = null;
 * }
 */
/**
 * @param {ListNode} l1
 * @param {ListNode} l2
 * @return {ListNode}
 */
var addTwoNumbers = function(l1, l2) {
    let node = null;
    const carry = arguments[2];
    if (l1 || l2) {
        const val1 = l1 ? l1.val : 0;
        const val2 = l2 ? l2.val : 0;
        const next1 = l1 ? l1.next : null;
        const next2 = l2 ? l2.next : null;
        const val = carry ? val1 + val2 + 1 : val1 + val2;
        node = new ListNode(val % 10);
        node.next = addTwoNumbers(next1, next2, val >= 10);
    } else if (carry) {
        node = new ListNode(1);
        node.next = null;
    }
    return node;
};
```

## [Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/)

### 需求

输入一个字符串，输出该字符串中最长的没有重复字符的子字符串的长度

### 例

lengthOfLongestSubstring("abcabcbb") => 3

### 答案

#### 方法一

```
var lengthOfLongestSubstring = function(s) {
    const len = s.length;
    let ans = 0;
    for (let i = 0; i < len; i++) {
        for (let j = i + 1; j <= len; j++) {
            if (allUnique(s, i, j)) {
                ans = Math.max(ans, j - i);
            }
        }
    }
    return ans;
}
var allUnique = function(s, start, end) {
    let hashSet = {};
    for (let i = start; i < end; i++) {
        const ch = s.charAt(i);
        if (hashSet[ch]) return false;
        hashSet[ch] = ch;
    }
    return true;
}
```

#### 方法二

```
var lengthOfLongestSubstring = function(s) {
    const len = s.length;
    let ans = 0, i = 0, j = 0, hashSet = {};
    while (i < len && j < len) {
        const ch = s.charAt(j);
        if (!hashSet[ch]) {
            hashSet[ch] = ch;
            j += 1;
            ans = Math.max(ans, j - i);
        } else {
            delete hashSet[s.charAt(i)];
            i += 1;
        }
    }
    return ans;
}
```

#### 方法三

```
var lengthOfLongestSubstring = function(s) {
    const len = s.length;
    let ans = 0, map = {};
    for (let j = 0, i = 0; j < len; j++) {
        const ch = s.charAt(j);
        if (map[ch]) {
            i = Math.max(map[ch], i);
        }
        ans = Math.max(ans, j - i + 1);
        map[ch] = j + 1;
    }
    return ans;
}
```

#### 方法四

```
var lengthOfLongestSubstring = function(s) {
    const len = s.length;
    let ans = 0, index = new Array(128);
    for (let j = 0, i = 0; j < len; j++) {
        const ch = s.charAt(j);
        if (index[ch]) {
            i = Math.max(index[ch], i);
        }
        ans = Math.max(ans, j - i + 1);
        index[ch] = j + 1;
    }
    return ans;
}
```

