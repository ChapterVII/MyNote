---
title: leetCode训练（二）
date: 2019-01-11 23:21:50
tags: 
---

## [Longest Palindromic Substring](https://leetcode.com/problems/longest-palindromic-substring/)

### 需求

给定一个字符串，输出最长的回文子串

### 例

longestPalindrome('babad')  => 'aba'

### 答案

#### 方法一：暴力求解，列举所有的子串，判断是否为回文串，保存最长的回文串

```js
const longestPalindrome = function (s) {
    let ans = "";
    let max = 0;
    const len = s.length;
    for (let i = 0; i < len; i++) {
        for (let j = i + 1; j <= len; j++) {
            let sub = s.substring(i, j);
            if (isPalindromic(sub) && sub.length > max) {
                ans = sub;
                max = Math.max(max, ans.length);
            }
        }
    }
    return ans;
}

function isPalindromic (s) {
    let len = s.length;
    for (let i = 0; i < len / 2; i++) {
        if (s.charAt(i) !== s.charAt(len - i - 1)) {
            return false;
        }
    }
    return true;
}
```

#### 方法二：较快，占用内存最多，使用hash table

```js
const longestPalindrome = function (s) {
    const len = s.length;
    if (!len) return "";
    const origin = s.split('');
    const reverse = s.split('').reverse();
    let arr = new Array(len);
    let maxLen = 0, maxEnd = 0;

    for (let i = 0; i < len; i++) {
        arr[i] = new Array(len);
        for (let j = 0; j < len; j++) {
            if (origin[i] === reverse[j]) {
                if (i === 0 || j === 0) {
                    arr[i][j] = 1;
                } else {
                    arr[i][j] = arr[i - 1][j - 1] + 1;
                }
                if (arr[i][j] > maxLen) {
                    const beforeRev = len - 1 -j;
                    if (beforeRev + arr[i][j] - 1 === i) {
                        maxLen = arr[i][j];
                        maxEnd = i;
                    }
                }
            } else {
                arr[i][j] = 0;
            }
        }
    }
    return s.substring(maxEnd - maxLen, maxEnd + 1);
}
```

## 扩展

### 求两个字符串的最长公共子串

#### 算法思路：

1、把两个字符串分别以行和列组成一个二维矩阵。

2、比较二维矩阵中每个点对应行列字符中否相等，相等的话值设置为1，否则设置为0。

3、通过查找出值为1的最长对角线就能找到最长公共子串。

为了进一步优化算法的效率，我们可以再计算某个二维矩阵的值的时候顺便计算出来当前最长的公共子串的长度，即某个二维矩阵元素的值由`item[i][j]=1`演变为`item[i][j]=1 +item[[i-1][j-1]`这样就避免了后续查找对角线长度的操作了。

```js
const getLCSLength = function (str1, str2) {
    const arr1 = str1.split('');
    const arr2 = str2.split('');
    let temp = new Array(arr1.length), length = 0;
    for (let i = 0; i < arr1.length; i++) {
        temp[i] = new Array(arr2.length);
        temp[i][0] = (arr2[0] === arr1[i]) ? 1 : 0;
    }
    for (let j = 0; j < arr2.length; j++) {
        temp[0][j] = (arr1[0] === arr2[j]) ? 1 : 0;
    }
    for (let i = 1; i < arr1.length; i++) {
        for (let j = 1; j < arr2.length; j++) {
            if (arr1[i] === arr2[j]) {
                temp[i][j] = temp[i - 1][j - 1] + 1;
                if (temp[i][j] > length) {
                    length = temp[i][j];
                }
            } else {
                temp[i][j] = 0;
            }
        }
    }
    return length;
}
```

参考http://windliang.cc/2018/08/05/leetCode-5-Longest-Palindromic-Substring/

