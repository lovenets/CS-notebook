#### 1.[Task Scheduler](https://leetcode.com/problems/task-scheduler)

Given a char array representing tasks CPU need to do. It contains capital letters A to Z where different letters represent different tasks.Tasks could be done without original order. Each task could be done in one interval. For each interval, CPU could finish one task or just be idle.

However, there is a non-negative cooling interval **n** that means between two **same tasks**, there must be at least n intervals that CPU are doing different tasks or just be idle.

You need to return the **least** number of intervals the CPU will take to finish all the given tasks.

**Example:**

```
Input: tasks = ["A","A","A","B","B","B"], n = 2
Output: 8
Explanation: A -> B -> idle -> A -> B -> idle -> A -> B.
```

**Note:**

1. The number of tasks is in the range [1, 10000].
2. The integer n is in the range [0, 100].

**Solution**

(1) 

> This solution is too smart to understand it...

First consider the most frequent characters, we can determine their relative positions first and use them as a frame to insert the remaining less frequent characters. Here is a proof by construction:

Let F be the set of most frequent chars with frequency k.
We can create k chunks, each chunk is identical and is a string consists of chars in F in a specific fixed order.
Let the heads of these chunks to be H_i; then H_2 should be at least n chars away from H_1, and so on so forth; then we insert the less frequent chars into the gaps between these chunks sequentially one by one ordered by frequency in a decreasing order and try to fill the k-1 gaps as full or evenly as possible each time you insert a character. **In summary, append the less frequent characters to the end of each chunk of the first k-1 chunks sequentially and round and round, then join the chunks and keep their heads' relative distance from each other to be at least n**.

Examples:

`AAAABBBEEFFGG 3`

here X represents a space gap:

```
Frame: "AXXXAXXXAXXXA"
insert 'B': "ABXXABXXABXXA" <--- 'B' has higher frequency than the other characters, insert it first.
insert 'E': "ABEXABEXABXXA"
insert 'F': "ABEFABEXABFXA" <--- each time try to fill the k-1 gaps as full or evenly as possible.
insert 'G': "ABEFABEGABFGA"
```

`AACCCBEEE 2`

```
3 identical chunks "CE", "CE CE CE" <-- this is a frame
insert 'A' among the gaps of chunks since it has higher frequency than 'B' ---> "CEACEACE"
insert 'B' ---> "CEABCEACE" <----- result is tasks.length;
```

`AACCCBBEEE 3`

```
3 identical chunks "CE", "CE CE CE" <--- this is a frame.
Begin to insert 'A'->"CEA CEA CE"
Begin to insert 'B'->"CEABCEABCE" <---- result is tasks.length;
```

`ACCCEEE 2`

```
3 identical chunks "CE", "CE CE CE" <-- this is a frame
Begin to insert 'A' --> "CEACE CE" <-- result is (c[25] - 1) * (n + 1) + 25 -i = 2 * 3 + 2 = 8
```

```java
// (c[25] - 1) * (n + 1) + 25 - i  is frame size
// when inserting chars, the frame might be "burst", then tasks.length takes precedence
// when 25 - i > n, the frame is already full at construction, the following is still valid.
public class Solution {
    public int leastInterval(char[] tasks, int n) {

        int[] c = new int[26];
        for(char t : tasks){
            c[t - 'A']++;
        }
        Arrays.sort(c);
        int i = 25;
        while(i >= 0 && c[i] == c[25]) i--;

        return Math.max(tasks.length, (c[25] - 1) * (n + 1) + 25 - i);
    }
}
```

Time Complexity: $$O(n)$$, n is the number of tasks.

(2)

The idea is:

0. To work on the same task again, CPU has to wait for time `n`, therefore we can think of as if there is a `cycle`, of `time n+1`, regardless whether you schedule some other task in the cycle or not.

1. To avoid leave the CPU with limited choice of tasks and having to sit there cooling down frequently at the end, it is critical the keep the diversity of the task pool for as long as possible.
2. In order to do that, we should try to schedule the CPU to `always try round robin between the most popular tasks at any time`.

```java
class Solution {
    public int leastInterval(char[] tasks, int n) {
        Map<Character, Integer> count = new HashMap<>();
        // calculate the frequencies of tasks
        for (char t : tasks) {
            count.put(t, count.getOrDefault(t, 0) + 1);
        }

        // descending by frequencies,
        // because we should schedule tasks
        // based on the most frequent task
        PriorityQueue<Integer> pq = new PriorityQueue<>((a, b) -> b - a);
        pq.addAll(count.values());

        int total = 0;
        int cycle = n + 1;
        while (!pq.isEmpty()) {
            int workTime = 0;
            List<Integer> frequencies = new ArrayList<>();
            // a cycle lasts (n + 1) interval
            // because there is at least n interval
            // between two same task
            for (int i = 0; i < cycle; i++) {
                if (!pq.isEmpty()) {
                    frequencies.add(pq.poll());
                    workTime++;
                }
            }
            // if c > 0, it means a specific kind of
            // task hasn't been accomplished yet
            for (int f : frequencies) {
                f--;
                if (f > 0) {
                    pq.offer(f);
                }
            }
            // if the queue is empty, that means we can finish all tasks
            // during a single cycle
            total += !pq.isEmpty() ? cycle : workTime;
        }

        return total;
    }
}
```

#### 2.[Design Circular Queue](https://leetcode.com/problems/design-circular-queue)

Design your implementation of the circular queue. The circular queue is a linear data structure in which the operations are performed based on FIFO (First In First Out) principle and the last position is connected back to the first position to make a circle. It is also called "Ring Buffer".

One of the benefits of the circular queue is that we can make use of the spaces in front of the queue. In a normal queue, once the queue becomes full, we cannot insert the next element even if there is a space in front of the queue. But using the circular queue, we can use the space to store new values.

Your implementation should support following operations:

- `MyCircularQueue(k)`: Constructor, set the size of the queue to be k.
- `Front`: Get the front item from the queue. If the queue is empty, return -1.
- `Rear`: Get the last item from the queue. If the queue is empty, return -1.
- `enQueue(value)`: Insert an element into the circular queue. Return true if the operation is successful.
- `deQueue()`: Delete an element from the circular queue. Return true if the operation is successful.
- `isEmpty()`: Checks whether the circular queue is empty or not.
- `isFull()`: Checks whether the circular queue is full or not.

**Example:**

```
MyCircularQueue circularQueue = new MyCircularQueue(3); // set the size to be 3
circularQueue.enQueue(1);  // return true
circularQueue.enQueue(2);  // return true
circularQueue.enQueue(3);  // return true
circularQueue.enQueue(4);  // return false, the queue is full
circularQueue.Rear();  // return 3
circularQueue.isFull();  // return true
circularQueue.deQueue();  // return true
circularQueue.enQueue(4);  // return true
circularQueue.Rear();  // return 4
```

**Note:**

- All values will be in the range of [0, 1000].
- The number of operations will be in the range of [1, 1000].
- Please do not use the built-in Queue library.

**My Solution**

```go
type MyCircularQueue struct {
	elements []int
	front    int
	rear     int
	count    int
}

/** Initialize your data structure here. Set the size of the queue to be k. */
func Constructor(k int) MyCircularQueue {
	arr := make([]int, k, k)
	for i, _ := range arr {
		arr[i] = -1
	}
	return MyCircularQueue{elements: arr}
}

/** Insert an element into the circular queue. Return true if the operation is successful. */
func (this *MyCircularQueue) EnQueue(value int) bool {
	if this.IsFull() {
		return false
	}
	this.elements[this.rear] = value
	this.rear = (this.rear + 1) % cap(this.elements)
	this.count++
	return true
}

/** Delete an element from the circular queue. Return true if the operation is successful. */
func (this *MyCircularQueue) DeQueue() bool {
	if this.IsEmpty() {
		return false
	}
	this.elements[this.front] = -1
	this.front = (this.front + 1) % cap(this.elements)
	this.count--
	return true
}

/** Get the front item from the queue. */
func (this *MyCircularQueue) Front() int {
	return this.elements[this.front]
}

/** Get the last item from the queue. */
func (this *MyCircularQueue) Rear() int {
	pos := this.rear - 1
	if pos < 0 {
		return this.elements[len(this.elements)-1]
	} else {
		return this.elements[pos]
	}
}

/** Checks whether the circular queue is empty or not. */
func (this *MyCircularQueue) IsEmpty() bool {
	return this.count == 0
}

/** Checks whether the circular queue is full or not. */
func (this *MyCircularQueue) IsFull() bool {
	return this.count == cap(this.elements)
}
```

Time complexity:

- `Constructor`: $$O(n)$$. n is the length of queue.

- Others: $$O(1)$$

#### 3.[Design Circular Deque](https://leetcode.com/problems/design-circular-deque/)

Design your implementation of the circular double-ended queue (deque).

Your implementation should support following operations:

- `MyCircularDeque(k)`: Constructor, set the size of the deque to be k.
- `insertFront()`: Adds an item at the front of Deque. Return true if the operation is successful.
- `insertLast()`: Adds an item at the rear of Deque. Return true if the operation is successful.
- `deleteFront()`: Deletes an item from the front of Deque. Return true if the operation is successful.
- `deleteLast()`: Deletes an item from the rear of Deque. Return true if the operation is successful.
- `getFront()`: Gets the front item from the Deque. If the deque is empty, return -1.
- `getRear()`: Gets the last item from Deque. If the deque is empty, return -1.
- `isEmpty()`: Checks whether Deque is empty or not. 
- `isFull()`: Checks whether Deque is full or not.

**Example:**

```
MyCircularDeque circularDeque = new MycircularDeque(3); // set the size to be 3
circularDeque.insertLast(1);			// return true
circularDeque.insertLast(2);			// return true
circularDeque.insertFront(3);			// return true
circularDeque.insertFront(4);			// return false, the queue is full
circularDeque.getRear();  			// return 2
circularDeque.isFull();				// return true
circularDeque.deleteLast();			// return true
circularDeque.insertFront(4);			// return true
circularDeque.getFront();			// return 4
```

**Note:**

- All values will be in the range of [0, 1000].
- The number of operations will be in the range of [1, 1000].
- Please do not use the built-in Deque library.

**Solution**

(1) use circular array

```go
type MyCircularDeque struct {
	arr   []int
	front int
	rear  int
}

/** Initialize your data structure here. Set the size of the deque to be k. */
func Constructor(k int) MyCircularDeque {
	return MyCircularDeque{make([]int, k, k), -1, 0}
}

/** Adds an item at the front of Deque. Return true if the operation is successful. */
func (this *MyCircularDeque) InsertFront(value int) bool {
	if this.IsFull() {
		return false
	}

	if this.front == -1 {
		// the queue is empty
		this.front, this.rear = 0, 0
	} else if this.front == 0 {
		// the front pointer is in the first position of array
		this.front = len(this.arr) - 1
	} else {
		this.front--
	}
	this.arr[this.front] = value
	return true
}

/** Adds an item at the rear of Deque. Return true if the operation is successful. */
func (this *MyCircularDeque) InsertLast(value int) bool {
	if this.IsFull() {
		return false
	}

	if this.front == -1 {
		this.front, this.rear = 0, 0
	} else if this.rear == len(this.arr)-1 {
		// the rear pointer is in the last position of array
		this.rear = 0
	} else {
		this.rear++
	}
	this.arr[this.rear] = value
	return true
}

/** Deletes an item from the front of Deque. Return true if the operation is successful. */
func (this *MyCircularDeque) DeleteFront() bool {
	if this.IsEmpty() {
		return false
	}

	if this.front == this.rear {
		// there is only on element in the queue
		this.front, this.rear = -1, -1
	} else {
		if this.front == len(this.arr)-1 {
			this.front = 0
		} else {
			this.front++
		}
	}
	return true
}

/** Deletes an item from the rear of Deque. Return true if the operation is successful. */
func (this *MyCircularDeque) DeleteLast() bool {
	if this.IsEmpty() {
		return false
	}

	if this.front == this.rear {
		this.front, this.rear = -1, -1
	} else if this.rear == 0 {
		this.rear = len(this.arr) - 1
	} else {
		this.rear--
	}
	return true
}

/** Get the front item from the deque. */
func (this *MyCircularDeque) GetFront() int {
	if this.IsEmpty() {
		return -1
	} else {
		return this.arr[this.front]
	}
}

/** Get the last item from the deque. */
func (this *MyCircularDeque) GetRear() int {
	if this.IsEmpty() {
		return -1
	} else {
		return this.arr[this.rear]
	}
}

/** Checks whether the circular deque is empty or not. */
func (this *MyCircularDeque) IsEmpty() bool {
	return this.front == -1
}

/** Checks whether the circular deque is full or not. */
func (this *MyCircularDeque) IsFull() bool {
	return (this.front == 0 && this.rear == len(this.arr)-1) || this.front == this.rear+1
}
```

Time complexity: all operations are $$O(1)$$.

(2) use linked list

```go
type ListNode struct {
	val  int
	next *ListNode
}

// linked-list=based
type MyCircularDeque struct {
	front *ListNode
	rear  *ListNode
	size  int
	cap   int
}

/** Initialize your data structure here. Set the size of the deque to be k. */
func Constructor(k int) MyCircularDeque {
	front := &ListNode{-1, nil}
	rear := &ListNode{-1, nil}
	front.next = rear
	return MyCircularDeque{front, rear, 0, k}
}

/** Adds an item at the front of Deque. Return true if the operation is successful. */
func (this *MyCircularDeque) InsertFront(value int) bool {
	if this.IsFull() {
		return false
	}

	if this.IsEmpty() {
		this.front.val = value
		this.rear.val = value
	} else {
		newNode := &ListNode{value, this.front}
		this.front = newNode
	}
	this.size++
	return true
}

/** Adds an item at the rear of Deque. Return true if the operation is successful. */
func (this *MyCircularDeque) InsertLast(value int) bool {
	if this.IsFull() {
		return false
	}

	if this.IsEmpty() {
		this.rear.val = value
		this.front.val = value
	} else {
		newNode := &ListNode{value, nil}
		this.rear.next = newNode
		this.rear = newNode
	}
	this.size++
	return true
}

/** Deletes an item from the front of Deque. Return true if the operation is successful. */
func (this *MyCircularDeque) DeleteFront() bool {
	if this.IsEmpty() {
		return false
	}

	this.front = this.front.next
	this.size--
	return true
}

/** Deletes an item from the rear of Deque. Return true if the operation is successful. */
func (this *MyCircularDeque) DeleteLast() bool {
	if this.IsEmpty() {
		return false
	}

	pre := this.front
	for pre.next != nil && pre.next.next != nil {
		pre = pre.next
	}
	this.rear = pre
	this.size--
	return true
}

/** Get the front item from the deque. */
func (this *MyCircularDeque) GetFront() int {
	if this.IsEmpty() {
		return -1
	} else {
		return this.front.val
	}
}

/** Get the last item from the deque. */
func (this *MyCircularDeque) GetRear() int {
	if this.IsEmpty() {
		return -1
	} else {
		return this.rear.val
	}
}

/** Checks whether the circular deque is empty or not. */
func (this *MyCircularDeque) IsEmpty() bool {
	return this.size == 0
}

/** Checks whether the circular deque is full or not. */
func (this *MyCircularDeque) IsFull() bool {
	return this.size == this.cap
}
```

Time complexity: `DeleteRear` is $$O(n)$$ and other operations are $$O(1)$$.

#### 4. [Max Sum of Rectangle No Larger Than K](https://leetcode.com/problems/max-sum-of-rectangle-no-larger-than-k/)

Given a non-empty 2D matrix *matrix* and an integer *k*, find the max sum of a rectangle in the *matrix* such that its sum is no larger than *k*.

**Example:**

```
Input: matrix = [[1,0,1],[0,-2,3]], k = 2
Output: 2 
Explanation: Because the sum of rectangle [[0, 1], [-2, 3]] is 2,
             and 2 is the max number no larger than k (k = 2).
```

**Note:**

1. The rectangle inside the matrix must have an area > 0.
2. What if the number of rows is much larger than the number of columns?

**Solution**

Reference: 

1. maximum subarray-[Kadane's algorithm](https://en.wikipedia.org/wiki/Maximum_subarray_problem)
2. [2D Kadane](https://www.slideshare.net/TusharBindal/2-d-kadane)

```java
public int maxSumSubmatrix(int[][] matrix, int k) {
        int row=matrix.length, col=matrix[0].length, ans=Integer.MIN_VALUE;
        for(int left=0;left<col;left++){
            int[] sum=new int[row];
            for(int right=left;right<col;right++){
                for(int r=0;r<row;r++){
                    sum[r]+=matrix[r][right];
                }
                TreeSet<Integer> curSums=new TreeSet<Integer>();
                curSums.add(0);
                int curMax=Integer.MIN_VALUE, cum=0;
                for(int s:sum){
                    cum+=s;
                    // val >= cum-k ie. k >= cum-val
                    Integer val=curSums.ceiling(cum-k);
                    if(val!=null) curMax=Math.max(curMax,cum-val);
                    curSums.add(cum);
                }
                ans=Math.max(ans,curMax);
            }
        }
        return ans;
    }
```

Time complexity: $$O(min(rows,cols)^2*max(rows,cols)*log(max(rows,cols)))$$.

#### 5. [BInary Tree Zigzag Level Order Traversal](https://leetcode.com/problems/binary-tree-zigzag-level-order-traversal)

Given a binary tree, return the *zigzag level order* traversal of its nodes' values. (ie, from left to right, then right to left for the next level and alternate between).

For example:
Given binary tree `[3,9,20,null,null,15,7]`,

```
    3
   / \
  9  20
    /  \
   15   7
```

return its zigzag level order traversal as:

```
[
  [3],
  [20,9],
  [15,7]
]

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
func zigzagLevelOrder(root *TreeNode) [][]int {
    res := make([][]int, 0)
    traverse(root, &res, 0)
    return res
}

func traverse(cur *TreeNode, res *[][]int, level int) {
    if cur == nil {
        return 
    }
    
    if len(*res) <= level {
        newLevel := make([]int, 0)
        *res = append(*res, newLevel)
    }
    
    nodes := (*res)[level]
    if level % 2 == 0 {
        // LTR
        nodes = append(nodes, cur.Val)
    } else {
        // RTL
        nodes = append([]int{cur.Val}, nodes...)
    }
    (*res)[level] = nodes
    
    traverse(cur.Left, res, level+1)
    traverse(cur.Right, res, level+1)
}
```

Time complexity: $$O(n)$$, n is the number of nodes.

(2) queue

Assuming after traversing the 1st level, nodes in queue are {9, 20, 8}, And we are going to traverse 2nd level, which is even line and should print value from right to left.

Let's say when we finish 2nd level, we know there are 3 nodes in current queue, and the nodes are also {9, 20, 8} so the list for this level in final result should be of size 3. For example, for node(9), it's index in queue is 0, so its index in the list for this level in final result should be (3-1-0) = 2.

```go
func zigzagLevelOrder(root *TreeNode) [][]int {
    if root == nil {
        return [][]int{}
    }
    
    res := make([][]int, 0)
    queue := make([]*TreeNode, 0)
    queue = append(queue, root)
    LTR := true
    for len(queue) > 0 {
        size := len(queue)
        // insert nodes' values of current level into result 
        row := make([]int, size)
        for i := 0; i < size; i++ {
            node := queue[0]
            queue = queue[1:]
            // find the position of current node
            var index int
            if LTR {
                index = i
            } else {
                index = size - 1 - i
            }
            row[index] = node.Val
            
            if node.Left != nil {
                queue = append(queue, node.Left)
            } 
            if node.Right != nil {
                queue = append(queue, node.Right)
            }
        }       
        // after this level
        LTR  = !LTR
        res = append(res, row)
    }
    return res
}
```

Time complexity: $$O(n)$$

#### 6. [Shortest Subarray with Sum at Least K](https://leetcode.com/problems/shortest-subarray-with-sum-at-least-k/)

Return the **length** of the shortest, non-empty, contiguous subarray of `A` with sum at least `K`.

If there is no non-empty subarray with sum at least `K`, return `-1`.

**Example 1:**

```
Input: A = [1], K = 1
Output: 1
```

**Example 2:**

```
Input: A = [1,2], K = 4
Output: -1
```

**Example 3:**

```
Input: A = [2,-1,2], K = 3
Output: 3
```

**Note:**

1. `1 <= A.length <= 50000`
2. `-10 ^ 5 <= A[i] <= 10 ^ 5`
3. `1 <= K <= 10 ^ 9`

**Solution**

_Recall of the Sliding window solution in a positive array_

The `Sliding window` solution finds the subarray we are looking for in a `linear` time complexity. The idea behind it is to maintain two pointers: **start** and **end**, moving them in a `smart way` to avoid examining all possible values `0<=end<=n-1` and `0<=start<=end` (to avoid brute force).
What it does is:

1. Incremeting the **end** pointer while the sum of current subarray (defined by current values of `start` and `end`) is smaller than the target.
2. `Once we satisfy` our condition (the sum of current subarray >= target) we keep `incrementing` the **start** pointer until we `violate` it (until `sum(array[start:end+1]) < target`).
3. Once we violate the condition we keep incrementing the **end** pointer until the condition is satisfied again and so on.

The reason why we stop incrementing `start` when we violate the condition is that we are sure we will not satisfy it again if we keep incrementing `start`. In other words, if the sum of the current subarray `start -> end` is smaller than the target then the sum of `start+1 -> end` is neccessarily smaller than the target. (positive values)
The problem with this solution is that it doesn't work if we have negative values, this is because of the sentence above `Once we "violate" the condition we stop incrementing start`.

_Problem of the Sliding window with negative values_

Now, let's take an example with negative values `nums = [3, -2, 5]` and `target=4`. Initially `start=0`, we keep moving the **end** pointer until we satisfy the condition, here we will have `start=0` and `end=2`. Now we are going to move the start pointer `start=1`. The sum of the current subarray is `-2+5=3 < 4` so we violate the condition. However if we just move the **start** pointer another time `start=2` we will find `5 >= 4` and we are satisfying the condition. And this is not what the Sliding window assumes.

_Deque solution_

The Deque solution is just a `modification` of the Sliding window solution above. We will modify the way we are updating `start`.

```go
func shortestSubarray(A []int, K int) int {
    N, res := len(A), len(A) + 1
    B := make([]int, N + 1, N + 1)
    for i := 0; i < N; i++ {
        B[i+1] = B[i] + A[i]
    }
    deque := make([]int, 0)
    for i := 0; i < N + 1; i++ {
        for len(deque) > 0 && B[i] - B[deque[0]] >= K {
            if res > i - deque[0] {
                res = i - deque[0]
            }
            deque = deque[1:]
        }
        for len(deque) > 0 && B[i] <= B[deque[len(deque)-1]] {
            deque = deque[:len(deque)-1]
        }
        deque = append(deque, i)
    }
    if res <= N {
        return res
    } else {
        return -1
    }
}
```

**What does the Deque store :**
The deque stores the `possible` values of the **start** pointer. Unlike the sliding window, values of the `start` variable will not necessarily be contiguous.

**Why is it increasing :**
So that when we move the **start** pointer and we violate the condition, we are sure we will violate it if we keep taking the other values from the Deque. In other words, if the sum of the subarray from `start=first value in the deque` to `end` is smaller than `target`, then the sum of the subarray from `start=second value in the deque` to `end` is necessarily smaller than `target`.
So because the Deque is increasing (`B[d[0]] <= B[d[1]]`), we have `B[i] - B[d[0]] >= B[i] - B[d[1]]`, which means the sum of the subarray starting from `d[0]` is greater than the sum of the sub array starting from `d[1]`.

**Why are we having a prefix array and not just the initial array like in the sliding window :**
Because in the sliding window when we move `start` (typically when we increment it) we can just substract `nums[start-1]` from the current sum and we get the sum of the new subarray. Here the value of the `start` is `jumping` and the only way to compute the sum of the current subarray in a `constant` time is to have the prefix array.

**Why using Deque and not simply an array :**
We can use an array, however we will find ourselves doing only three operations:
1- `remove_front` : when we satisfy our condition and we want to move the start pointer
2- `append_back` : for any index that may be a future *start pointer*
3- `remove_back` : When we are no longer satisfying the increasing order of the array
Deque enables doing these 3 operations in a constant time.

Time complexity: $$O(n)$$, n is the length of input array.