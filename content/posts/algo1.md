---
title: "纯编程题"
date: 2021-06-150T00:00:00+08:00
draft: false
tags: ["算法"]

---

## 纯编程题解题思路

1. 举例读懂题意，梳理题目要求
2. 列出测试用例（测试驱动开发）
3. 总结归纳处理思想（把逻辑中重复的部分抽象出来）
4. 第一轮编写（写注释，写清逻辑）
5. 使用测试用例验证

## 练习
#### [1.两数之和（简单）](https://leetcode-cn.com/problems/two-sum/)

```go
func twoSum(nums []int, target int) []int {
    // 思路：遍历数组，检查 map 中是否含有 target - nums[i], 如果有的话，则查找完毕。
    // 如果没有将 （nums[i], i）写入 map 中。

    m := make(map[int]int)
    for i, v := range nums {
        if k, ok := m[target-v]; ok {
            return []int{k, i}
        }
        m[v] = i 
    }
    return []int{}
}
```

#### [2.IP 地址无效化（简单）](https://leetcode-cn.com/problems/defanging-an-ip-address/) 

```go
import "strings"

func defangIPaddr(address string) string {
    // 将 address 以 "." 分割开
    // 遍历分割后的数组时，在后面追加 "[.]"，当遍历最后一个时，不需要追加。
    segments := strings.Split(address, ".")
    result := []byte{}
    for i, v := range segments {
        result = append(result, v...)
        if i < len(segments) - 1 {
            result = append(result, "[.]"...)
        }
    }
    return string(result)
}
```

#### [3. 反转字符串（简单）](https://leetcode-cn.com/problems/reverse-string/)

```go
func reverseString(s []byte)  {
    // 坐标前后置换。
    // 1, 2, 3, 4 -> 2, 
    // 1, 2, 3, 4, 5 -> 2,
    for i := 0; i < len(s)/2; i++ {
        s[i], s[len(s)-i-1] = s[len(s)-i-1], s[i]
    }
}
```

[4.剑指 Offer 58 - I. 翻转单词顺序（简单）](https://leetcode-cn.com/problems/fan-zhuan-dan-ci-shun-xu-lcof/) 

```go
func reverseWords(s string) string {
    // 倒序遍历字符串，用双指针来标记单词，每标记一个，则将其 push 到单词数组中
    // 遍历单词数组且用 " " 进行连接
    // 注意事项：单词与单词之间可能存在多个空格，字符串头尾可能存在空格。
    words := []string{}
    for i := len(s) - 1; i >=0; i-- {
        if s[i] == ' ' {
            continue
        }
        for j := i; j >= 0; j-- {
            if s[j] == ' ' {
                words = append(words, s[j+1:i+1])
                i = j
                break
            } else if j == 0 {
                words = append(words, s[j:i+1])
                i = j
                break
            }
        }
    }
    
    ret := []byte{}
    for i := 0; i < len(words); i++ {
        ret = append(ret, words[i]...)
        if i + 1 != len(words) {
            ret = append(ret, ' ')
        }
    }
    return string(ret)
}
```

#### [5. 验证回文串（简单）](https://leetcode-cn.com/problems/valid-palindrome/) 

```go
import "strings"

func isPalindrome(s string) bool {
    // 回文串的意思：正着读，反着读有一样。
    // 这个题忽略字母的大小写，忽略标点符号
    bytes := []byte{}
    for i := 0; i < len(s); i++ {
        if isalnum(s[i]) {
            bytes = append(bytes, s[i])
        }
    }

    s = strings.ToLower(string(bytes))
    for i := 0; i < len(s)/2; i++ {
        if s[i] != s[len(s) - 1 -i] {
            return false
        }
    }
    return true
}

func isalnum(c byte) bool {
    return (c >= 'A' && c <= 'Z') || (c >= 'a' && c <= 'z') || (c >= '0' && c <= '9')
}
```



#### [6. 回文数（简单）](https://leetcode-cn.com/problems/palindrome-number/)

```go
import "strconv"

func isPalindrome(x int) bool {
    // 将 int 转成 string，然后比较字符
    str := strconv.Itoa(x)
    for i := 0; i < len(str)/2; i++ {
        if str[i] != str[len(str)-i-1] {
            return false
        }
    }
    return true
}
```

#### [7. 最后一个单词的长度（简单）](https://leetcode-cn.com/problems/length-of-last-word/)

```go
func lengthOfLastWord(s string) int {
    // 1. "world "
    // 2. " world "
    count := 0
    for i := len(s) - 1; i >= 0; i-- {
        if s[i] == ' ' {
            if count > 0 {
                break
            } else {
                continue
            }
        }
        count++
    }
    return count
}
```

#### [8.剑指 Offer 05. 替换空格（简单）](https://leetcode-cn.com/problems/ti-huan-kong-ge-lcof/)

```go
func replaceSpace(s string) string {
    ret := []byte{}
    for i := 0; i < len(s); i++ {
        if s[i] == ' ' {
            ret = append(ret, []byte("%20")...)
        } else {
            ret = append(ret, s[i])
        }
    }
    return string(ret)
}
```

#### [9.剑指 Offer 58 - II. 左旋转字符串（简单）](https://leetcode-cn.com/problems/zuo-xuan-zhuan-zi-fu-chuan-lcof/)

```go
func reverseLeftWords(s string, n int) string {
    ret := []byte{}
    ret = append(ret, s[n:]...)
    ret = append(ret, s[:n]...)
    return string(ret)
}
```

#### [10. 删除排序数组中的重复项（简单）](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-array/)

```go
func removeDuplicates(nums []int) int {
    // 遍历 for i := 0; i < len(nums); i++ 
    // 用 idx 表示不重复下标
    idx := 0
    for i := 0; i < len(nums); i++ {
        if i == 0 {
            idx++
            continue
        }
        if nums[i] == nums[idx-1] {
            continue
        }
        nums[i], nums[idx] = nums[idx], nums[i]
        idx++
    }
    return idx
}
```

#### [11.剑指 Offer 67. 把字符串转换成整数（中等）](https://leetcode-cn.com/problems/ba-zi-fu-chuan-zhuan-huan-cheng-zheng-shu-lcof/)

```go
func strToInt(str string) int {
	// 第一个字符如果是 '-', '+' 才进行转换
	// result = result * 10 + int (v - '0')
	// 如果 result 大于 int32 最大值，则根据符合决定返回 Int32.Max 还是 Int32.Min
	// 我们需要过滤字符前面的 ' '。

	result, i, sign, length := 0, 0, 1, len(str)
	if len(str) == 0 {
		return 0
	}
	for str[i] == ' ' {
		i++
		if i == length {
			return 0
		}
	}

	for idx, v := range str[i:] {
		if v >= '0' && v <= '9' {
			result = result*10 + int(v-'0')
		} else if v == '-' && idx == 0 {
			sign = -1
		} else if v == '+' && idx == 0 {
			sign = 1
		} else {
			break
		}

		if result > math.MaxInt32 {
			if sign == -1 {
				return math.MinInt32
			}
			return math.MaxInt32
		}

	}
	return sign * result
}
```

