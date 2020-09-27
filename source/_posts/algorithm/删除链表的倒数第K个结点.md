---
title: 删除链表的倒数第K个结点
date: 2018-11-27 15:52:16
index_img: /images/algorithm.jpg
banner_img: /images/20191231163323.jpg
tags: [算法, JavaScript]
categories: [算法与逻辑]
---
删除链表的倒数第 K 个结点 [要求 Tc: O(L) Sc:O(1)]

[LeetCode 第 19 题](https://leetcode-cn.com/problems/remove-nth-node-from-end-of-list/)

#### 基本解法
```js
var removeNthFromEnd = function(head, n) {
    const findNode = (node, n, tempNode, level = 0) => {
        if (node.next) {
            if (level < n) {
                return findNode(node.next, n, tempNode, level + 1);
            } else {
                return findNode(node.next, n, tempNode.next, level);
            }
        } else {
            if (level === n) {
                return tempNode;
            } else {
                return null;
            }

        }
    };
    const deletedNode = findNode(head, n, head);
    if (deletedNode && deletedNode.next) {
        deletedNode.next = deletedNode.next.next;
    } else {
        if (head.next) {
            head = head.next;
        } else {
            head = null;
        }
    }
    return head;
};
```

#### 快慢指针
```js
var removeNthFromEnd = function(head, n) {
    let dummyNode = new ListNode(0);
    dummyNode.next = head;
    let fast = dummyNode, slow = dummyNode;
    // 快先走 n+1 步
    while(n--) {
        fast = fast.next
    }
    // fast、slow 一起前进
    while(fast && fast.next) {
        fast = fast.next
        slow = slow.next
    }
    slow.next = slow.next.next
    return dummyNode.next
};
```
