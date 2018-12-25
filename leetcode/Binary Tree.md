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