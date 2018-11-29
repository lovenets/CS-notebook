#### 1.[Range Sum of BST](https://leetcode.com/problems/range-sum-of-bst/)

Given the `root` node of a binary search tree, return the sum of values of all nodes with value between `L` and `R` (inclusive).

The binary search tree is guaranteed to have unique values.

**Example 1:**

```
Input: root = [10,5,15,3,7,null,18], L = 7, R = 15
Output: 32
```

**Example 2:**

```
Input: root = [10,5,15,3,7,13,18,1,null,6], L = 6, R = 10
Output: 23
```

**Note:**

1. The number of nodes in the tree is at most `10000`.
2. The final answer is guaranteed to be less than `2^31`.

**Solution**

(1) Recursion

```go
func rangeSumBST(root *TreeNode, L int, R int) int {
	sum := 0
	// Depth-first traverse the tree
	dfs(root, L, R, &sum)
	return sum
}

func dfs(node *TreeNode, L int, R int, sum *int) {
	if node == nil {
		return
	}

	if node.Val >= L && node.Val <= R {
		*sum += node.Val
	}
	if node.Val > L {
		dfs(node.Left, L, R, sum)
	}
	if node.Val < R {
		dfs(node.Right, L, R, sum)
	}
}
```

Time complexity: $$O(n)​$$, n is the number of nodes.

(2) Iteration

```go
func rangeSumBST(root *TreeNode, L int, R int) int {
    sum := 0
    stack := make([]*TreeNode,0)
    stack = append(stack,root)
    
    for len(stack) > 0 {
        node := stack[len(stack)-1]
        stack = append(stack[:len(stack)-1],stack[len(stack):]...)
        
        if node == nil {
            continue
        }
        if node.Val >= L && node.Val <= R {
            sum += node.Val
        }
        if L < node.Val {
            stack = append(stack,node.Left)
        }
        if R > node.Val {
            stack = append(stack,node.Right)
        }
    }
    
    return sum
}
```

Time complexity: $$O(n)$$, n is the number of nodes.