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

#### 2.[Add Two Numbers II](https://leetcode.com/problems/add-two-numbers-ii)

You are given two **non-empty** linked lists representing two non-negative integers. The most significant digit comes first and each of their nodes contain a single digit. Add the two numbers and return it as a linked list.

You may assume the two numbers do not contain any leading zero, except the number 0 itself.

**Follow up:**
What if you cannot modify the input lists? In other words, reversing the lists is not allowed.

**Example:**

```
Input: (7 -> 2 -> 4 -> 3) + (5 -> 6 -> 4)
Output: 7 -> 8 -> 0 -> 7
```

**Solution**

using stack

```go
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func addTwoNumbers(l1 *ListNode, l2 *ListNode) *ListNode {
	// use two stacks to store every digit of two numbers
	digits1 := make([]int, 0)
	digits2 := make([]int, 0)
	for ; l1 != nil; l1 = l1.Next {
		digits1 = append(digits1, l1.Val)
	}
	for ; l2 != nil; l2 = l2.Next {
		digits2 = append(digits2, l2.Val)
	}

	// add two numbers
	cur := new(ListNode)
	sum := 0
	for len(digits1) != 0 || len(digits2) != 0 {
		if len(digits1) != 0 {
			sum += digits1[len(digits1)-1]
			digits1 = digits1[:len(digits1)-1]
		}
		if len(digits2) != 0 {
			sum += digits2[len(digits2)-1]
			digits2 = digits2[:len(digits2)-1]
		}
		cur.Val = sum % 10
		// what if two numbers have the same length
		pre := &ListNode{sum / 10, cur}
		cur = pre
		sum /= 10
	}
	// avoid leading 0
	if cur.Val == 0 {
		return cur.Next
	} else {
		return cur
	}
}
```

Time complexity: $$O(n)$$, n is the length of greater number.

#### 3.[Convert Sorted List to Binary Search Tree](https://leetcode.com/problems/convert-sorted-list-to-binary-search-tree/)

Given a singly linked list where elements are sorted in ascending order, convert it to a height balanced BST.

For this problem, a height-balanced binary tree is defined as a binary tree in which the depth of the two subtrees of *every* node never differ by more than 1.

**Example:**

```
Given the sorted linked list: [-10,-3,0,5,9],

One possible answer is: [0,-3,9,-10,null,5], which represents the following height balanced BST:

      0
     / \
   -3   9
   /   /
 -10  5
```

**Solution**

(1) Recursion

NOTE: **The given list is sorted in ascending order**.

The middle element of the given list would form the root of the binary search tree. All the elements to the left of the middle element would form the left subtree recursively. Similarly, all the elements to the right of the middle element will form the right subtree of the binary search tree. This would ensure the height balance required in the resulting binary search tree.

1. Since we are given a linked list and not an array, we don't really have access to the elements of the list using indexes. We want to know the middle element of the linked list.
2. We can use the two pointer approach for finding out the middle element of a linked list. Essentially, we have two pointers called `slow_ptr` and `fast_ptr`. The `slow_ptr` moves one node at a time whereas the `fast_ptr` moves two nodes at a time. By the time the `fast_ptr` reaches the end of the linked list, the `slow_ptr` would have reached the middle element of the linked list. For an even sized list, any of the two middle elements can act as the root of the BST.
3. Once we have the middle element of the linked list, we disconnect the portion of the list to the left of the middle element. The way we do this is by keeping a `prev_ptr` as well which points to one node before the `slow_ptr` i.e. `prev_ptr.next` = `slow_ptr`. For disconnecting the left portion we simply do `prev_ptr.next = None`
4. We only need to pass the head of the linked list to the function that converts it to a height balances BST. So, we recurse on the left half of the linked list by passing the original head of the list and on the right half by passing `slow_ptr.next` as the head.

```go
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
/**
 * Definition for a binary tree node.
 * type TreeNode struct {
 *     Val int
 *     Left *TreeNode
 *     Right *TreeNode
 * }
 */
func sortedListToBST(head *ListNode) *TreeNode {
    if head == nil {
        return nil
    }
    
    mid := findMiddle(head)
    
    root := &TreeNode{mid.Val,nil,nil}
    
    if head == mid {
        // if the list has only one node
        return root
    } else {
        root.Left = sortedListToBST(head)
        root.Right = sortedListToBST(mid.Next)
    }
    return root
}

func findMiddle(head *ListNode) *ListNode {
    pre,slow,fast := new(ListNode),head,head
    
    for fast != nil && fast.Next != nil {
        pre = slow
        slow = slow.Next
        fast = fast.Next.Next
    }
    
    // cut the list into left and right parts
    if pre != nil {
        pre.Next = nil
    }
    
    return slow
}
```

Time Complexity: 

$$O(NlogN)$$. Suppose our linked list consists of $$N$$ elements. For every list we pass to our recursive function, we have to calculate the middle element for that list. For a list of size $$N$$, it takes $$N / 2$$ steps to find the middle element i.e. $$O(N)$$ to find the mid. We do this for **every** half of the original linked list. From the looks of it, this seems to be an $$O(N^2)$$ algorithm. However, on closer analysis, it turns out to be a bit more efficient than $$O(N^2)$$.

Let's look at the number of operations that we have to perform on each of the halves of the linked list. As we mentioned earlier, it takes $$N/2$$ steps to find the middle of a linked list with NN elements. After finding the middle element, we are left with two halves of size $$N / 2$$ each. Then, we find the middle element for `both` of these halves and it would take a total of $$2 \times N / 4$$ steps for that. And similarly for the smaller sublists that keep forming recursively. This would give us the following series of operations for a list of size $$N$$.

$$N / 2 \; + 2 * N / 4 \; + 4 * N / 8 \; + 8 * N / 16 \; ....$$

**Essentially**, this is done $$logN$$ times since we split the linked list in half every time. Hence, the above equation becomes:

$$\sum_{i = 1}^{i = logN} 2^{i - 1} \times N / 2^i = N / 2 = N / 2 \; logN = O(NlogN)$$

We can also use [master theorem](https://en.wikipedia.org/wiki/Master_theorem_(analysis_of_algorithms)) to analyze it quickly. Obviously, the recurrence relation is $$T(n) = 2T({\frac {n}{2}})+f(n),f(n)=O(n^{log_{2}2}log^0n)=O(n)$$. Hence the time complexity is $$O(n^{log_{2}2}log^1n)=O(nlogn)$$.  

(2) Recursion + Conversion to Array

This approach is a classic example of the time-space tradeoff. More space, less time.

1. Convert the given linked list into an array. Let's call the beginning and the end of the array as `left` and `right`
2. Find the middle element as `(left + right) / 2`. Let's call this element as `mid`. This is a $$O(1)$$ time operation and is the only major improvement over the previous algorithm.
3. The middle element forms the root of the BST.
4. Recursively form binary search trees on the two halves of the array represented by `(left, mid - 1)`and `(mid + 1, right)` respectively.

```go
func sortedListToBST(head *ListNode) *TreeNode {
    vals := convertToArray(head)
    return listToBST(vals,0,len(vals)-1)
}

func convertToArray(head *ListNode) []int {
    vals := make([]int,0)
    for head != nil {
        vals = append(vals,head.Val)
        head = head.Next
    }
    return vals
}

func listToBST(vals []int,beg,end int) *TreeNode {
    if beg > end {
        return nil
    }
    
    mid := (beg + end) / 2
    root := &TreeNode{vals[mid],nil,nil}
    
    if beg == end {
        // if there is only one element in the array
        return root
    } else {    
        root.Left = listToBST(vals,beg,mid-1)
        root.Right = listToBST(vals,mid+1,end)
        return root
    }
}
```

Time Complexity: $$O(N)$$. Since we convert the linked list to an array initially and then we convert the array into a BST. Accessing the middle element now takes $$O(1)$$ time and hence the time complexity comes down.

If we use master theorem, $$T(n) = 2T({\frac {n}{2}})+f(n),f(n)=O(n^{log_{2}2-1})=O(1)$$ hence $$T(n) = O(n^{log_{2}2})=O(n)$$.

(3) Inorder Simulation

As we know, there are three different types of traversals for a binary tree:

- Inorder
- Preorder and
- Postorder traversals.

The inorder traversal on a binary search tree leads to a very interesting outcome. Elements processed in the inorder fashion on a binary search tree turn out to be sorted in ascending order.

We know that the leftmost element in the inorder traversal has to be the head of our given linked list. Similarly, the next element in the inorder traversal will be the second element in the linked list and so on. This is made possible because the initial list given to us is sorted in ascending order.

1. Iterate over the linked list to find out it's length. We will make use of two different pointer variables here to mark the beginning and the end of the list. Let's call them `start` and `end` with their initial values being `0` and `length - 1` respectively.
2. Remember, we have to simulate the inorder traversal here. We can find out the middle element by using `(start + end) / 2`. Note that we don't really find out the middle node of the linked list. We just have a variable telling us the index of the middle element. We simply need this to make recursive calls on the two halves.
3. Recur on the left half by using `start, mid - 1` as the starting and ending points.
4. The invariance that we maintain in this algorithm is that whenever we are done building the left half of the BST, the head pointer in the linked list will point to the root node or the middle node (which becomes the root). So, we simply use the current value pointed to by `head` as the root node and progress the head node by once i.e. `head = head.next`
5. We recur on the right hand side using `mid + 1, end` as the starting and ending points.

```java
/**
 * Definition for singly-linked list. public class ListNode { int val; ListNode next; ListNode(int
 * x) { val = x; } }
 */
/**
 * Definition for a binary tree node. public class TreeNode { int val; TreeNode left; TreeNode
 * right; TreeNode(int x) { val = x; } }
 */
class Solution {

  private ListNode head;

  private int findSize(ListNode head) {
    ListNode ptr = head;
    int c = 0;
    while (ptr != null) {
      ptr = ptr.next;  
      c += 1;
    }
    return c;
  }

  private TreeNode convertListToBST(int l, int r) {
    // Invalid case
    if (l > r) {
      return null;
    }

    int mid = (l + r) / 2;

    // First step of simulated inorder traversal. Recursively form
    // the left half
    TreeNode left = this.convertListToBST(l, mid - 1);

    // Once left half is traversed, process the current node
    TreeNode node = new TreeNode(this.head.val);
    node.left = left;

    // Maintain the invariance mentioned in the algorithm
    this.head = this.head.next;

    // Recurse on the right hand side and form BST out of them
    node.right = this.convertListToBST(mid + 1, r);
    return node;
  }

  public TreeNode 
      sortedListToBST(ListNode head) {
    // Get the size of the linked list first
    int size = this.findSize(head);

    this.head = head;

    // Form the BST now that we know the size
    return convertListToBST(0, size - 1);
  }
}
```

Time Complexity: The time complexity is still $$O(N)$$ since we still have to process each of the nodes in the linked list once and form corresponding BST nodes.

#### 4.[Copy List With Random Pointer](https://leetcode.com/problems/copy-list-with-random-pointer/)

A linked list is given such that each node contains an additional random pointer which could point to any node in the list or null.

Return a deep copy of the list.

**Solution**

Notice that we need to return a **deep** copy of the list, which means that we must create every node again.

 **The idea is to associate the original node with its copy node in a single linked list. In this way, we don't need extra space to keep track of the new nodes.**

The algorithm is composed of the follow three steps which are also 3 iteration rounds.

1. Iterate the original list and duplicate each node. The duplicate
   of each node follows its original immediately.
2. Iterate the new list and assign the random pointer for each
   duplicated node.
3. Restore the original list and extract the duplicated nodes.

```java
/**
 * Definition for singly-linked list with a random pointer.
 * class RandomListNode {
 *     int label;
 *     RandomListNode next, random;
 *     RandomListNode(int x) { this.label = x; }
 * };
 */
public class Solution {
    public RandomListNode copyRandomList(RandomListNode head) {
        RandomListNode iter = head;
        RandomListNode next;

        // 1st round: copy every node and
        // make them linked to their original nodes respectively
        while (iter != null) {
            next = iter.next;
            RandomListNode copy = new RandomListNode(iter.label);
            // make the copy one become its original node's next node
            iter.next = copy;
            copy.next = next;
            // move forward
            iter = next;
        }

        // 2nd round: assign random pointers for the copy ones
        iter = head;
        while (iter != null) {
            if (iter.random != null) {
                // NOTE: iter.next is the copy one
                // and iter.random.next is also a copy copy one
                iter.next.random = iter.random.next;
            }
            // iter.next.next is the original one
            iter = iter.next.next;
        }

        // 3rd round: restore the original list and
        // extract the copy ones
        iter = head;
        RandomListNode pseudoHead = new RandomListNode(0);
        RandomListNode copy;
        RandomListNode copyIter = pseudoHead;
        while (iter != null) {
            // iter.next.next is the original one
            next = iter.next.next;

            // extract the copy one
            copy = iter.next;
            copyIter.next = copy;
            copyIter = copy;
            // restore the original list
            iter.next = next;

            iter = next;
        }

        return pseudoHead.next;
    }
}
```

Time complexity: $$O(n)$$, n is the length of list.

#### 5.[Flatten a Multilevel Doubly Linked List](https://leetcode.com/problems/flatten-a-multilevel-doubly-linked-list/)

You are given a doubly linked list which in addition to the next and previous pointers, it could have a child pointer, which may or may not point to a separate doubly linked list. These child lists may have one or more children of their own, and so on, to produce a multilevel data structure, as shown in the example below.

Flatten the list so that all the nodes appear in a single-level, doubly linked list. You are given the head of the first level of the list.

**Example:**

```
Input:
 1---2---3---4---5---6--NULL
         |
         7---8---9---10--NULL
             |
             11--12--NULL

Output:
1-2-3-7-8-11-12-9-10-4-5-6-NULL
```

**Solution**

(1) straightforward iteration 

Basic idea is straight forward:

1. Start form the `head` , move one step each time to the next node
2. When meet with a node with child, say node `p`, follow its `child chain` to the end and connect the tail node with `p.next`, by doing this we merged the `child chain` back to the `main thread`
3. Return to `p` and proceed until find next node with child.
4. Repeat until reach `null`

```java
/*
// Definition for a Node.
class Node {
    public int val;
    public Node prev;
    public Node next;
    public Node child;

    public Node() {}

    public Node(int _val,Node _prev,Node _next,Node _child) {
        val = _val;
        prev = _prev;
        next = _next;
        child = _child;
    }
};
*/
class Solution {
    public Node flatten(Node head) {
        if (head == null) {
            return head;
        }
        Node cur = head;
        while (cur != null) {
            // When there is no child, just go ahead.
            if (cur.child == null) {
                cur = cur.next;
                continue;
            }
            // When we find a child, find the last node of child list
            //  and make it become the previous node of current node in main list
            Node tmp = cur.child;
            while (tmp.next != null) {
                tmp = tmp.next;
            }
            tmp.next = cur.next;
            if (cur.next != null) {
                cur.next.prev = tmp;
            }
            // Then the next of current node is its child.
            cur.next = cur.child;
            cur.child.prev = cur;
            cur.child = null;
        }
        return head;
    }
}
```

Time complexity: $$O(n)$$, since it just visits every node.

(2) recursion 

```java
    public Node flatten(Node head) {
    	flattentail(head);
    	return head;
    }

    // flattentail: flatten the node "head" and return the tail in its child (if exists)
    // the tail means the last node after flattening "head"

    // Five situations:
    // 1. null - no need to flatten, just return it
    // 2. no child, no next - no need to flatten, it is the last element, just return it
    // 3. no child, next - no need to flatten, go next
    // 4. child, no next - flatten the child and done
    // 5. child, next - flatten the child, connect it with the next, go next

    private Node flattentail(Node head) {
    	if (head == null) return head; // CASE 1
    	if (head.child == null) {
    		if (head.next == null) return head; // CASE 2
    		return flattentail(head.next); // CASE 3
    	}
    	else {
    		Node child = head.child;  
    		head.child = null;
    		Node next = head.next;
    		Node childtail = flattentail(child);
    		head.next = child;
    		child.prev = head;  
			if (next != null) { // CASE 5
				childtail.next = next;
				next.prev = childtail;
				return flattentail(next);
			}
            return childtail; // CASE 4
    	}	   	
    }
```

Time complexity: $$O(n)$$, since it just visits every node.

#### 6. [Linked List Cycle II](https://leetcode.com/problems/linked-list-cycle-ii/)

Given a linked list, return the node where the cycle begins. If there is no cycle, return `null`.

**Note**: Do not modify the linked list.

**Follow up**:
Can you solve it without using extra space?

**Solution**

[Floyd's cycle detection algorithm](https://www.youtube.com/watch?time_continue=2&v=zbozWoMgKW0)

(1) detect a loop

The algorithm is to start two pointers, slow and fast from head of linked list. We move slow one node at a time and fast two nodes at a time. If there is a loop, then they will definitely meet. This approach works because of the following facts.

1) When slow pointer enters the loop, the fast pointer must be inside the loop. Let fast pointer be distance `k` from slow.

2) Now if consider movements of slow and fast pointers, we can notice that distance between them (from slow to fast) increase by one after every iteration. After one iteration (of `slow = next` and `fast = next of next`), distance between slow and fast becomes `k+1`, after two iterations, `k+2`, and so on. When distance becomes `n`, they meet because they are moving in a cycle of length `n`.

For example, we can see in below diagram, initial distance is 2. After one iteration, distance becomes 3, after 2 iterations, it becomes 4. After 3 iterations, it becomes 5 which is distance 0. And they meet.

![img](https://cdncontribute.geeksforgeeks.org/wp-content/uploads/Floyd-Proof.jpg)

(2) find the start of loop

Let slow and fast meet at some point after Floyd’s Cycle finding algorithm. Below diagram shows the situation when cycle is found.

[![LinkedListCycle](http://www.geeksforgeeks.org/wp-content/uploads/LinkedListCycle.jpg)](http://www.geeksforgeeks.org/wp-content/uploads/LinkedListCycle.jpg)

We can conclude below from above diagram

```
Distance traveled by fast pointer = 2 * (Distance traveled 
                                         by slow pointer)

(m + n*x + k) = 2*(m + n*y + k)

Note that before meeting the point shown above, fast
was moving at twice speed.

x -->  Number of complete cyclic rounds made by 
       fast pointer before they meet first time

y -->  Number of complete cyclic rounds made by 
       slow pointer before they meet first time
```

From above equation, we can conclude below

```
    m + k = (x-2y)*n

Which means m+k is a multiple of n. 
```

So if we start moving both pointers again at the **same speed** such that one pointer (say slow) begins from head node of linked list and other pointer (say fast) begins from meeting point. When slow pointer reaches beginning of loop (has made m steps), fast pointer would have made also moved m steps as they are now moving same pace. Since `m+k` is a multiple of `n` and fast starts from `k`, they would meet at the beginning.

```go
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func detectCycle(head *ListNode) *ListNode {
        if p := detect(head); p != nil {
            return start(p, head)
        } else {
            return nil
        }
}

func detect(head *ListNode) *ListNode {
    slow, fast := head, head
    for slow != nil && fast != nil && fast.Next != nil {
        slow = slow.Next
        fast = fast.Next.Next
        if slow == fast {
            return slow
        }
    }
    return nil
}

func start(p, head *ListNode) *ListNode {
    q := head
    for p != q {
        p = p.Next
        q = q.Next
    }
    return q
}
```

Time complexity: $$O(n)$$