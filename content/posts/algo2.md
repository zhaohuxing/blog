---
title: "找规律题"
date: 2021-06-16T00:00:00+08:00
draft: false
tags: ["算法"]
---

#### [1. 零矩阵 （简单）](https://leetcode-cn.com/problems/zero-matrix-lcci/)

```go
func setZeroes(matrix [][]int) {
    // 1. 首先我需要先找出所有的零值
    // 2. 然后遍历，将其都置为 0

    // 利用 map 去重
    imap := map[int]struct{}{}
    jmap := map[int]struct{}{}
    for i := 0; i < len(matrix); i++ {
        for j := 0; j < len(matrix[0]); j++ {
            if matrix[i][j] == 0 {
                imap[i] = struct{}{}
                jmap[j] = struct{}{}
            }
        }
    }

    rows, cols := len(matrix), len(matrix[0])
    for i := 0; i < rows; i++ {
        for j := range jmap {
            matrix[i][j] = 0
        }
    }
    for i := range imap {
        for j := 0; j < len(matrix[0]); j++ {
            matrix[i][j] = 0
        }
    }
}
```

#### [2. 扑克牌中的顺子(中等)](https://leetcode-cn.com/problems/bu-ke-pai-zhong-de-shun-zi-lcof/)

```go
func isStraight(nums []int) bool {
    // 我们需要把 nums 排序
    // 统计 0 的个数
    // 从非零的位置开始遍历，比较前后值是否相差一，如果相差不是一，则看看是否有 0 可以补，
    // 同时记录顺子的个数

    sort.Ints(nums)
    zeros, straights := 0, 0
    for i := 0; i < len(nums); i++ {
        if nums[i] != 0 {
            break
        }
        zeros++
    }
    if zeros == len(nums) {
        return true
    }

    for i := zeros; i < len(nums); i++ {
        if straights == 0 {
            straights++
            continue
        }

        v := nums[i] - nums[i-1]
        if v-1 > zeros || v == 0 {
            return false
        }
        zeros -= (v - 1)
        straights += v
    }

    // 遍历完 zeros 可能还要剩余，比如：0, 0, 0, 9, 11
    straights += zeros

    if straights == len(nums) {
        return true
    }
    return false
}
```

#### [3. 跳水板（简单）](https://leetcode-cn.com/problems/diving-board-lcci/)

```go
func divingBoard(shorter, longer, k int) []int {
    // 如果 k 为零，直接返回
    // 如果 shorter == longer，直接返回 shorter * longer
    // 用 i 标记 longer 的数量
    if k == 0 {
        return []int{}
    }
    if shorter == longer {
        return []int{shorter * k}
    }

    result := []int{}
    for i := 0; i <= k; i++ {
        result = append(result, shorter*(k-i)+longer*i)
    }
    return result
}
```

#### [4.一次编辑（中等）](https://leetcode-cn.com/problems/one-away-lcci/)

```go
func oneEditAway(first string, second string) bool {
    if first == second {
        return true
    }

    flag := false
    p1, p2 := 0, 0
    for p1 < len(first) || p2 < len(second) {
        if p1 < len(first) && p2 < len(second) && first[p1] == second[p2] {
            p1++
            p2++
            continue
        } else if !flag {
            if len(first)-p1 > len(second)-p2 {
                p1++
            } else if len(first)-p1 < len(second)-p2 {
                p2++
            } else {
                p1++
                p2++
            }
            flag = true
        } else {
            return false
        }
    }

    if p1 == len(first) && p2 == len(second) {
        return true
    }
    return false
}
```

#### [5. 珠玑妙算 （中等）](https://leetcode-cn.com/problems/master-mind-lcci/)

```go
func masterMind(solution, guess string) []int {
    result := []int{0, 0}
    m := map[byte]int{}

    for i := 0; i < 4; i++ {
        m[solution[i]]++
    }

    for i := 0; i < 4; i++ {
        if solution[i] == guess[i] {
            result[0]++
        }
        if m[guess[i]] > 0 {
            result[1]++
            m[guess[i]]--
        }
    }
    result[1] -= result[0]
    return result
}
```

#### [6. 面试题 16.04. 井字游戏（中等）](https://leetcode-cn.com/problems/tic-tac-toe-lcci/)

```go
func tictactoe(board []string) string {
    stringToNum := map[string]int{
        "X": 1,
        "O": 0,
        " ": -100,
    }

    flag := 0
    // 计算列
    for i := 0; i < len(board); i++ {
        sum := 0
        for j := 0; j < len(board); j++ {
            sum += stringToNum[string(board[j][i])]
        }
        if sum == len(board) {
            return "X"
        } else if sum == 0 {
            return "O"
        } else if sum < 0 {
            flag = 1
        }
    }

    // 计算行
    for i := 0; i < len(board); i++ {
        sum := 0
        for j := 0; j < len(board); j++ {
            sum += stringToNum[string(board[i][j])]
        }
        if sum == len(board) {
            return "X"
        } else if sum == 0 {
            return "O"
        } else if sum < 0 {
            flag = 1
        }
    }

    sum1 := 0 // 右上左下对角线之和
    sum2 := 0 // 左上右下对角线之和
    for i := 0; i < len(board); i++ {
        sum1 = sum1 + stringToNum(string(board[len(board)-i-1][i]))
        sum2 = sum2 + stringToNum(string(board[i][i]))
    }
    if sum1 == len(board) || sum2 == len(board) {
        return "X"
    } else if sum1 == 0 || sum2 == 0 {
        return "O"
    } else if sum1 < 0 || sum2 < 0 {
        flag = 1
    }

    if flag == 1 {
        return "Pending"
    }

    return "Draw"
}
```

#### [7.跳跃游戏 （中等）](https://leetcode-cn.com/problems/jump-game/)

```go
func canJump(nums []int) bool {
    reach, n := 0, len(nums)
    for i := 0; i < n; i++ {
        if i > reach {
            return false
        }
        if reach < i + nums[i] {
            reach = i + nums[i]
        }
    }
    return true
}
```

#### [8. 旋转图像 （中等）](https://leetcode-cn.com/problems/rotate-image/)

```go
func rotate(matrix [][]int) {
    n := len(matrix)
    for i := 0; i < n/2; i++ {
        for j := 0; j < (n+1)/2; j++ {
            matrix[i][j], matrix[n-j-1][i], matrix[n-i-1][n-j-1], matrix[j][n-i-1] =
                matrix[n-j-1][i], matrix[n-i-1][n-j-1], matrix[j][n-i-1], matrix[i][j]
        }
    }
}
```

#### [9. 螺旋矩阵（中等）](https://leetcode-cn.com/problems/spiral-matrix/)

```go
unc spiralOrder(matrix [][]int) []int {
	if len(matrix) == 0 || len(matrix[0]) == 0 {
		return []int{}
	}

	// 定义四个下标，从外层往里层打印
	order := make([]int, 0)
	left, right, top, bottom := 0, len(matrix[0])-1, 0, len(matrix)-1

	for left <= right && top <= bottom {
		for column := left; column <= right; column++ {
			order = append(order, matrix[top][column])
		}
		for row := top + 1; row <= bottom; row++ {
			order = append(order, matrix[row][right])
		}
		if left < right && top < bottom {
			for column := right - 1; column > left; column-- {
				order = append(order, matrix[bottom][column])
			}
			for row := bottom; row > top; row-- {
				order = append(order, matrix[row][left])
			}
		}
		left++
		right--
		top++
		bottom--
	}
	return order
}
```

#### [10.搜索二维矩阵 II （中等）](https://leetcode-cn.com/problems/search-a-2d-matrix-ii/)

```go
func searchMatrix(array [][]int, target int) (result bool) {
    if len(array) == 0 {
        return
    }

    // 这个矩阵的特点，右边比左边大，下边比上边大。
    // 那就从右上角开始比较:
    // 1. 如果相等，则返回
    // 2. 如果 array[i][j] < target，则行坐标 + 1
    // 3. 如果 array[i][j] < target, 则列坐标 + 1

    i, j := 0, len(array[0])-1
    for i < len(arary) && j >= 0 {
        if array[i][j] == target {
            return true
        } else if array[i][j] < target {
            i++
        } else {
            j--
        }
    }

    return
}
```
