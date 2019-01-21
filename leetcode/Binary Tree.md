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

Time complexity: $$O(n)$$, n is the number of nodes.

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

#### 2.[My Calendar II](https://leetcode.com/problems/my-calendar-ii)

Implement a `MyCalendarTwo` class to store your events. A new event can be added if adding the event will not cause a **triple** booking.

Your class will have one method, `book(int start, int end)`. Formally, this represents a booking on the half open interval `[start, end)`, the range of real numbers `x` such that `start <= x < end`.

A *triple booking* happens when **three** events have some non-empty intersection (ie., there is some time that is common to all 3 events.)

For each call to the method `MyCalendar.book`, return `true` if the event can be added to the calendar successfully without causing a **triple** booking. Otherwise, return `false` and do not add the event to the calendar.

Your class will be called like this: 

```
MyCalendar cal = new MyCalendar();
```

```
MyCalendar.book(start, end)
```

**Example 1:**

```
MyCalendar();
MyCalendar.book(10, 20); // returns true
MyCalendar.book(50, 60); // returns true
MyCalendar.book(10, 40); // returns true
MyCalendar.book(5, 15); // returns false
MyCalendar.book(5, 10); // returns true
MyCalendar.book(25, 55); // returns true
Explanation: 
The first two events can be booked.  The third event can be double booked.
The fourth event (5, 15) can't be booked, because it would result in a triple booking.
The fifth event (5, 10) can be booked, as it does not use time 10 which is already double booked.
The sixth event (25, 55) can be booked, as the time in [25, 40) will be double booked with the third event;
the time [40, 50) will be single booked, and the time [50, 55) will be double booked with the second event.
```

**Note:**

The number of calls to `MyCalendar.book` per test case will be at most `1000`.

In calls to `MyCalendar.book(start, end)`, `start` and `end` are integers in the range `[0, 10^9]`.

**Solution**

Maintain a list of bookings and a list of double bookings. When booking a new event `[start, end)`, if it conflicts with a double booking, it will have a triple booking and be invalid. Otherwise, parts that overlap the calendar will be a double booking.

Evidently, two events `[s1, e1)` and `[s2, e2)` do *not* conflict if and only if one of them starts after the other one ends: either `e1 <= s2` OR `e2 <= s1`. By De Morgan's laws, this means the events conflict when `s1 < e2` AND `s2 < e1`.

If our event conflicts with a double booking, it's invalid. Otherwise, we add conflicts with the calendar to our double bookings, and add the event to our calendar.

```go
/**
 * Definition for singly-linked list.
 */
type MyCalendarTwo struct {
	Events [][]int
	// 2-booking overlap intervals
	Overlaps [][]int
}

func Constructor() MyCalendarTwo {
	return MyCalendarTwo{make([][]int, 0), make([][]int, 0)}
}

func (this *MyCalendarTwo) Book(start int, end int) bool {
	// if this will generate a triple booking
	for _, v := range this.Overlaps {
		if v[0] < end && start < v[1] {
			return false
		}
	}

	// update 2-booking overlap intervals
	for _, v := range this.Events {
		if v[0] < end && start < v[1] {
			var s int
			if start < v[0] {
				s = v[0]
			} else {
				s = start
			}
			var e int
			if end < v[1] {
				e = end
			} else {
				e = v[1]
			}
			this.Overlaps = append(this.Overlaps, []int{s, e})
		}
	}
	this.Events = append(this.Events, []int{start, end})
	return true
}
```

Time Complexity: $$O(N^2)$$, where $$N$$ is the number of events booked. For each new event, we process every previous event to decide whether the new event can be booked. This leads to $$\sum_k^N O(k) = O(N^2)$$ complexity.

#### 3. [Range Sum Query - Mutable](https://leetcode.com/problems/range-sum-query-mutable/)

Given an integer array *nums*, find the sum of the elements between indices *i* and *j* (*i* ≤ *j*), inclusive.

The *update(i, val)* function modifies *nums* by updating the element at index *i* to *val*.

**Example:**

```
Given nums = [1, 3, 5]

sumRange(0, 2) -> 9
update(1, 2)
sumRange(0, 2) -> 8
```

**Note:**

1. The array is only modifiable by the *update* function.
2. You may assume the number of calls to *update* and *sumRange* function is distributed evenly.

**Solution**

Binary Indexed Trees (BIT or Fenwick tree):

Example: given an array a[0]...a[7], we use a array BIT[9] to represent a tree, where index [2] is the parent of [1] and [3], [6] is the parent of [5] and [7], [4] is the parent of [2] and [6], and [8] is the parent of [4]. i.e. BIT[] as a binary tree:

```
	 * BIT[] as a binary tree:
	 *            ______________*
	 *            ______*
	 *            __*     __*
	 *            *   *   *   *
	 * indices: 0 1 2 3 4 5 6 7 8
```

```
if BIT[i] is a left child then
	BIT[i] = the partial sum from its left most descendant to itself
else 
	BIT[i] = the partial sum from its parent (exclusive) to itself
	
e.g.
	 BIT[1]=a[0]
	 BIT[2]=a[1]+a[0]=a[1]+BIT[1]
	 BIT[3]=a[2]
	 BIT[4]=a[3]+a[2]+a[1]+a[0]=a[3]+BIT[3]+BIT[2]
	 BIT[6]=a[5]+a[4]=a[5]+BIT[5]
	 BIT[8]=a[7]+a[6]+...+a[0]=a[7]+BIT[7]+BIT[6]+BIT[4]
```

> Why use BIT?
>
> In a flat array of $${\displaystyle n}$$ numbers, you can either store the elements, or the prefix sums. In the first case, computing prefix sums requires linear time; in the second case, updating the array elements requires linear time (in both cases, the other operation can be performed in constant time). Fenwick trees allow both operations to be performed in $${\displaystyle O(\log n)}$$ time. This is achieved by representing the numbers as a [tree](https://en.wikipedia.org/wiki/Tree_(data_structure)), where the value of each node is the sum of the numbers in that subtree.

If we update a node in BIT, such as `BIT[2]`,we shall update `BIT[2], BIT[4], BIT[8]`, i.e., for current `[i]`, the next update `[j]` is `j=i+(i&-i)` (double the last 1 bit from `[i]`).

If we want to get the partial sum up to `BIT[7]`, we shall get the sum of `BIT[7], BIT[6], BIT[4]`, i.e., for current `[i]`, the next summand `[j]` is `j=i-(i&-i)` ( delete the last 1 bit from `[i]`).

To obtain the original value of `a[7]`(corresponding to index `[8]` of BIT), we have to subtract `BIT[7], BIT[6], BIT[4]` from `BIT[8]`, i.e. starting from `[idx-1]`, for current `[i]`, the next subtrahend `[j]` is `j=i-(i&-i)`, up to `idx-(idx&-idx)` exclusive. (However, a quicker way but using extra space is to store the original array.)

```go
type NumArray struct {
	originals []int
	BIT       []int
}

func Constructor(nums []int) NumArray {
	BIT := make([]int, len(nums)+1, len(nums)+1)
	for i, n := range nums {
		initBIT(i, n, &BIT)
	}
	return NumArray{nums, BIT}
}

func initBIT(i, val int, BIT *[]int) {
	i++
	for i <= len(*BIT)-1 {
		(*BIT)[i] += val
		i += i & -i
	}
}

func (this *NumArray) Update(i int, val int) {
	diff := val - this.originals[i]
	this.originals[i] = val
	initBIT(i, diff, &this.BIT)
}

func (this *NumArray) SumRange(i int, j int) int {
	return getSum(j, this.BIT) - getSum(i-1, this.BIT)
}

func getSum(i int, BIT []int) int {
	sum := 0
	i++
	for i > 0 {
		sum += BIT[i]
		i -= i & -i
	}
	return sum
}
```

Time complexity: 

(1) `Constructor`: $$O(nlogn)$$, cuz the time complexity of  `initBIT` is $$O(logn)$$, n is the length pf original array.

(2) Others: $$O(logn)$$, n is the length pf original array.

#### 4. [My Calendar III](https://leetcode.com/problems/my-calendar-iii/)

Implement a `MyCalendarThree` class to store your events. A new event can **always** be added.

Your class will have one method, `book(int start, int end)`. Formally, this represents a booking on the half open interval `[start, end)`, the range of real numbers `x` such that `start <= x < end`.

A *K-booking* happens when **K** events have some non-empty intersection (ie., there is some time that is common to all K events.)

For each call to the method `MyCalendar.book`, return an integer `K` representing the largest integer such that there exists a `K`-booking in the calendar.

Your class will be called like this: 

```
MyCalendarThree cal = new MyCalendarThree();
```

```
MyCalendarThree.book(start, end)
```

**Example 1:**

```
MyCalendarThree();
MyCalendarThree.book(10, 20); // returns 1
MyCalendarThree.book(50, 60); // returns 1
MyCalendarThree.book(10, 40); // returns 2
MyCalendarThree.book(5, 15); // returns 3
MyCalendarThree.book(5, 10); // returns 3
MyCalendarThree.book(25, 55); // returns 3
Explanation: 
The first two events can be booked and are disjoint, so the maximum K-booking is a 1-booking.
The third event [10, 40) intersects the first event, and the maximum K-booking is a 2-booking.
The remaining events cause the maximum K-booking to be only a 3-booking.
Note that the last event locally causes a 2-booking, but the answer is still 3 because
eg. [10, 20), [10, 40), and [5, 15) are still triple booked.
```

**Note:**

- The number of calls to `MyCalendarThree.book` per test case will be at most `400`.
- In calls to `MyCalendarThree.book(start, end)`, `start`and `end` are integers in the range `[0, 10^9]`.

**Solution**

This is to find the maximum number of concurrent ongoing event at any time.

We can log the `start` & `end` of each event on the timeline, each `start` add a new ongoing event at that time, each `end` terminate an ongoing event. Then we can scan the timeline to figure out the maximum number of ongoing event at any time.

```java
class MyCalendarThree {
    // use TreeMap to save time spots in order 
    private Map<Integer, Integer> timeline;

    public MyCalendarThree() {
        timeline = new TreeMap<>();
    }

    public int book(int start, int end) {
        timeline.put(start, timeline.getOrDefault(start, 0) + 1);
        timeline.put(end, timeline.getOrDefault(end, 0) - 1);
        int on = 0; // the number of ongoing events
        int K = 0;
        for (int v : timeline.values()) {
            on += v;
            K = Math.max(K, on);
        }
        return K;
    }
}
```

Time complexity: $$O(n)$$, n is the number of time spots.

#### 5. [Add One Row to Tree](https://leetcode.com/problems/add-one-row-to-tree/)

Given the root of a binary tree, then value `v` and depth `d`, you need to add a row of nodes with value `v` at the given depth `d`. The root node is at depth 1.

The adding rule is: given a positive integer depth `d`, for each NOT null tree nodes `N` in depth `d-1`, create two tree nodes with value `v` as `N's` left subtree root and right subtree root. And `N's` **original left subtree** should be the left subtree of the new left subtree root, its **original right subtree** should be the right subtree of the new right subtree root. If depth `d` is 1 that means there is no depth d-1 at all, then create a tree node with value **v** as the new root of the whole original tree, and the original tree is the new root's left subtree.

**Example 1:**

```
Input: 
A binary tree as following:
       4
     /   \
    2     6
   / \   / 
  3   1 5   

v = 1

d = 2

Output: 
       4
      / \
     1   1
    /     \
   2       6
  / \     / 
 3   1   5   
```

**Example 2:**

```
Input: 
A binary tree as following:
      4
     /   
    2    
   / \   
  3   1    

v = 1

d = 3

Output: 
      4
     /   
    2
   / \    
  1   1
 /     \  
3       1
```

**Note:**

1. The given d is in range [1, maximum depth of the given tree + 1].
2. The given binary tree has at least one tree node.

**Solution**

(1) level order traversal

```go
/**
 * Definition for a binary tree node.
 * type TreeNode struct {
 *     Val int
 *     Left *TreeNode
 *     Right *TreeNode
 * }
 */
func addOneRow(root *TreeNode, v int, d int) *TreeNode {
	if d == 1 {
		newRoot := &TreeNode{v, root, nil}
		return newRoot
	}

    // find all not null nodes on level d-1 and modify them
	nodes := visitNodesOnLevel(root, d-1)
	for _, node := range nodes {
		oldLeft := node.Left
		newLeft := &TreeNode{v, oldLeft, nil}
		node.Left = newLeft
		oldRight := node.Right
		newRight := &TreeNode{v, nil, oldRight}
		node.Right = newRight
	}
	return root
}

// find all not null nodes on a specific level
func visitNodesOnLevel(root *TreeNode, target int) []*TreeNode {
	res := make([]*TreeNode, 0)
	if root != nil {
        // level 1 is just the root
		if target == 1 {
			res = append(res, root)
		} else {
			levelOrder(root, target, 1, &res)
		}
	}
	return res
}

func levelOrder(node *TreeNode, target, current int, res *[]*TreeNode) {
	if node == nil {
		return
	} else if target == current+1 {
		if node.Left != nil {
			*res = append(*res, node.Left)
		}
		if node.Right != nil {
			*res = append(*res, node.Right)
		}
	} else {
		levelOrder(node.Left, target, current+1, res)
		levelOrder(node.Right, target, current+1, res)
	}
}
```

Time complexity: $$O(n)$$

(2)

In addition to use `1` to indicate `attach to left node` as required, we can also use `0` to indicate `attach to right node`.

```java
public class Solution {
    public TreeNode addOneRow(TreeNode root, int v, int d) {
        if (d == 0 || d == 1) {
            TreeNode newroot = new TreeNode(v);
            newroot.left = d == 1 ? root : null;
            newroot.right = d == 0 ? root : null;
            return newroot;
        }
        if (root != null && d >= 2) {
            root.left  = addOneRow(root.left,  v, d > 2 ? d - 1 : 1);
            root.right = addOneRow(root.right, v, d > 2 ? d - 1 : 0);
        }
        return root;
    }
}
```

Time complexity: $$O(n)$$

#### 6. [All Nodes Distance K in Binary Tree](https://leetcode.com/problems/all-nodes-distance-k-in-binary-tree/)

We are given a binary tree (with root node `root`), a `target` node, and an integer value `K`.

Return a list of the values of all nodes that have a distance `K` from the `target` node.  The answer can be returned in any order.

**Example 1:**

![img](https://s3-lc-upload.s3.amazonaws.com/uploads/2018/06/28/sketch0.png)

```
Input: root = [3,5,1,6,2,0,8,null,null,7,4], target = 5, K = 2

Output: [7,4,1]

Explanation: 
The nodes that are a distance 2 from the target node (with value 5)
have values 7, 4, and 1.

Note that the inputs "root" and "target" are actually TreeNodes.
The descriptions of the inputs above are just serializations of these objects.
```

**Note:**

1. The given tree is non-empty.
2. Each node in the tree has unique values `0 <= node.val <= 500`.
3. The `target` node is a node in the tree.
4. `0 <= K <= 1000`.

**Solution**

Transfer the tree to a undirected graph then search vertices by BFS.

```go
func distanceK(root *TreeNode, target *TreeNode, K int) []int {
	res := make([]int, 0)
	if root == nil || K < 0 {
		return res
	}
	
	graph := make(map[*TreeNode][]*TreeNode)
	constructGraph(root, nil, graph)
	if _, ok := graph[target]; !ok {
		return res
	}
	// BFS
	visited := make(map[*TreeNode]bool)
	queue := make([]*TreeNode, 0)
	queue = append(queue, target)
	visited[target] = true
	for len(queue) > 0 {
		if K == 0 {
			for len(queue) > 0 {
				res = append(res, queue[0].Val)
				queue = queue[1:]
			}
			return res
		}
		size := len(queue)
		for i := 0; i < size; i++ {
			node := queue[0]
			queue = queue[1:]
			for _, next := range graph[node] {
				if visited[next] {
					continue
				}
				visited[next] = true
				queue = append(queue, next)
			}
		}
		K--
	}
	return res
}

// Transfer a binary tree to a undirected graph. 
func constructGraph(node *TreeNode, parent *TreeNode, graph map[*TreeNode][]*TreeNode) {
	if node == nil {
		return
	}
	if _, ok := graph[node]; !ok {
		graph[node] = make([]*TreeNode, 0)
		if parent != nil {
			graph[node] = append(graph[node], parent)
			graph[parent] = append(graph[parent], node)
		}
		constructGraph(node.Left, node, graph)
		constructGraph(node.Right, node, graph)
	}
}
```

Time complexity: `constructGraph`costs $$O(V)$$, `distanceK` costs $$O(V+ E)$$.

#### 7. [Binary Tree Level Order Traversal](https://leetcode.com/problems/binary-tree-level-order-traversal)

Given a binary tree, return the *level order*traversal of its nodes' values. (ie, from left to right, level by level).

For example:
Given binary tree `[3,9,20,null,null,15,7]`,

```
    3
   / \
  9  20
    /  \
   15   7
```

return its level order traversal as:

```
[
  [3],
  [9,20],
  [15,7]
]
```

**Solution**

```go
/**
 * Definition for a binary tree node.
 * type TreeNode struct {
 *     Val int
 *     Left *TreeNode
 *     Right *TreeNode
 * }
 */
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

Time complexity: $$O(n)$$, n is the number of nodes in the whole tree.

#### 8. [Check Completeness of a Binary Tree](https://leetcode.com/problems/check-completeness-of-a-binary-tree/)

Given a binary tree, determine if it is a *complete binary tree*.

**Definition of a complete binary tree from Wikipedia:**
In a complete binary tree every level, except possibly the last, is completely filled, and all nodes in the last level are as far left as possible. It can have between 1 and 2h nodes inclusive at the last level h.

**Example 1:**

**![img](https://assets.leetcode.com/uploads/2018/12/15/complete-binary-tree-1.png)**

```
Input: [1,2,3,4,5,6]
Output: true
Explanation: Every level before the last is full (ie. levels with node-values {1} and {2, 3}), and all nodes in the last level ({4, 5, 6}) are as far left as possible.
```

**Example 2:**

**![img](https://assets.leetcode.com/uploads/2018/12/15/complete-binary-tree-2.png)**

```
Input: [1,2,3,4,5,null,7]
Output: false
Explanation: The node with value 7 isn't as far left as possible.
```

**Note:**

1. The tree will have between 1 and 100 nodes.

**Solution**

The properties of binary tree show that if we assign the root to `i`, the left child will be `2*i+1`and the right child will be `2+i+2`. A complete binary tree can be represented by an array `[0,1,...,n-1]`, which n is the number of nodes minus 1.

```go
/**
 * Definition for a binary tree node.
 * type TreeNode struct {
 *     Val int
 *     Left *TreeNode
 *     Right *TreeNode
 * }
 */
func isCompleteTree(root *TreeNode) bool {
    return checkComplete(root, 0, countNodes(root))
}

func countNodes(root *TreeNode) int {
    if root == nil {
        return 0
    }
    return 1 + countNodes(root.Left) + countNodes(root.Right)
}

func checkComplete(root *TreeNode, index, numberOfNodes int) bool {
    if root == nil {
        return true
    }
    
    if index >= numberOfNodes {
        return false
    }
    
    return checkComplete(root.Left, 2*index+1, numberOfNodes) && checkComplete(root.Right, 2*index+2, numberOfNodes)
}
```

Time complexity: $$O(n)$$, n is the number of nodes.

#### 9. [Construct Binary Tree from Inorder and Postorder Traversal](https://leetcode.com/problems/construct-binary-tree-from-inorder-and-postorder-traversal/)

Given inorder and postorder traversal of a tree, construct the binary tree.

**Note:**
You may assume that duplicates do not exist in the tree.

For example, given

```
inorder = [9,3,15,20,7]
postorder = [9,15,7,20,3]
```

Return the following binary tree:

```
    3
   / \
  9  20
    /  \
   15   7
```

**Solution**

The the basic idea is to take the last element in postorder array as the root, find the position of the root in the inorder array; then locate the range for left sub-tree and right sub-tree and do recursion. Use a HashMap to record the index of roots in the inorder array.

```kotlin
/**
 * Definition for a binary tree node.
 * class TreeNode(var `val`: Int = 0) {
 *     var left: TreeNode? = null
 *     var right: TreeNode? = null
 * }
 */
class Solution {
    fun buildTree(inorder: IntArray, postorder: IntArray): TreeNode? {
        if (inorder.isEmpty() || postorder.isEmpty() || inorder.size != postorder.size) {
            return null
        }
        val map = mutableMapOf<Int, Int>()
        for (i in inorder.indices) {
            map[inorder[i]] = i
        }
        return build(inorder, 0, inorder.size - 1, postorder, 0, postorder.size - 1, map)
    }

    private fun build(
        inorder: IntArray,
        inStart: Int,
        inEnd: Int,
        postorder: IntArray,
        postStart: Int,
        postEnd: Int,
        map: MutableMap<Int, Int>
    ): TreeNode? {
        if (postStart > postEnd || inStart > inEnd) {
            return null
        }

        val root = TreeNode(postorder[postEnd])
        val indexOfRoot = map[postorder[postEnd]]
        root.left =
                build(
                    inorder,
                    inStart,
                    indexOfRoot!! - 1,
                    postorder,
                    postStart,
                    postStart + indexOfRoot - inStart - 1,
                    map
                )
        root.right =
                build(
                    inorder,
                    indexOfRoot + 1,
                    inEnd,
                    postorder,
                    postStart + indexOfRoot - inStart,
                    postEnd - 1,
                    map
                )
        return root
    }
}
```

Time complexity: $$O(n)​$$

#### 10. [Construct Binary Tree from Preorder and Inorder Traversal](https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)

Given preorder and inorder traversal of a tree, construct the binary tree.

**Note:**
You may assume that duplicates do not exist in the tree.

For example, given

```
preorder = [3,9,20,15,7]
inorder = [9,3,15,20,7]
```

Return the following binary tree:

```
    3
   / \
  9  20
    /  \
   15   7
```

**Solution**

(1) 

```go
/**
 * Definition for a binary tree node.
 * type TreeNode struct {
 *     Val int
 *     Left *TreeNode
 *     Right *TreeNode
 * }
 */
func buildTree(preorder []int, inorder []int) *TreeNode {
	return build(&preorder, inorder)
}

func build(preorder *[]int, inorder []int) *TreeNode {
	if len(inorder) == 0 {
		return nil
	}

	rootVal := (*preorder)[0]
	*preorder = (*preorder)[1:]
	var indexOfRoot int
	for i, v := range inorder {
		if v == rootVal {
			indexOfRoot = i
			break
		}
	}
	return &TreeNode{
		rootVal,
		build(preorder, inorder[0:indexOfRoot]),
		build(preorder, inorder[indexOfRoot+1:]),
	}
}
```

Time complexity: $$O(n)​$$

(2) 

```kotlin
/**
 * Definition for a binary tree node.
 * class TreeNode(var `val`: Int = 0) {
 *     var left: TreeNode? = null
 *     var right: TreeNode? = null
 * }
 */
class Solution {
    fun buildTree(preorder: IntArray, inorder: IntArray): TreeNode? {
        return build(0, 0, inorder.size - 1, preorder, inorder)
    }

    private fun build(preStart: Int, inStart: Int, inEnd: Int, preorder: IntArray, inorder: IntArray): TreeNode? {
        if (preStart > preorder.size - 1 || inStart > inEnd) {
            return null
        }

        val root = TreeNode(preorder[preStart])
        val indexOfRoot = inorder.indexOf(root.`val`)
        root.left = build(preStart + 1, inStart, indexOfRoot - 1, preorder, inorder)
        root.right = build(preStart + 1 + indexOfRoot - inStart, indexOfRoot + 1, inEnd, preorder, inorder)
        return root
    }
}
```

Time complexity: $$O(n)$$

#### 11. [Construct Binary Tree from Preorder and Postorder Traversal](https://leetcode.com/problems/construct-binary-tree-from-preorder-and-postorder-traversal/)

Return any binary tree that matches the given preorder and postorder traversals.

Values in the traversals `pre` and `post` are distinct positive integers.

**Example 1:**

```
Input: pre = [1,2,4,5,3,6,7], post = [4,5,2,6,7,3,1]
Output: [1,2,3,4,5,6,7]
```

**Note:**

- `1 <= pre.length == post.length <= 30`
- `pre[]` and `post[]` are both permutations of `1, 2, ..., pre.length`.
- It is guaranteed an answer exists. If there exists multiple answers, you can return any of them.

(1) recursion

```
[root][......left......][...right..]  ---pre
[......left......][...right..][root]  ---post
```

```go
func constructFromPrePost(pre []int, post []int) *TreeNode {
	valToIdx := make(map[int]int)
	for i, v := range post {
		valToIdx[v] = i
	}
	return construct(pre, post, 0, len(post)-1, 0, len(post)-1, valToIdx)
}

func construct(pre []int, post []int, preS int, preE int, postS int, postE int, valToIdx map[int]int) *TreeNode {
	n := &TreeNode{Val: pre[preS]}
	// no subtree
	if preS == preE {
		return n
	}
	subTreeRootVal := pre[preS+1]
	indexOfRoot := valToIdx[subTreeRootVal]
	l := indexOfRoot - postS + 1
	n.Left = construct(pre, post, preS+1, preS+l, postS, postS+l-1, valToIdx)
	// no right child because post[postE] indicates the root of current subtree
	if indexOfRoot+1 == postE {
		return n
	}
	n.Right = construct(pre, post, preS+l+1, preE, indexOfRoot+1, postE-1, valToIdx)
	return n
}
```

Time complexity: $$O(n)$$, we use a hashmap to reduce complexity.

(2) iteration

We will **preorder** generate TreeNodes, push them to `stack` and **postorder** pop them out.

1. Loop on `pre` array and construct node one by one.
2. `stack` save the current path of tree.
3. `node = new TreeNode(pre[i])`, if not left child, add node to the left. otherwise add it to the right.
4. If we meet a same value in the pre and post, it means we complete the construction for current subtree. We pop it from `stack`.

```go
/**
 * Definition for a binary tree node.
 * type TreeNode struct {
 *     Val int
 *     Left *TreeNode
 *     Right *TreeNode
 * }
 */
func constructFromPrePost(pre []int, post []int) *TreeNode {
	stack := make([]*TreeNode, 0)
	stack = append(stack, &TreeNode{Val: pre[0]})
	j := 0
	for _, v := range pre[1:] {
		node := &TreeNode{Val: v}
		for stack[len(stack)-1].Val == post[j] {
			stack = stack[:len(stack)-1]
			j++
		}
		if stack[len(stack)-1].Left == nil {
			stack[len(stack)-1].Left = node
		} else {
			stack[len(stack)-1].Right = node
		}
		stack = append(stack, node)
	}
	return stack[0]
}
```

Time complexity: $$O(n)​$$

#### 12. [Count Complete Tree Nodes](https://leetcode.com/problems/count-complete-tree-nodes)

Given a **complete** binary tree, count the number of nodes.

**Note:**

**Definition of a complete binary tree from Wikipedia:**
In a complete binary tree every level, except possibly the last, is completely filled, and all nodes in the last level are as far left as possible. It can have between 1 and 2h nodes inclusive at the last level h.

**Example:**

```
Input: 
    1
   / \
  2   3
 / \  /
4  5 6

Output: 6
```

**Solution**

(1) straightforward 

```go
/**
 * Definition for a binary tree node.
 * type TreeNode struct {
 *     Val int
 *     Left *TreeNode
 *     Right *TreeNode
 * }
 */
func countNodes(root *TreeNode) int {
    if root == nil {
        return 0
    }
    var count int
    traverse(root, &count)
    return count 
}

func traverse(node *TreeNode, count *int) {
    if node == nil {
        return
    }
    (*count)++
    traverse(node.Left, count)
    traverse(node.Right, count)
}
```

Time complexity: $$O(n)$$, n is the number of nodes.

(2) 

The height of a tree can be found by just going left. Let a single node tree have height 0. Find the height `h` of the whole tree. If the whole tree is empty, i.e., has height -1, there are 0 nodes.

Otherwise check whether the height of the right subtree is just one less than that of the whole tree, meaning left and right subtree have the same height.

- If yes, then the last node on the last tree row is in the right subtree and the left subtree is a full tree of height h-1. So we take the 2^h-1 nodes of the left subtree plus the 1 root node plus recursively the number of nodes in the right subtree.
- If no, then the last node on the last tree row is in the left subtree and the right subtree is a full tree of height h-2. So we take the 2^(h-1)-1 nodes of the right subtree plus the 1 root node plus recursively the number of nodes in the left subtree.

```kotlin
class Solution {
    fun countNodes(root: TreeNode?): Int {
        val h = height(root)
        return when {
            h < 0 -> 0
            // bit operations can speed the calculations
            height(root?.right) == h - 1 -> (1 shl h) + countNodes(root?.right)
            else -> (1 shl h - 1) + countNodes(root?.left)
        }
    }

    private fun height(root: TreeNode?): Int {
        return if (root == null) -1 else 1 + height(root.left)
    }
}
```

Time complexity: Since we halve the tree in every recursive step, we have $$O(log(n))$$ steps. Finding a height costs $$O(log(n))$$. So overall $$O(log(n)^2)$$.

#### 13. [Delete Node in a BST](https://leetcode.com/problems/delete-node-in-a-bst/)

Given a root node reference of a BST and a key, delete the node with the given key in the BST. Return the root node reference (possibly updated) of the BST.

Basically, the deletion can be divided into two stages:

1. Search for a node to remove.
2. If the node is found, delete the node.

**Note:** Time complexity should be O(height of tree).

**Example:**

```
root = [5,3,6,2,4,null,7]
key = 3

    5
   / \
  3   6
 / \   \
2   4   7

Given key to delete is 3. So we find the node with value 3 and delete it.

One valid answer is [5,4,6,2,null,null,7], shown in the following BST.

    5
   / \
  4   6
 /     \
2       7

Another valid answer is [5,2,6,null,4,null,7].

    5
   / \
  2   6
   \   \
    4   7
```

**Solution**

(1) recursion 

```go
/**
 * Definition for a binary tree node.
 * type TreeNode struct {
 *     Val int
 *     Left *TreeNode
 *     Right *TreeNode
 * }
 */

// Algorithm, 4nd Edition 
func deleteNode(root *TreeNode, key int) *TreeNode {
    return deleteHelper(root, key)
}

func deleteHelper(node *TreeNode, key int) *TreeNode {
    if node == nil {
        return nil
    }
    
    if key < node.Val {
        node.Left = deleteHelper(node.Left, key)
    } else if key > node.Val {
        node.Right = deleteHelper(node.Right, key)
    } else {
        if node.Left == nil {
            return node.Right
        }
        if node.Right == nil {
            return node.Left
        }
        // replace the deleted node with the minimum node of its right subtree
        tmp := node
        node = min(tmp.Right)
        node.Right = deleteMin(tmp.Right)
        node.Left = tmp.Left
    }
    return node
}

func min(node *TreeNode) *TreeNode {
    if node.Left == nil {
        return node
    }
    return min(node.Left)
}

func deleteMin(node *TreeNode) *TreeNode {
    if node.Left == nil {
        return node.Right
    }
    node.Left = deleteMin(node.Left)
    return node
}
```

Time complexity: $$O(h)$$, h is the height of tree.

(2) iteration 

```go
func deleteNode(root *TreeNode, key int) *TreeNode {
    // Find the node to be deleted 
    cur := root
    var pre *TreeNode
    for cur != nil && cur.Val != key {
        pre = cur 
        if key < cur.Val {
            cur = cur.Left
        } else if key > cur.Val {
            cur = cur.Right
        }
    }
    
    if pre == nil {
        return deleteRoot(cur)
    }
    if pre.Left == cur {
        pre.Left = deleteRoot(cur)
    } else {
        pre.Right = deleteRoot(cur)
    }
    return root
}

func deleteRoot(root *TreeNode) *TreeNode {
    if root == nil {
        return nil
    }
    if root.Left == nil {
        return root.Right
    }
    if root.Right == nil {
        return root.Left
    }
    
    next := root.Right
    var pre *TreeNode
    for next.Left != nil {
        pre = next
        next = next.Left
    }
    next.Left = root.Left
    if root.Right != next {
        pre.Left = next.Right
        next.Right = root.Right
    }
    return next
}
```

