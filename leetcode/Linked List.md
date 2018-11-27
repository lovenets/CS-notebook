#### 1.[Add Two Numbers](https://leetcode.com/problems/add-two-numbers/)

You are given two **non-empty** linked lists representing two non-negative integers. The digits are stored in **reverse order** and each of their nodes contain a single digit. Add the two numbers and return it as a linked list.

You may assume the two numbers do not contain any leading zero, except the number 0 itself.

**Example:**

```
Input: (2 -> 4 -> 3) + (5 -> 6 -> 4)
Output: 7 -> 0 -> 8
Explanation: 342 + 465 = 807.
```

**My Solution**

Well, it can work as long as the number will not overflow...

```go
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func addTwoNumbers(l1 *ListNode, l2 *ListNode) *ListNode {
	// get every digit and construct origin numbers
	digits1 := make([]int, 0)
	for n := l1; n != nil; n = n.Next {
		digits1 = append(digits1, n.Val)
	}
	num1 := 0
	for i, v := range digits1 {
		num1 += v*int(math.Pow10(i))
	}
	digits2 := make([]int, 0)
	for n := l2; n != nil; n = n.Next {
		digits2 = append(digits2, n.Val)
	}
	num2 := 0
	for i, v := range digits2 {
		num2 += v*int(math.Pow10(i))
	}

	// add two numbers
	sum := num1 + num2

	// what if sum is 0
	if sum == 0 {
		return &ListNode{0,nil}
	}
	// get every digit to construct the result List
	head := &ListNode{-1,nil}
	pre := new(ListNode)
	for sum > 0 {
		cur := new(ListNode)
		cur.Val = sum % 10
		if head.Val == -1 {
			// initialize the list
			head = cur
		} else {
			pre.Next = cur
		}
		pre = cur
		sum /= 10
	}
	return head
}
```

Time Complexity: $$O(n)$$, n is the length of result list.

**Solution**

Actually, we just need to simulate the process in which we use a pen and a piece of paper to sum two numbers. We start from units, and then tens and so on. When we add two digits of matching position and we get a number greater than 10, we will add 1, which overflows, to the next position. 

```go
func addTwoNumbers(l1 *ListNode, l2 *ListNode) *ListNode {
	// carry will store the overflow digit
	carry := 0
	n := new(ListNode)
	head := n
	for l1 != nil || l2 != nil || carry != 0 {
		var v1 int
		if l1 != nil {
			v1 = l1.Val
			l1 = l1.Next
		}
		var v2 int
		if l2 != nil {
			v2 = l2.Val
			l2 = l2.Next
		}
		n.Next = &ListNode{(v1 + v2 + carry) % 10, nil}
		n = n.Next
		carry = (v1 + v2 + carry) / 10
	}
	return head.Next
}
```

Time Complexity: $$O(n)$$, n is the length of result list.