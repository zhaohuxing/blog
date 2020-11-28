---
title: "二维数组中的查找"
date: 2020-11-28T18:25:46+08:00
draft: false
tags: ["算法"]
---

## 问题描述

题目：在一个二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。

```
如下，如果查找数字 7，返回 true；如果查找数字 5，返回 false。

1	2	8	9
2	4	9	12
4	7	10	13
6	8	11	15
```

## 解题思路

##### 思路一：
依次遍历左上角和右下角连线的值，并检查该位置上面的值和左边的值。

##### 思路二：
「右上角的值」与「查找值」比较，如果相等，则结束；如果「右上角的值」大于「查找值」则去除「右上角所在的列」，继续查找；如果「右上角值」小于「查找值」则去除「右上角所在的行」，继续查找。


> 思路二较思路一遍历的数要少。


## 代码实现

##### Go
```
func find(target int, array [][]int) (result bool) {
	if len(array) == 0 {
		return
	}
	i, j := 0, len(array[0])-1
	for i < len(array) && j >= 0 {
		if array[i][j] == target {
			return true
		} else if array[i][j] > target {
			j = j - 1
		} else if array[i][j] < target {
			i = i + 1
		}
	}

	return
}
```

##### Rust
```
pub fn find(target: i32, array: Vec<Vec<i32>>) -> bool {
    let (row_len, col_len) = ( 
        array.len(),
        if array.is_empty() { 0 } else { array[0].len() },
    );  
    let mut row = 0;
    let mut column = col_len - 1;
    while row < row_len {
        match array[row][column].cmp(&target) {
            std::cmp::Ordering::Less => row += 1,
            std::cmp::Ordering::Greater => column -= 1,
            std::cmp::Ordering::Equal => return true,
        }   
    }   
    false
}
```

## 参考资料
- 《剑指 Offer》
