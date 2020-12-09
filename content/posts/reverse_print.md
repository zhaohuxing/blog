---
title: 从尾到头打印链表
date: 2020-12-09T13:54:39+08:00
draft: false
tags: ["算法"]
---

## 问题描述

输入一个链表，按链表从尾到头的顺序返回一个 ArrayList
```
例如：
输入：【1，2，3，4】
输出：【4，3，2，1】
```

## 解题思路

##### 思路一：
遍历链表，将值存到栈中，然后遍历栈中的值。
##### 思路二：
基于思路一，递归也是一种栈结构，在访问结点时，先递归输出它后面的结点，再输出改结点自身。

> Tips：如果链表很长，就会导致函数调用的层级很深，从而有可能导致函数调用栈溢出。相比之下，思路一更健壮一些。

## 考察点
- 链表遍历
- 栈
- 递归

## 代码实现

#### Go
```
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func reversePrint(head *ListNode) []int {
    vals := []int{}
    for head != nil {
        vals = append(vals, head.Val)
        head = head.Next
    }   

    result := []int{}
    length := len(vals)
    for i := 0; i < len(vals); i++ {
        result = append(result, vals[length-i-1])
    }   
    return result
}
```

#### Rust
```
// Definition for singly-linked list.
// #[derive(PartialEq, Eq, Clone, Debug)]
// pub struct ListNode {
//   pub val: i32,
//   pub next: Option<Box<ListNode>>
// }
// 
// impl ListNode {
//   #[inline]
//   fn new(val: i32) -> Self {
//     ListNode {
//       next: None,
//       val
//     }
//   }
// }
impl Solution {
    pub fn reverse_print(head: Option<Box<ListNode>>) -> Vec<i32> {
        let mut ans: Vec<i32> = vec![];
        let mut current = head;
        while let Some(node) = current {
            ans.push(node.val);
            current = node.next;
        }   
        ans.reverse();
        ans 
    }   
}
```

> 同样的思路，rust 比 go 又快，占内存又少！

参考资料
- 《剑指 Offer》

