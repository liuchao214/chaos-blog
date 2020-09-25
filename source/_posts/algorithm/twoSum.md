---
title: 'twoSum [要求 Tc: O(n) Sc:O(n)]'
date: 2017-05-07 14:04:21
index_img: /images/algorithm.jpg
banner_img: /images/20191231163323.jpg
tags: [算法, JavaScript]
categories: [算法与逻辑]
---
[LeetCode 第 1 题](https://leetcode-cn.com/problems/two-sum/)

按照题目要求, 我们第一时间想到的会是两层循环暴力解法：

#### **解法1: Time = O(n²), Space = O(1)**

思路: 遍历每个元素 nums[j]，并查找是否存在一个值与 target - nums[j] 相等的目标元素。

```js
function twoSum(nums, target) {
     for (let i = 0; i < nums.length; i++) {
         for (let j = i + 1; j < nums.length; j++) {
             if (nums[j] == target - nums[i]) {
                 return [i,j];
             }
         }
     }
     return [];
}

```

#### **解法2: Time = O(n), Space = O(n)**

我们可以通过哈希表空间换时间。在进行迭代并将元素插入到表中的同时，我们回过头来检查哈希表中是否已经存在当前元素所对应的目标元素，如果存在，那我们就找到了问题的解，将其返回即可.(时间复杂度为 O(n), 空间复杂度也为 O(n))

符合题目要求 bingo✌

```js
var twoSum = function(nums, target) {
    let reduceHash = {};
    for (let i = 0; i < nums.length; i++) {
        let reduceResult = target - nums[i];
        if (reduceHash[reduceResult] !== undefined) {
            return [reduceHash[reduceResult], i];
        }
        reduceHash[nums[i]] = i;
    }
};
```
