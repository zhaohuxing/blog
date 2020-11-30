---
title: "替换空格"
date: 2020-11-30T13:54:39+08:00
draft: false
tags: ["算法"]
---

## 问题描述

给一个字符串，将字符串中的空格替换成 %20。
```
例如：
输入：“We are happy.”
输出：“We%20are%20happy.”
```

## 解题思路

##### 思路一：
申请一个新变量，遍历整个字符串，如果是空格就替换成 %20 写到新变量中，否则将原值写到新变量中。

##### 思路二：
如果复用当前字符串的话，从前往后遍历，那么每次遇到空格都需要将后面的字符串往后移动。可以先遍历出都多少个空格，算出要正增加的长度。使当前字符串扩充该长度，然后从后开始遍历。

## 代码实现

##### Go
```
func ReplaceBlank(strs string) (result string) {
	return strings.Join(strings.Split(strs, " "), "%20")
}
```

##### Rust
```
fn replace_blank(strs: String) -> String {
    let mut result = "".to_string();
    for c in strs.chars() {
        if c == ' ' {
            result.push_str("%20");
        } else {
            result.push(c);
        }
    }
    result
}
```


