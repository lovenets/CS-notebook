>  The key to solve algorithm problems posed in technical interviews or elsewhere is to quickly identify the underlying patterns.

## Iterative In-order Traversal 

### Template

```go
func inorder(root *TreeNode) []*TreeNode {
    res := make([]int, 0)
    stack := make([]*TreeNode, 0)
    if root == nil {
        return res
    }
    cur := root
    for cur != nil || len(stack) > 0 {
        // Push left child if available
        for cur != nil {
            stack = append(stack, cur)
            cur = cur.Left
        }
        // Pop the stack and get its right child if available
        top := stack[len(stack)-1]
        stack = stack[:len(stack)-1]
        res = append(res, top.Val)
        cur = top.Right
    } 
    return res
}
```

### Problems

We can use iterative in-order traversal to solve many problems about BST since the in-order traversal of a BST must be ascending.

(1) [Validate if Tree is a BST](https://leetcode.com/problems/validate-binary-search-tree/)

```go
func isValidBST(root *TreeNode) bool {
    if root == nil {
        return true 
    }
    stack := make([]*TreeNode, 0)
    var pre *TreeNode = nil
    for cur := root; cur != nil || len(stack) > 0; {
        for cur != nil {
            stack = append(stack, cur)
            cur = cur.Left
        }
        top := stack[len(stack)-1]
        stack = stack[:len(stack)-1]
        if pre != nil && pre.Val >= top.Val {
            return false
        }
        pre = top
        cur = top.Right
    }
    return true
}
```

(2) [Kth smallest Element in a BST](https://leetcode.com/problems/kth-smallest-element-in-a-bst/description/)

```go
func kthSmallest(root *TreeNode, k int) int {
    stack := make([]*TreeNode, 0)
    cur := root 
    for cur != nil || len(stack) > 0 {
        for cur != nil {
            stack = append(stack, cur)
            cur = cur.Left
        }
        node := stack[len(stack)-1]
        stack = stack[:len(stack)-1]
        if k = k - 1; k == 0 {
            return node.Val
        }
        cur = node.Right
    }
    return -1
}
```

## DFS

### Template 

Below is the iterative DFS pattern using a stack that will allow us to solve a ton of problems.

```
1] push to stack
2] pop top 
3] retrieve unvisited neighbours of top, push them to stack 
4] repeat 1,2,3 while stack not empty.
```

> A tree can be thought of as a connected acyclic graph with N nodes and N-1 edges. Any two vertices are connected by *exactly one* path. So naturally the question arises, what about a DFS or a BFS on binary trees ? well there are 6 possible DFS traversals for binary trees ( 3 rose to fame while the other 3 are just symmetric )
>
> 1. left, right, root ( Postorder) ~ 4. right, left, root
> 2. left, root, right ( Inorder) ~ 5. right, root, left
> 3. root, left, right ( Preorder) ~ 6. root, right, left
>
> And there is one BFS which is the level order traversal ( can be done using queue). 

### Problems 

(1) [Binary Tree Preorder Traversal](https://leetcode.com/problems/binary-tree-preorder-traversal/)

```go
func preorderTraversal(root *TreeNode) []int {
    res := make([]int, 0)
    stack := make([]*TreeNode, 0)
    stack = append(stack, root)
    for len(stack) > 0 {
        cur := stack[len(stack)-1]
        stack = stack[:len(stack)-1]
        if cur != nil {
            stack = append(stack, cur.Right)
            stack = append(stack, cur.Left)
            res = append(res, cur.Val)
        }
    }
    return res
}
```

Note that it's impossible for a tree to have a cycle so we don't need to keep track of visited nodes. 

(2) Number of Connected Components in an Undirected Graph 

```go
func countComponents(n int, edges [][]int) int {
    visited := make([]bool, n)
    adjList := make([][]int, n)
    for i := range adjList {
        adjList[i] = make([]int, 0) 
    }
    stack := make([]int, 0)
    count, res := 0, 0
    
    // Construct the adjacency list
    for i := range edges {
        f, t := edges[i][0], edges[i][1]
        adjList[f] = append(adjList[f], t)
        adjList[t] = append(adjList[t], f)
    }
    // DFS
    for i := 0; i < n; i++ {
        if !visited[i] {
            res++
            stack = append(stack, i)
            for len(stack) > 0 {
                cur := stack[len(stack)-1]
                stack = stack[:len(stack)-1]
                visited[cur] = true
                for j := range adjList[cur] {
                    if !visited[adjList[cur][j]] {
                        stack = append(stack, adjList[cur][j])
                    }
                }
            }
        }
    }
    return res
}
```

## BFS

### Template 

We use a queue to implement BFS.

```
1] enqueue
2] dequeue
3] retrieve unvisited neighbours of head, enqueue them
4] repeat 1,2,3 while queue not empty.
```

### Problems 

(1) [Binary Tree Level Order Traversal](https://leetcode.com/problems/binary-tree-level-order-traversal/)

```go
func levelOrder(root *TreeNode) [][]int {
    if root == nil {
        return [][]int{}
    }
    
    res := make([][]int, 0)
    queue := make([]*TreeNode, 0)
    queue = append(queue, root)
    for {
        // count indicates the number of nodes in current level
        count := len(queue)
        if count == 0 {
            break
        }
        curLevel := make([]int, 0)
        // traverse current level
        for count > 0 {
            node := queue[0]
            queue = queue[1:]
            curLevel = append(curLevel, node.Val)
            count--
            if node.Left != nil {
                queue = append(queue, node.Left)
            }
            if node.Right != nil {
                queue = append(queue, node.Right)
            }
        }
        res = append(res, curLevel)
    }
    return res
}
```

(2) [01 Matrix](https://leetcode.com/problems/01-matrix/)

```go
func updateMatrix(matrix [][]int) [][]int {
    queue := make([][]int, 0)
    m, n := len(matrix), len(matrix[0])
    for i := 0; i < m; i++ {
        for j := 0; j < n; j++ {
            if matrix[i][j] == 0 {
                // Keep track of all 0s
                queue = append(queue, []int{i, j})
            } else {
                // Or initialize distance to infinity 
                matrix[i][j] = math.MaxInt8   
            }
        }
    }
    // BFS
    directions := [][]int {{1, 0}, {-1, 0}, {0, 1}, {0, -1}}
    for len(queue) > 0 {
        cur := queue[0]
        queue = queue[1:]
        // Retrieve near cells
        for _, d := range directions {
            i, j := cur[0] + d[0], cur[1] + d[1]
            if dist := matrix[cur[0]][cur[1]]+1; i >= 0 && i < m && j >= 0 && j < n && matrix[i][j] > dist {
                queue = append(queue, []int{i, j})
                // Update smaller distance 
                matrix[i][j] = dist
            }
        }
    }
    return matrix
}
```

### Conclusion

1. Problems in which you have to *find shortest path* are most likely calling for a BFS.
2. *For graphs having unit edge distances*, shortest paths from any point is just a BFS starting at that point, no need for Dijkstra’s algorithm.

> Why are we using stack for DFS , couldn’t we use a queue ? ( always remember : stack for DFS, imagine a vertical flow | queue for BFS, horizontal flow, more on this later)

## Sliding Window

### Template 

(1) Array

Let's say we want to calculate the maximum sum of all subarrays whose lengths are `k`.

![img](https://cdn-images-1.medium.com/max/1100/1*RMCjDprUdgiQI-2kbroDqA.jpeg)

```go
max, window := 0, 0
// Calculate sum of first window
for i := 0; i < k; i++ {
    window += arr[i]
}
// Move forward window
for i := k; i < len(arr); i++ {
    // Save re-computation
    window += arr[i] - arr[i-k]
    if max < window {
        max = window
    }
}
```

(2) Substring 

For most substring problem, we are given a string and need to find a substring of it which satisfy some restrictions. A general way is to use a hash table assisted with two pointers which indicate a sliding window. 

```go
func findSubstring(s string) int {
    m := make(map[uint8]int)
    // Check if the substring is valid
    var count int
    // Two pointers indicating the sliding window
    beg, end := 0, 0
    // The length of substring
    var d int
    
    // Initialize the hash table here
    
    for end < len(s) {
        if condition(m[s[end]]) {
            // Modify count here
        }
        m[s[end]]--
        end++
        
        for condition(count) {
            // Update d if found a shorter substring
            
            // Move forward beg pointer to make substring invalid /valid again
            
            if condition(m[s[beg]]) {
                // Modify count here
            }
            m[s[beg]]++
            beg++
        }
        
        // Update d if found a longer substring
    }
    
    return d
}
```

One thing needs to be mentioned is that when asked to find maximum substring, we should update maximum after the inner while loop to guarantee that the substring is valid. On the other hand, when asked to find minimum substring, we should update minimum inside the inner while loop.

### Problems 

(1) [Minimum Window Substring](https://leetcode.com/problems/minimum-window-substring/description/)

```go
func minWindow(s string, t string) string {
	table := make(map[uint8]int)
	for i := range t {
		table[t[i]]++
	}
	count := len(t)
	beg, end, head := 0, 0, 0
	l := math.MaxInt64
	for end < len(s) {
		if table[s[end]] > 0 {
			count--
		}
		table[s[end]]--
		end++
		for count == 0 {
			if end-beg < l {
				l = end - beg
				head = beg
			}
			if table[s[beg]] == 0 {
				count++
			}
			table[s[beg]]++
			beg++
		}
	}
	if l == math.MaxInt64 {
		return ""
	} else {
		return s[head : head+l]
	}
}
```

(2) Longest Substring with At Most Two Distinct Characters

```go
func lengthOfLongestSubstringTwoDistinct(s string) int {
    m := make(map[uint8]int)
    count := 0
    beg, end := 0
    res := 0
    for end < len(s) {
        if m[s[end]] == 0 {
            count++
        }
        m[s[end]]++
        end++
        for count > 2 {
            if m[s[beg]] == 1 {
                count--
            }
            m[s[beg]]--
            beg++
            if l := end-beg; l > res {
                res = l
            } 
        }
    }
    return res
}
```

## Prefix Sum 

### Template 

 If we know `SUM[0, i - 1]` and `SUM[0, j]`, then we can easily get `SUM[i, j].` We can use prefix sum to solve problems associated with sum of subarray.

Keep in mind that we can use a hash map to accelerate the process by mapping sum to its corresponding index.

### Problems

(1) [Contiguous Array](https://leetcode.com/problems/contiguous-array)

```go
func findMaxLength(nums []int) int {
    sum2index := make(map[int]int)
    sum2index[0] = -1
    prefixsum, res := 0, 0 
    for i, v := range nums {
        if v == 0 {
            prefixsum--
        } else {
            prefixsum++
        }
        if j, ok := sum2index[prefixsum]; ok {
            if i-j > res {
                res = i - j
            }
        } else {
            sum2index[prefixsum] = i
        }
    }
    return res
}
```

(2) [Subarray Sum Equals K](https://leetcode.com/problems/subarray-sum-equals-k/)

```go
func subarraySum(nums []int, k int) int {
    res, sum := 0, 0
    presum := make(map[int]int)
    presum[0] = 1
    for i := range nums {
        sum += nums[i]
        if _, ok := presum[sum-k]; ok {
            res += presum[sum-k]
        }
        presum[sum]++
    }
    return res
}
```

## Backtracking 

### Template 

Backtracking can be seen as an optimized way to brute force. Brute force approaches evaluate every possibility. In backtracking *you stop evaluating a possibility as soon it breaks some constraint provided in the problem, take a step back and keep trying other possible cases*, see if those lead to a valid solution.

The problems that can be solved using this tool generally satisfy the following criteria :

1. You are explicitly asked to return *a collection of all answers*.
2. You are concerned with what the actual solutions are rather than say the most optimum value of some parameter. (if it were the latter it’s most likely DP or greedy).

We usually use recursive backtracking to solve problems. 

### Problems 

(1) [Subsets](https://leetcode.com/problems/subsets/)

![img](https://cdn-images-1.medium.com/max/1100/1*ddhF2JWfmEl8yLwjx5vt7Q.jpeg)

```go
func subsets(nums []int) [][]int {
	powerset, subset := make([][]int, 0), make([]int, 0)
	backtrack(&powerset, &subset, nums, 0)
	return powerset
}

func backtrack(powerset *[][]int, subset *[]int, nums []int, start int) {
	// WARN: subset is a pointer so the program will go wrong 
	// if we use it directly because we will modify it later
	s := make([]int, len(*subset))
	copy(s, *subset)
	*powerset = append(*powerset, s)
	for i := start; i < len(nums); i++ {
		*subset = append(*subset, nums[i])
        // We let i increments before we step forward
        // therefore we won't find duplicate subsets
		backtrack(powerset, subset, nums, i+1)
		*subset = (*subset)[:len(*subset)-1]
	}
}
```

In this problem, we step back when `i`is out of range. 

 (2) [Subsets II](https://leetcode.com/problems/subsets-ii/)

Same problem with the added constraint that the set may contain duplicates but the output power set should not contain duplicate subsets.

```go
func subsetsWithDup(nums []int) [][]int {
    sort.Ints(nums)
	powerset, subset := make([][]int, 0), make([]int, 0)
	backtrack(&powerset, &subset, nums, 0)
	return powerset
}

func backtrack(powerset *[][]int, subset *[]int, nums []int, start int) {
	// WARN: subset is a pointer so the program will go wrong
	// if we use it directly because we will modify it later
	s := make([]int, len(*subset))
	copy(s, *subset)
	*powerset = append(*powerset, s)
	for i := start; i < len(nums); i++ {
        // Skip duplicates ONLY WHEN i > start
		if i > start && nums[i] == nums[i-1] {
			continue
		}
		*subset = append(*subset, nums[i])
		backtrack(powerset, subset, nums, i+1)
		*subset = (*subset)[:len(*subset)-1]
	}
}
```

(3) [Combination Sum](https://leetcode.com/problems/combination-sum/description/)

The slight difference is we use every candidate at least once. 

```go
func combinationSum(candidates []int, target int) [][]int {
    res, comb := make([][]int, 0), make([]int, 0)
    backtrack(&res, &comb, candidates, target, 0)
    return res
}

func backtrack(res *[][]int, comb *[]int, candidates []int, target, start int) {
    if target == 0 {
        // Found a valid combination 
        c := make([]int, len(*comb))
        copy(c, *comb)
        *res = append(*res, c)
        return
    } else if target < 0 {
        // Found a invalid combination 
        return
    } else {
        for i := start; i < len(candidates); i++ {
            *comb = append(*comb, candidates[i])
            // i doesn't increment before we step forward 
            // so candidates[i] will be used at least once 
            backtrack(res, comb, candidates, target-candidates[i], i)
            *comb = (*comb)[:len(*comb)-1]
        }
    }
}
```

(4) [N-Queens](https://leetcode.com/problems/n-queens/description/)

We try placing queens column by column. Place a queen, go to next column and try placing another queen such that it doesn’t face a queen in the same row or diagonals, and keep going. If we encounter an invalid spot we backtrack and keep trying other spots in that column vertically.

```go
func solveNQueens(n int) [][]string {
	ans := make([][]string, 0)
	board := make([]string, n)
    // Initialize the board
	for i := range board {
        board[i] = strings.Repeat(".", n)
	}
	solve(&ans, &board, 0, n)
	return ans
}

func solve(ans *[][]string, board *[]string, col int, n int) {
	if col == n {
        // Found a solution
		b := make([]string, len(*board))
		copy(b, *board)
		*ans = append(*ans, b)
		return
	}
	for row := 0; row < n; row++ {
		if validate(*board, row, col) {
			(*board)[row] = (*board)[row][:col] + "Q" + (*board)[row][1+col:]
			solve(ans, board, col+1, n)
			(*board)[row] = (*board)[row][:col] + "." + (*board)[row][1+col:]
		}
	}
}

func validate(board []string, row, col int) bool {
	n := len(board)
	// Check the same row
	for i := 1; i <= col; i++ {
		if board[row][col-i] == 'Q' {
			return false
		}
	}
	// Check 45° diagonal
	for i := 1; row-i >= 0 && col-i >= 0; i++ {
		if board[row-i][col-i] == 'Q' {
			return false
		}
	}
    // Check 135° diagonal
	for i := 1; row+i < n && col-i >= 0; i++ {
		if board[row+i][col-i] == 'Q' {
			return false
		}
	}
	return true
}
```

