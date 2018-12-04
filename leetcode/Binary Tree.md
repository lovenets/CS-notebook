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

Brute Force

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

Time complexity: Time Complexity: $$O(N^2)$$, where $$N$$ is the number of events booked. For each new event, we process every previous event to decide whether the new event can be booked. This leads to $$\sum_k^N O(k) = O(N^2)$$ complexity.