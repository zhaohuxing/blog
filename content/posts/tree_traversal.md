---
title: "二叉树遍历"
date: 2020-12-16T13:54:39+08:00
draft: false
tags: ["算法"]
---

## 二叉树（每个结点最多有两个子结点）

- 前序遍历：先访问根结点，再访问左子结点，最后访问右子结点
- 中序遍历：先访问左子结点，再访问根结点，最后访问右子结点
- 后序遍历：先访问左子结点，再访问右子结点，最后访问根结点
- 宽度优先遍历：从上到下，从左到右。



## 代码实现

##### Go
```
package main

import "fmt"

type TreeNode struct {
	Val   int
	Left  *TreeNode
	Right *TreeNode
}

func preorderTraversalRecursive(root *TreeNode) []int {
	result := []int{}
	if root != nil {
		result = append(result, root.Val)
		if root.Left != nil {
			result = append(result, preorderTraversalRecursive(root.Left)...)
		}
		if root.Right != nil {
			result = append(result, preorderTraversalRecursive(root.Right)...)
		}
	}
	return result
}

func preorderTraversal(root *TreeNode) []int {
	result := []int{}
	if root == nil {
		return result
	}

	stack := []*TreeNode{root}
	for len(stack) > 0 {
		// Pop
		node := stack[len(stack)-1]
		stack = stack[:len(stack)-1]
		result = append(result, node.Val)

		// Push right selfnode
		if node.Right != nil {
			stack = append(stack, node.Right)
		}

		// Push left selfnode
		if node.Left != nil {
			stack = append(stack, node.Left)
		}
	}

	return result
}

func inorderTraversal(root *TreeNode) []int {
	result := []int{}
	if root == nil {
		return result
	}

	stack := []*TreeNode{}
	currentNode := root
	for currentNode != nil || len(stack) > 0 {
		if currentNode != nil {
			stack = append(stack, currentNode)
			currentNode = currentNode.Left
		} else {
			currentNode = stack[len(stack)-1]
			stack = stack[:len(stack)-1]
			result = append(result, currentNode.Val)
			currentNode = currentNode.Right
		}
	}
	return result
}

func inorderTraversalRecursive(root *TreeNode) []int {
	result := []int{}
	if root != nil {
		if root.Left != nil {
			result = append(result, inorderTraversalRecursive(root.Left)...)
		}
		result = append(result, root.Val)
		if root.Right != nil {
			result = append(result, inorderTraversalRecursive(root.Right)...)
		}
	}
	return result
}

func postorderTraversalRecursive(root *TreeNode) []int {
	result := []int{}
	if root != nil {
		if root.Left != nil {
			result = append(result, postorderTraversalRecursive(root.Left)...)
		}
		if root.Right != nil {
			result = append(result, postorderTraversalRecursive(root.Right)...)
		}
		result = append(result, root.Val)
	}
	return result
}

func postorderTraversal(root *TreeNode) []int {
	result := []int{}
	if root == nil {
		return result
	}

	stack := []*TreeNode{root}
	for len(stack) > 0 {
		node := stack[len(stack)-1]
		stack = stack[:len(stack)-1]
		result = append(result, node.Val)

		if node.Left != nil {
			stack = append(stack, node.Left)
		}
		if node.Right != nil {
			stack = append(stack, node.Right)
		}
	}
	for i, j := 0, len(result)-1; i < j; i, j = i+1, j-1 {
		result[i], result[j] = result[j], result[i]
	}
	return result
}

func levelorderTraversal(root *TreeNode) []int {
	result := []int{}
	if root == nil {
		return result
	}

	levelNodes := []*TreeNode{root}
	for {
		if len(levelNodes) == 0 {
			break
		}

		nodes := []*TreeNode{}
		for _, node := range levelNodes {
			result = append(result, node.Val)
			if node.Left != nil {
				nodes = append(nodes, node.Left)
			}
			if node.Right != nil {
				nodes = append(nodes, node.Right)
			}
		}
		levelNodes = nodes
	}
	return result
}

func main() {
	node := &TreeNode{
		Val: 1,
		Right: &TreeNode{
			Val: 2,
			Left: &TreeNode{
				Val: 3,
			},
			Right: &TreeNode{
				Val: 4,
				Left: &TreeNode{
					Val: 5,
					Left: &TreeNode{
						Val: 6,
					},
					Right: &TreeNode{
						Val: 7,
					},
				},
			},
		},
	}
	fmt.Println("递归：")
	fmt.Println(preorderTraversalRecursive(node))
	fmt.Println(inorderTraversalRecursive(node))
	fmt.Println(postorderTraversalRecursive(node))
	fmt.Println("迭代：")
	fmt.Println(preorderTraversal(node))
	fmt.Println(inorderTraversal(node))
	fmt.Println(postorderTraversal(node))

	 fmt.Println(levelorderTraversal(node))
}
```
