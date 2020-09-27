---
title: RainbowSort
date: 2017-11-09 19:09:45
index_img: /images/algorithm.jpg
banner_img: /images/20191231163323.jpg
tags: [算法, JavaScript]
categories: [算法与逻辑]
---
给定一系列球，其中球的颜色只能是红色，黄色或蓝色，对球进行排序，以使所有红色球都分组在左侧，所有黄色球都分组在中间，所有蓝色球分组在右侧。
[要求 Tc: O(n) Sc:O(1)]
例：

> [红] 被排序为 [红]

> [黄，红] 被排序为 [红，黄]

> [黄, 红, 红, 蓝, 黄, 红, 蓝] 被排序为 [红, 红, 红, 黄, 黄, 蓝, 蓝]

假设条件:

> 输入数组不为 null。

corner case:

如果输入数组的长度为零怎么办？在这种情况下，我们应该直接返回空数组。

**解法：**

**思路:** 本题思路是挡板思想, 使用三个挡板四个区域的思想进行划分 (交换数组元素位置)

**挡板的物理意义: [0-i) 全是红色,[i,j) 之间为黄色,(k->n-1] 全为蓝色，[j-k] 为未知探索区域**

j 为快指针

```js
const input = ['黄', '红', '红', '蓝', '黄', '红', '蓝'];

function rainbowSort(rainbow) {
    let i = 0, j = 0, k = rainbow.length - 1;
    while (j <= k) {
        if (rainbow[j] === '红') {
            swap(rainbow, i, j);
            i++;
            j++;
        }
        if (rainbow[j] === '黄') {
            j++;
        }
        if (rainbow[j] === '蓝') {
            swap(rainbow, j, k); //这里不写j++是因为从k交换过来的元素不能保证就是黄色,为了安全起见下次循环再检查一次j位置。
            k--;
        }
    }
}

//辅助交换函数
function swap(arr, i, j) {
    [arr[i], arr[j]] = [arr[j], arr[i]];
}

rainbowSort(input);
```

利用sort：

```js
const input = ['黄', '红', '红', '蓝', '黄', '红', '蓝'];
const enums = {
    '红': 1,
    '黄': 2,
    '蓝': 3
};

function rainbowSort(rainbow) {
    return rainbow.sort((a, b) => {
        return enums[a] - enums[b];
    });
}

rainbowSort(input);
```
