## 数组中重复的数字

### 1. 找出数组中重复的数字

在一个长度为 n 的数组里的所有数字都在 0~n-1 的范围内，数组中有些数字是重复的，但不知道有几个数字重复了，也不知道每个数字重复了几次。请找出数组中任意一个重复的数字。例如，如果输入长度为 7 的数组 `{2, 3, 1, 0, 2, 5, 3}`，那么对应的输出是重复的数字 2 或者 3。

#### 分析

- 可以先对数组进行排序，然后顺序扫描数组，判断相邻的数字是否相同。快速排序的时间复杂度是$$O(nlogn)$$，所以这个解法的时间复杂度是$$O(nlogn)$$；空间复杂度是$$O(logn)$$

- 可以利用哈希表，顺序扫描的过程中判断数字是否已经出现过。因为已经明确所有数字都在 0~n-1 范围内，所以可以直接用一个长度为 n 的数组作为哈希表，键就是下标，所以查找时间是$$O(1)$$，整个算法的时间复杂度是$$O(n)$$，空间复杂度是$$O(n)$$

可以根据排序的思想，用一个空间复杂度为$$O(1)$$的方法解决问题。如果数组有序，那么第一个位置上的数字是 0，第二个位置上的数字是 1，即`a[i] == i`。所以，顺序扫描数组，对于数字`a[i]`，判断`a[i] == i`，如果成立，说明这个数字位于正确的位置上，接着扫描下一个数字；如果不成立，判断`a[i] == a[a[i]]`，如果成立，就说明有重复，否则交换`a[i]`和`a[a[i]]`，直到`a[i]`位于正确的位置上。

```go
// 存在则返回一个重复的数字，其他情况返回 -1
func findDuplication1(numbers []int) int {
	// 数组为空
	if len(numbers) == 0 {
		return -1
	}
	// 数组中的数字不在 0~n-1 的范围内
	for _, n := range numbers {
		if n < 0 || n > len(numbers)-1 {
			return -1
		}
	}

	for i := range numbers {
		// 当前数字不在正确的位置上
		if numbers[i] != i {
			if numbers[i] == numbers[numbers[i]] {
				// 找到重复的数字
				return numbers[i]
			} else {
				// 将当前数字放到正确的位置上
				for numbers[i] != i {
					numbers[i], numbers[numbers[i]] = numbers[numbers[i]], numbers[i]
				}
			}
		}
	}
	return -1
}
```

每个数字最多交换两次就能位于正确的位置上，所以时间复杂度是$$O(n)$$，空间复杂度是$$O(1)$$。

### 2. 不修改数组中找出重复的数字

在一个长度为 n+1 的数组中，所有数字都在 1~n 的范围内，所以数组中至少有一个数字是重复的，请找出任意一个重复的数字，要求不能修改原数组。

#### 分析

如果不重复，那么 1~n 的范围内正好是每个数字一个。把 1~n 的数字从中间的数字 m 分为两部分，前面一半为 1~m，后面一半为 m+1~n。如果前面一半的数字个数超过 m，意味着这一个区间里面一定有重复的数；否则，就是后面一半这个区间里面有重复的数字。按照这种类似二分查找的方法，就可以找到重复的数字。

```go
func findDuplicatioin2(numbers []int) int {
	if len(numbers) == 0 {
		return -1
	}
	for _, n := range numbers {
		if n < 1 || n > len(numbers) {
			return -1
		}
	}
	start, end := 0, len(numbers)-1
	for start <= end {
		mid := (start + end) / 2
		count := countRange(numbers, start, mid)
		// 区间长度为1，也就是只需判断这个数在整个数组中出现了多少次
		// 就能直到它是否重复
		if end == start {
			if count > 1 {
				return start
			} else {
				break
			}
		}
		// 确定下一次查找的区间
		if count > mid-start+1 {
			end = mid
		} else {
			start = mid + 1
		}
	}
	return -1
}

func countRange(numbers []int, start, end int) int {
	if len(numbers) == 0 {
		return 0
	}
	count := 0
	for i := range numbers {
		if numbers[i] >= start && numbers[i] <= end {
			count++
		}
	}
	return count
}
```

`countRange`被调用$$logn$$次，它的时间复杂度是$$O(n)$$，因此整个算法时间复杂度是$$O(nlogn)$$，空间复杂度是$$O(1)$$。

## 二维数组中的查找

在一个二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序，请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。

### 分析

以下面的矩阵为例：

```
1 2 8  9
2 4 9  12
4 7 10 13
6 8 11 15
```

查找 7 的的过程为：
（1）首先看矩阵右上角的数字 9，9 大于 7，说明 9 所在的列不可能包含 7，所以把这一列去掉；

（2）在剩下的矩阵中同样是先看右上角的数字，也就是 8，8 也大于 7，这一列舍去；

（3）在剩下的矩阵中取右上角数字 2，2 小于 7，说明 7 不可能在这一行，所以把这一行去掉。

以此类推，当剩下的矩阵为：

```
4 7
6 8
```

这时右上角的数字就是 7。

也就说，每次都取当前矩阵右上角数字，如果不是所要找的数，那么就根据大小关系去掉一行或是一列；重复这个过程，直到找到目标或是整个矩阵都被去掉。

```go
func findInMatrix(matrix [][]int, target int) bool {
   if len(matrix) == 0 || len(matrix[0]) == 0 {
      return false
   }
   i, j := 0, len(matrix[0])-1
   for i < len(matrix) && j >= 0 {
      if matrix[i][j] == target {
         return true
      } else if matrix[i][j] < target {
         // 去掉一行
         i++
      } else {
         // 去掉一列    
         j--
      }
   }
   return false
}
```

最好的情况下只用找比较 1 次，最坏的情况下需要比较的次数为副对角线上的数字个数，所以时间复杂度为$$O(n)$$，空间复杂度是$$O(1)$$。

## 替换空格

请实现一个函数，把字符串中的每个空格替换成“%20”.

### 分析

可以直接使用库函数`strings.Replace`来实现，这里参考`strings.Replace`的源码给出一种实现。

```go
func Replace(s string) string {
	// Compute number of replacements.
	m := countBlank(s)
	if  m == 0 {
		return s // avoid allocation
	}
    
	// Calculate the length of string to be generated
    // and make a buffer.
	t := make([]byte, len(s)+m*(len("%20")-len(" ")))
	w := 0
	start := 0
	for i := 0; i < m; i++ {
        j := start
        // let j point to the first blank in the left string
        j += indexOfBlank(s[start:])
        // s[start:j] is the substring before a blank
        w += copy(t[w:], s[start:j])
        // let new string replace a blank
        w += copy(t[w:], "%20")
        // let start point to the place just after the blank replaced
        start = j + len(" ")
	}
    // copy the left substring
	w += copy(t[w:], s[start:])
	return string(t[0:w])
}

func countBlank(s string) int {
    if s == "" {
        return 0
    }
    count := 0
    for _, r := range s {
        if r == ' ' {
            count++
        }
    }
    return count
}

func indexOfBlank(s string) int {
    for i, r := range s {
        if r == ' ' {
            return i
        }
    }
    return -1
}
```

空间复杂度和源字符串以及替换的内容有关，时间复杂度是$$O(n)$$。

### 相关题目

有两个排序的数组 A1 和 A2，内存在 A1 的末尾有足够的空间容纳 A2.请实现一个函数，把 A2 中的所有数字插入 A1 中，并且所有数字是有序的。

#### 分析

这个题目要求将两个数组归并到其中一个数组中（不同于常规的可以利用额外空间的二分归并），如果从前到后归并，那么显然会移动较多的元素，不妨从后往前归并。

```go
// n 是当前 a1 含有的元素个数
func mergeTwoArrays(a1 []int, n int, a2 []int) []int {
	k := n + len(a2) - 1 // 归并之后最后一个位置下标为 k
	i, j := n-1, len(a2)-1
	// 下面就是二路归并
	for i >= 0 && j >= 0 {
		if a1[i] > a2[j] {
			a1[k] = a1[i]
			i--
		} else {
			a1[k] = a2[j]
			j--
		}
		k--
	}
	for i >= 0 {
		a1[k] = a1[i]
		i--
		k--
	}
	for j >= 0 {
		a1[k] = a2[j]
		j--
		k--
	}
	return a1
}
```

时间复杂度是显然是$$O(两个数组长度之和)$$，空间复杂度是$$O(1)$$。

## 从尾到头打印链表

输入一个链表的头节点，从尾到头反过来打印每个节点的值。

### 分析

最容易想到的做法就是顺序遍历链表，遍历的过程中节点入栈，最后再出栈。空间复杂度和时间复杂度都是$$O(n)​$$。

```go
type ListNode struct {
	Key  int
	Next *ListNode
}

func reversePrint(head *ListNode) {
	if head == nil {
		return 
	}
	stack := make([]*ListNode, 0)
	for cur := head; cur != nil; cur = cur.Next {
		stack = append(stack, cur)
	}
	for len(stack) > 0 {
		node := stack[len(stack)-1]
		fmt.Println(node.Key)
		stack = stack[:len(stack)-1]
	}
}
```

也可以采用递归的方法，但是如果链表太长，递归的方法很可能导致调用栈溢出。

```go
func reversePrintRecur(head *ListNode) {
    if head != nil {
        if head.Next != nil {
            reversePrintRecur(head.Next)
        }
        fmt.Println(head.Key)
    }
}
```

还有一种思路，可以先把链表中每个节点的 Next 指针反转，这样就得到一个反向的链表。但是这种操作会修改原来的链表，**在面试中，如果打算修改输入的数据，最好先问面试官是不是允许修改**。

## 重建二叉树

输入某二叉树的前序遍历和中序遍历的结果，请重建该二叉树。假设输入的前序遍历和中序遍历的结果中都不含有重复的数字。二叉树的节点定义为：

```pseudocode
struct BinaryTreeNode {
    int              value
    BinaryTreeNode*  left
    BinaryTreeNode*  right
}
```

### 分析

题目不难，关键是要考虑到程序的健壮性，要对无效的输入做出判断。

```go
type BinaryTreeNode struct {
   Value int
   Left  *BinaryTreeNode
   Right *BinaryTreeNode
}

func rebuildBT(preorder, inorder []int) (*BinaryTreeNode, error) {
   if len(preorder) <= 0 || len(inorder) <= 0 {
      return nil, nil
   }
   if len(preorder) == 1 && len(inorder) == 1 {
        // 同一个节点的值不一样
      if preorder[0] != inorder[0] {
         return nil, errors.New("invalid input")
      }
   }
    // 在中序序列中找根节点
   root := preorder[0]
   indexOfRoot := -1
   for i, v := range inorder {
      if v == root {
         indexOfRoot = i
         break
      }
   }
    // 在中序序列中找不到根节点
   if indexOfRoot == -1 {
      return nil, errors.New("invalid input")
   }
   left, err := rebuildBT(preorder[1:indexOfRoot+1], inorder[:indexOfRoot])
   if err != nil {
      return nil, err
   }
   right, err := rebuildBT(preorder[1+indexOfRoot:], inorder[indexOfRoot+1:])
   if err != nil {
      return nil, err
   }
   return &BinaryTreeNode{root, left, right}, nil
}
```

这个算法相当于每个节点都访问一次，时间复杂度为$$O(n)$$，空间复杂度为$$O(树的深度)$$。

## 二叉树的下一个节点

给定一棵二叉树和其中的一个节点，如何找出中序遍历序列的下一个节点？树中的节点除了有两个分别指向左、右子节点的指针，还有一个指向父节点的指针。

### 分析

![剑指 offer_8_1](img/剑指 offer_8_1.jpg)

以上图为例，中序遍历的结果为`{d, b, h, e, i, a, f, c, g}`，经过分析可知：

- 如果节点有右子树，那么中序遍历的下一个节点就是右子树的最左下节点
- 如果节点没有右子树，而且这个节点是其父节点的左孩子，那么中序遍历的下一个节点就是其父节点
- 如果节点没有右子树，而且这个节点是其父节点的右孩子，那么中序遍历的下一个节点应该是它的某一个祖先节点，而且这个祖先节点本身是左孩子；如果这样的祖先节点不存在，那么中序遍历的下一个节点不存在

```go
type BinaryTreeNode struct {
   Value  int
   Left   *BinaryTreeNode
   Right  *BinaryTreeNode
   Parent *BinaryTreeNode
}

func nextInorder(node *BinaryTreeNode) *BinaryTreeNode {
   if node == nil {
      return nil
   }
   if node.Right != nil {
      // 找右子树的最左下节点
      cur := node.Right
      for cur.Left != nil {
         cur = cur.Left
      }
      return cur
   }
   // 没有右子树，有父节点
   if node.Parent != nil {
      if node.Parent.Left == node {
         // 本身是左孩子
         return node.Parent
      } else {
         // 本身是右孩子
         for cur := node.Parent; cur != nil; cur = cur.Parent {
            if cur.Parent != nil && cur == cur.Parent.Left {
               return cur
            }
         }
      }
   }
   return nil
}
```

时间复杂度为$$O(树的深度)$$，空间复杂度为$$O(1)$$。

## 用两个栈实现队列

用两个栈实现一个队列，请实现它的两个函数`appendTail`和`deleteHead`，分别完成在队列尾部插入节点和在队列头部删除节点的功能。队列的声明为：

```go
type CQueue struct {
    Stack1 []interface{}
    Stack2 []interface{}
}

func NewCQueue() *CQueue {
    return &CQueue{make([]interface{}, 0), make([]interface{}, 0)}
}
```

### 分析

入队的时候，总是压入 stack1。出队的时候则分情况：

- 如果两个栈都为空，那么说明队列为空；
- 如果 stack2 为空，那么将 stack1 的元素依次出栈并压入 stack2，然后 stack2 的栈顶元素就是队头元素；
- 如果 stack2 不为空，那么栈顶元素就是队头元素。

```go
func (q *CQueue) AppendTail(e interface{}) {
   q.stack1 = append(q.stack1, e)
}

func (q *CQueue) DeleteHead() interface{} {
   if len(q.stack1) == 0 && len(q.stack2) == 0 {
      return nil
   }
   var top interface{}
   if len(q.stack2) == 0 {
      // stack2 为空，则 stack1 依次出栈
      for len(q.stack1) > 1 {
         q.stack2 = append(q.stack2, q.stack1[len(q.stack1)-1])
         q.stack1 = q.stack1[:len(q.stack1)-1]
      }
      top = q.stack1[0]
      q.stack1 = q.stack1[:0]
   } else {
      // stack2 不为空
      top = q.stack2[len(q.stack2)-1]
      q.stack2 = q.stack2[:len(q.stack2)-1]
   }
   return top
}
```

入队的时间复杂度是$$O(1)​$$，出队的时间复杂度最坏情况下是$$O(n)​$$，最好情况下是$$O(1)​$$。空间复杂度是$$O(1)​$$。

### 相关题目

用两个队列实现一个栈。

#### 分析

入栈的时候总是加入 queue1，出栈则需要分情况考虑：

- 如果两个队列都为空，说明栈为空
- 将不空的队列中的元素依次出队并加入另一个队列，直到原来不为空的队列中只剩一个元素，这个元素就是队头

```go
type CStack struct {
    queue1 []interface{}
    queue2 []interface{}
}

func NewCStack() *CStack {
    return &CStack{make([]interface{}, 0), make([]interface{}, 0)}
}

func (s *CStack) Push(e interface{}) {
    s.queue1 = append(s.queue1, e)
}

func (s *CStack) Pop() interface{} {
    if len(s.queue1) == 0 && len(s.queue2) == 0 {
        return nil
    }
    var top
    if len(s.queue1) > 0 {
        for len(s.queue1) > 1 {
            s.queue2 = append(s.queue2, s.queue1[0])
            s.queue1 = s.queue1[1:]
        }
        top = s.queue1[0]
        s.queue1 = s.queue1[1:]
    } else {
        for len(s.queue2) > 1 {
            s.queue1 = append(s.queue1, s.queue2[0])
            s.queue2 = s.queue2[1:]
        }
        top = s.queue2[0]
        s.queue2 = s.queue2[1:]
    }
    return top
}
```

时间复杂度为$$O(n)$$。

## 斐波那契数列

斐波那契数列及其相关的变形，解题的关键在于分析问题时找到递推关系（比如用数学归纳法），然后确定这是一个斐波那契数列问题。

### 1. 求斐波那契数列的第 n 项

#### 分析

递归是非常直观的解法，但是递归时间效率和空间效率都很差，因为在计算的过程有很多项的计算是重复的：

![剑指 offer_10_1](img/剑指 offer_10_1.jpg)

为了避免重复计算，就可以使用记忆化的方式：

```go
func MemomizedFibonacci(n int) int {
	// computed 用来记忆话计算结果
	computed := make(map[int]int)
	computed[0], computed[1] = 0, 1
	return helper(n, computed)
}

func helper(n int, computed map[int]int) int {
	if _, ok := computed[n]; ok {
		return computed[n]
	} else {
		res := helper(n-1, computed) + helper(n-2, computed)
		computed[n] = res
		return res
	}
}
```

实际上，用循环来计算，时间和空间的效率都是很高的。

```go
func IterativeFibonacci(n int) int {
    if n == 0 {
        return 0
    }
    if n == 1 {
        return 1
    }
    i, j := 0, 1
    res := 0
    for i := 2; i <= n; i++ {
        res = i + j
        i, j = j, res
    }
    return res
}
```

时间复杂度为$$O(n)$$，空间复杂度为$$O(1)$$。

### 2. 青蛙跳台阶问题

一只青蛙可以跳上 1 级台阶，也可以跳上 2 级台阶。求该青蛙跳上一个 n 级的台阶总共有多少种解法。

#### 分析

把跳 n 级台阶所需的跳数看成 n 的函数，记为$$f(n)$$。当$$n>2$$时，如果第一跳为 1 级，那么跳法总数等于剩下的 n - 1 级台阶的跳法数目；如果第一跳为 2 级，那么跳法总数等于剩下的 n - 2 级台阶的跳法数目。也就是说，$$f(n)=f(n-1)+f(n-2)$$。这就转换为了斐波那契数列问题。但是要注意边界条件不同。

```go
func FrogJump(n int) int {
    if n == 0 {
        return 0
    }
    if n == 1 {
        return 1
    }
    if n == 2 {
        return 2
    }
    i, j := 1, 2
    res := 0
    for i := 3; i <= n; i++ {
        res = i + j
        i, j = j, res
    }
    return res
}
```

## 旋转数组的最小数字

把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。输入一个递增排序的数组的一个旋转，输出旋转数组的最小元素。例如，数组`{3, 4, 5, 1, 2}`为`{1, 2, 3, 4, 5}`的一个旋转，该数组的最小值为 1。

### 分析

原数组是递增的，那么原数组的第一个元素就是最小元素。旋转之后形成的数组实际上是分部有序的，前一个有序的子数组的最后一个元素是最大元素，后一个有序子数组的第一个元素就是最小元素，两个元素显然是相邻的。可以从后往前遍历旋转后的数组，当发现前一个元素比当前元素大时，当前元素就是最小元素。

```go
func MinOfRotatedArray(arr []int) (int, error) {
	if len(arr) == 0 {
		return 0, errors.New("invalid input")
	}
	// 考虑了只有一个元素、所有元素均相同的特殊情况
	i := len(arr) - 1
	for ; i > 0; i-- {
		if arr[i] < arr[i-1] {
			break
		}
	}
	return arr[i], nil
}
```

时间复杂度是$$O(n)$$，空间复杂度是$$O(1)$$。

## 矩阵中的路径

请设计一个函数，用来判断在一个矩阵中是否存在一条包含某字符串所有字符的路径。路径可以从矩阵的任意一格开始，每一步可以在矩阵中向左、右、上、下移动一格。如果一条路径经过了矩阵的某一格，那么该路径不能再次进入该格子。

比如下面的矩阵中包含路径“bfcc”，但是不包含路径“abfb”。

```
a b t g
c f c s
j d c h
```

### 分析

在矩阵中查找路径，可以考虑使用 BFS。

```go
func FindPathInMatrix(matrix [][]byte, path string) bool {
	if len(matrix) == 0 {
		return false
	}
	if len(path) == 0 {
		return true
	}
	for i := 0; i < len(matrix); i++ {
		for j := 0; j < len(matrix[0]); j++ {
			// 从不同的起点开始进行 bfs
			if bfs(matrix, path, i, j) {
				return true
			}
		}
	}
	return false
}

func bfs(matrix [][]byte, path string, i int, j int) bool {
	dir := [][]int{{0, 1}, {0, -1}, {1, 0}, {-1, 0}}
	xMax, yMax := len(matrix), len(matrix[0])
	q := make([][]int, 0)
	visited := make([][]bool, xMax)
	for i := range visited {
		visited[i] = make([]bool, yMax)
	}
	
	count := 0 // 记录当前已经搜索到的字符数
	q = append(q, []int{i, j})
	visited[i][j] = true
	for len(q) != 0 {
		head := q[0]
		q = q[1:]
		if matrix[head[0]][head[1]] == path[count] {
			count++
			if count == len(path) {
				return true
			}
			for k := range dir {
				nextX, nextY := head[0]+dir[k][0], head[1]+dir[k][1]
				if nextX >= 0 && nextX < xMax && nextY >= 0 && nextY < yMax && !visited[nextX][nextY] {
					q = append(q, []int{nextX, nextY})
					visited[nextX][nextY] = true
				}
			}
		}
	}
	return false
}
```

总的时间复杂度是$$O(n^2)$$，空间复杂度是$$O(n)$$。

也可以使用回溯法，也就是用 DFS 的方法。

```go
func FindPathInMatrix(matrix [][]byte, path string) bool {
	if len(matrix) == 0 {
		return false
	}
	if len(path) == 0 {
		return true
	}
	for i := 0; i < len(matrix); i++ {
		for j := 0; j < len(matrix[0]); j++ {
			// 从不同的起点开始 dfs
			if dfs(matrix, path, i, j) {
				return true
			}
		}
	}
	return false
}

func dfs(matrix [][]byte, path string, i int, j int) bool {
   rows, cols := len(matrix), len(matrix[0])
   visited := make([][]bool, rows)
   for i := range visited {
      visited[i] = make([]bool, cols)
   }
   count := 0
   return dfsHelper(matrix, path, i, j, rows, cols, visited, &count)
}

func dfsHelper(matrix [][]byte, path string, i int, j int, rows int, cols int, visited [][]bool, count *int) bool {
   if i >= 0 && i < rows && j >= 0 && j < cols && !visited[i][j] {
      visited[i][j] = true
      if path[*count] == matrix[i][j] {
         *count++
         if *count == len(path) {
            return true
         }
         if dfsHelper(matrix, path, i+1, j, rows, cols, visited, count) ||
            dfsHelper(matrix, path, i-1, j, rows, cols, visited, count) ||
            dfsHelper(matrix, path, i, j+1, rows, cols, visited, count) ||
            dfsHelper(matrix, path, i, j-1, rows, cols, visited, count) {
            return true
         }
      }
   }
   return false
}
```

时间复杂度是$$O(n^2)$$。

## 机器人的运动范围

地上有一个 m 行 n 列的方格。一个机器人从坐标(0, 0)的格子开始移动，它每次可以向左、右、上、下移动一格，但不能进入行坐标和列坐标的数位之和大于大于 k 的格子。例如，当 k 为 18 时，机器人能够进入方格(35, 37)，因为 3+5+3+7=18。但它不能进入方格(35, 38)，因为 3+5+3+8=19。请问机器人能够到达多少个格子？

### 分析

类似的题目，依旧是搜索问题，只不过多了一些判断条件。

```go
func CountCells(m, n, k int) int {
   if k < 0 || m <= 0 || n <= 0 {
      return 0
   }
   visited := make([][]bool, m)
   for i := range visited {
      visited[i] = make([]bool, n)
   }
   dir := [][]int{{0, 1}, {0, -1}, {1, 0}, {-1, 0}}
   q := make([][]int, 0)
   q = append(q, []int{0, 0})
   visited[0][0] = true
   count := 0
   for len(q) > 0 {
      head := q[0]
      q = q[1:]
      count++
      for j := range dir  {
         x, y := head[0]+dir[j][0], head[1]+dir[j][1]
         if x >= 0 && x < m && y >= 0 && y < n && check(x, y, k) && !visited[x][y] {
            q = append(q, []int{x, y})
            visited[x][y] = true
         }
      }
   }
   return count
}

func check(x int, y int, k int) bool {
   // 已经考虑了 x，y 为 0 的情况
   count := 0
   for d := x; d > 0; d /= 10 {
      count += d % 10
      if count > k {
         return false
      }
   }
   for d := y % 10; d > 0; d /= 10 {
      count += d
      if count > k {
         return false
      }
   }
   return true
}
```

## 剪绳子

给你一根长度为 n 的绳子，请把绳子剪成 m 段（m、n 都是整数，n > 1 并且 m > 1），每段绳子的长度记为 k[0]，k[1]，……。请问所有绳子长度的最大乘积是多少？

### 分析

1.动态规划

假如定义函数 f(n) 为把长度为 n 的绳子剪成若干段之后各段长度乘积的最大值。在剪第一段的时候，我们有 n - 1 种可能的选择没也就是剪出来的第一段绳子的可能长度分别为1，2，……，n-1。因此， f(n)=max(f(i)*f(n-i))。于是这个问题就被分解成了若干个小的子问题，而每个子问题又可以继续分解，并且把它们各自的最优解组合起来之和就达到了整个问题的最优解。于是可以用动态规划的方法。

为了提高效率，采用记忆化的方法将一些中间结果保存下来，避免重复计算。当绳子的长度为 2 时，只可能剪成长度都为 1 的两段，因此 f(2) 等于 1。当绳子的长度为 3 时，可能把绳子剪成长度为 1 和长度为 2 的两段或者是长度为 1 的三段，此时最大长度乘积为 2。

```go
// 该解法存疑
func DPMaxProduct(n int) int {
    // 边界情况
   if n < 2 {
      return 0
   }
   if n == 2 {
      return 1
   }
   if n == 3 {
      return 2
   }
    // products 数组用来存储 f(n)
   products := make([]int, n+1)
   products[1], products[2] = 1, 2
   for i := 3; i <= n; i++ {
      max := 0
      for j := 1; j <= n/2; j++ {
         tmp := products[j] * products[n-j]
         if tmp > max {
            max = tmp
         }
      }
      products[i] = max
   }
   return products[n]
}
```

时间复杂度是$$O(n^2)$$，空间复杂度是$$O(n)$$。

2.贪婪算法

当绳子长度$$n≥5$$时，显然$$2(n-2)>n$$并且$$3(n-3)>n$$。也就是说，当剩下的绳子长度大于或者等于 5 时，应该剪成长度为 3 或 2 的段。$$n=5$$时，$$3(n-3)≥2(n-2)$$，所以应该尽可能多地剪成长度为 3 的段。

当$$n=4$$时，显然$$2×2$$是最优解。实际上这个时候绳子没必要剪，但是题目要求至少剪一刀。

这种方法并没有尝试每一种可能，而是在每一步都要找到最优解，因此这是一种贪婪算法。

```go
func GreedyProduct(n uint) uint {
	if n < 2 {
		return 0
	}
	if n == 2 {
		return 1
	}
	if n == 3 {
		return 2
	}
	// 先尽可能剪成长度为 3 的段
	numOf3 := n / 2 // 长度为 3 的段的数量
	// 如果原来长度为 4 的段，此时不是剪成长度为 3 的段
	// 而是要剪成长度为 2 的段
	if 3*numOf3+1 == n {
		numOf3--
	}
	// 剪成长度为 2 的段
	numOf2 := uint((n - 3*numOf3) / 2) // 长度为 2 的段的数量
	return 1 << numOf2 * uint(math.Pow(3.0, float64(numOf3)))
}
```

## 二进制中 1 的个数

请实现一个函数，输入一个整数，输出该数二进制中 1 的个数。

### 分析

（1）常规解法

最直接的想法就是把输入的数用除 k 取余法转换成二进制，这种方法的时间复杂度显然是线性的。注意如果输入的数是负数，那么最高位多一个 1.

```GO
func CountOnesInBinary(n int) int {
   count := 0
   // 如果为负数，那么就是高位多一个 1
   if n < 0 {
      count++
      n = -n
   }
   for n != 0 {
      if n%2 == 1 {
         count++
      }
      n /= 2
   }
   return count
}
```

还有一种做法，首先把输入的数和 1 做位与运算，判断最低位是不是 1；然后将 1 左移 1 位，做位与运算，判断次低位是否为 1 ……这种做法的时间复杂度也是线性的，与整数的位数有关（64 位整数就要循环 64  次）。

（2）技巧解法

**把一个数和它减去 1 的结果做位与运算，相当于把这个数最右边的 1 变为 0**。基于这个事实，可以不断进行这样的操作，总共能进行多少次，就说明有多少个 1.

```go
func CountOnesInBinary(n int) int {
	count := 0
	if n < 0 {
		count++
		n = -n
	}
	for n != 0 {
		count++
		n = (n-1) & n
	}
	return count
}
```

## 数值的整数次方

实现乘方函数，不得使用库函数，不用考虑大数的问题。

### 分析

问题本身并不难，但是要考虑到边界情况：当指数为 0 或者负数时，底数不能为 0。

```go
func Power(base, exp float64) (float64, error) {
	// 指数为 0
	if exp == 0.0 {
		if base == 0.0 {
			return 0, errors.New("base-zero exponential of 0 is invalid")
		} else {
			return 1, nil
		}
	}
	// 指数为负数
	neg := false
	if exp < 0.0 {
		if base == 0.0 {
			return 0, errors.New("base-zero exponential of a negative value is invalid")
		}
		exp = -exp
		neg = true
	}
	times := int(exp)
	n := base
	for i := 1; i < times; i++ {
		n *= base
	}
	if neg {
		return 1.0 / n, nil
	} else {
		return n, nil
	}
}
```

时间复杂度为$$O(n)$$，空间复杂度为$$O(1)$$。

这个解法的时间复杂和指数绝对值的大小正相关，指数绝对值越大，循环次数越多。

假设要计算 32 次方，如果已经知道了 16 次方，那么再平方就可以得到 32 次方；而 16 次方又是 8 次方的平方……

![剑指 offer 16_1](img/剑指 offer 16_1.jpg)

这个公式可以用递归实现：

```go
func Power(base, exp float64) (float64, error) {
	// 指数为 0
	if exp == 0.0 {
		if base == 0.0 {
			return 0, errors.New("base-zero exponential of 0 is invalid")
		} else {
			return 1, nil
		}
	}
	// 指数为负数
	neg := false
	if exp < 0.0 {
		if base == 0.0 {
			return 0, errors.New("base-zero exponential of a negative value is invalid")
		}
		exp = -exp
		neg = true
	}
	n := power(base, int(exp))
	if neg {
		return 1.0 / n, nil
	} else {
		return n, nil
	}
}

func power(base float64, exp int) float64 {
	if exp == 0 {
		return 1
	}
	if exp == 1 {
		return base
	}
	// 右移 1 位就是除以 2
	n := power(base, exp>>1)
	n *= n
	// 与 1 做位与运算，结果为 1 说明是奇数，为 0 说明时偶数
	if exp&1 == 1 {
		n *= base
	}
	return n
}
```

这个算法的时间复杂度是$$O(logn)$$。注意一些位运算的技巧：

- `1<<n`就是计算 2 的 n 次方
- `n>>1`就是将 n 除以 2
- `n&1`可以用来判断 n 的奇偶性，结果为 1 则为奇
- `n&(n-1)`就是将 n 最低位的 1 变为 0

## 打印从 1 到最大的 n 位数

输入数字 n，按顺序打印出从 1 到最大的 n 位十进制数。比如输入 3，则打印出 1、2、3 一直到最大的三位数 999。

### 分析

解决这题关键是要意识到当 n 很大时，不管是用什么内建的整数类型都有可能溢出。这题要求的是打印整数，所以不妨用字符串来操作，只不过要在字符串上模拟整数加 1 的运算。

```go
func PrintNumbers(n int64) {
	if n <= 0 {
		return
	}
	// n 位最大整数
	max := ""
	for i := int64(0); i < n; i++ {
		max += "9"
	}
	// 依次打印 1 - n 
	num := make([]byte, 0)
	num = append(num, '0')
	for string(num) != max {
		flag := false // 判断是否有进位
		// 从最低位开始加 1
		for i := len(num) - 1; i >= 0; i-- {
			// 如果当前位上的数已经是 9，就说明要进 1
			if num[i] == '9' {
				num[i] = '0'
				flag = true
			} else {
				num[i]++
			}
			if !flag {
				// 不用进位就说明运算结束
				break
			} else {
				// 如果当前是第一位，那就说明从下一个数开始就要多 1 位
				if i == 0 {
					num = append([]byte{'0'}, num...)
					i++
                }
				flag = false
			}
		}
		fmt.Println(string(num))
	}
}
```

时间复杂度是$$O(n)$$，空间复杂度是$$O(n)$$。

## 删除链表的节点

### 1. 给定单向链表的头指针和一个节点指针，定义一个函数在$O(1)$时间内删除该节点。

#### 分析

按照常规思路，找到被删除的节点就需要$$O(n)$$的时间，但是这题的特别之处在于题目已经给出了需要删除的节点的指针，所以可以这么考虑：

- 如果链表中只有一个节点，直接把头节点置为空
- 如果要删除的是尾节点，那么要先找到尾节点的前一个节点，把这个节点的 Next 指针置为空
- 如果要删除的是中间节点，那么可以把要删除的节点的下一个节点的值赋给它，然后把它的指针指向下下个节点，这样也就相当于删除了它

```go
func DeleteNode(head *ListNode, node *ListNode) {
	if head == nil || node == nil {
		return
	}
	// 链表只有一个节点
	if head == node {
		head = nil
		return
	}
	// 要删除的节点是尾节点
	if node.Next == nil {
		cur := head
		for cur.Next != node {
			cur = cur.Next
		}
		cur.Next = nil
		return
	}
	// 一般情况
	node.Key = node.Next.Key   // 赋值
	node.Next = node.Next.Next // 修改指针
}
```

在上面的代码中并没有考虑要删除的节点在链表中这种情况，但是因为时间复杂度必须是$$O(1)$$，所以没法检验。

### 2. 在一个排序的链表中，如何删除重复的节点？

```go
func DeleteDuplicateListNodes(head *ListNode) *ListNode {
   // 链表为空或者只有一个节点
   if head == nil || head.Next == nil {
      return head
   }
   dump := &ListNode{Next: head} // 辅助节点，便于处理从头节点开始一直都是重复的节点的情况
   pre, cur := dump, head
   var dup int
   for cur != nil && cur.Next != nil {
      if cur.Key == cur.Next.Key {
         // 删除重复的节点，注意重复的节点可能是连续多个
         // 注意尾节点也是重复节点的情况，先判断 cur 是否为空
         dup = cur.Key
         for cur != nil && cur.Key == dup {
            pre.Next, cur = cur.Next, cur.Next
         }
      } else {
         pre, cur = cur, cur.Next
      }
   }
   return dump.Next
}
```

时间复杂度是$$O(n)$$，空间复杂度是$$O(1)$$。

## 正则表达式的匹配

请实现一个函数用来匹配包含‘.’和'*'的正则表达式，前者表示任意一个字符，后者表示它前面的字符可以出现任意次（含 0 次）。在本题中，匹配是指字符串的所有字符匹配整个模式，比如字符串“aaa”与模式"a.a"和"ab\*ac\*a"匹配，但与"aa.a"和“ab\*a”均不匹配。

### 分析

```go
func Match(s, pattern string) bool {
   if s == "" && pattern == "" {
      return true
   }
   if s != "" && pattern == "" {
      return false
   }
   if pattern[1] == '*' {
      if pattern[0] == s[0] || (pattern[0] == '.' && s != "") {
         // 进入下一个状态
         return Match(s[1:], pattern[2:]) ||
            // 留在当前状态
            Match(s[1:], pattern) ||
            // 忽略一个 *
            Match(s, pattern[2:])
      } else {
         // 忽略一个 *
         return Match(s, pattern[2:])
      }
   }
   if s[0] == pattern[0] || (pattern[0] == '.' && s != "") {
      return Match(s[1:], pattern[1:])
   }
   return false
}
```

## 表示数值的字符串

请实现一个函数用来判断字符串是否表示数值（包括整数和小数）。例如，字符串“+100”、“5e2”、“-123”、“3.146”及“-1E-16”都表示数值，但是“12e"、“1a3.14”、“1.2.3”、“+-5”及“12e+5.4”都不是。

### 分析

首先尽可能多地扫描数字，如果遇到小数点，那么开始扫描表示小数的部分；如果遇到‘e’或者‘E’就开始扫描表示指数的部分。

```go
func IsNumeric(s string) bool {
	if s == "" {
		return false
	}
	numeric, _s := scanInteger(s)
	s = _s
	// 扫描小数部分
	if s != "" && s[0] == '.' {
		s = s[1:]
		res, _s := scanUnsignedInteger(s)
		s = _s
		// 小数可以前后至少有一个部分有数字就行
		numeric = res || numeric
	}
	if s != "" && (s[0] == 'e' || s[0] == 'E') {
		s = s[1:]
		res, _s := scanInteger(s)
		s = _s
		// e/E 前面必须有数值，后面必须是整数
		numeric = numeric && res
	}
	return numeric && s == ""
}

// 扫描数字部分（带符号）
func scanInteger(s string) (bool, string) {
	if s != "" && (s[0] == '+' || s[0] == '-') {
		s = s[1:]
	}
	return scanUnsignedInteger(s)
}

// 扫描无符号整数
func scanUnsignedInteger(s string) (bool, string) {
	i := 0
	for ; i < len(s); i++ {
		if s[i] < '0' || s[i] > '9' {
			break
		}
	}
	if i > 0 {
		return true, s[i:]
	} else {
		return false, s
	}
}
```

因为是顺序扫描输入的字符串，时间复杂度是$$O(n)$$，空间复杂度$$O(1)$$。

## 调整数组顺序使奇数位于偶数前面

输入一个整数数组，实现一个函数来调整数组中数字的顺序，使得所有奇数在前，偶数在后。

### 分析

（1）直接解法

类似快速排序，利用两个指针，一个从后往前，一个从前往后，如果发现奇数在后面而偶数在前面，那么就交换。

```go
func SwapOddsAndEvens(a []int) []int {
	if len(a) == 0 {
		return a
	}
	i, j := 0, len(a)-1
	for i < j {
		// 向后移动 i 直到发现一个偶数
		for i < len(a) && a[i]&1 == 1 {
			i++
		}
		// 向前移动 j 直到发现一个奇数
		for j >= 0 && a[j]&1 != 1 {
			j--
		}
		if i < j {
			a[i], a[j] = a[j], a[i]
		}
	}
	return a
}
```

时间复杂度是$$O(n)$$，空间复杂度是$$O(1)$$。

（2）抽象

如果题目修改成要求负数在前非负数在后，或者能被 3 整除的数在前不能被 3 整除的数在后等等，思路都是一样的，只不过是判断的条件发生了变化，这样就可以做一层抽象，以一个函数作为参数，作为判断条件。 

```go
func SwapNumbers(a []int, f func(int) bool) []int {
   if len(a) == 0 {
      return a
   }
   i, j := 0, len(a)-1
   for i < j {
      for i < len(a) && f(a[i]) {
         i++
      }
      for j >= 0 && !f(a[j]) {
         j--
      }
      if i < j {
         a[i], a[j] = a[j], a[i]
      }
   }
   return a
}
```

## 链表中倒数第 k 个节点

输入一个链表，输出该链表中倒数第 k 个节点。为了符合习惯，本题从 1 开始记树，即链表的尾节点是倒数第 1 个节点。例如，一个链表有 6 个节点，从头节点开始，它们的值依次是 1，2，3，4，5，6。这个链表的倒数第 3 个节点是值为 4 的节点。

### 分析

典型的双指针类型题。但是要考虑一些特殊情况：

- 链表为空
- 参数 k 不是正树
- 链表中节点数小于 k

```go
func CountDownToKInList(head *ListNode, k int) *ListNode {
	if head == nil || k <= 0 {
		// 链表为空或参数 k 不是正数
		return nil
	}
	i := 0
	slow, fast := head, head
	for ; fast != nil && i < k; fast, i = fast.Next, i+1 {
	}
	if i < k {
		// 链表中的节点数小于 k
		return nil
	}
	for ; fast != nil && slow != nil; fast, slow = fast.Next, slow.Next {
	}
	return slow
}
```

时间复杂度为$$O(n)$$，空间复杂度是$$O(1)​$$。

## 链表中环的入口节点

如果一个链表中包含环，如何找出环的入口节点？

### 分析

[LeetCode](<https://leetcode.com/problems/linked-list-cycle-ii/>) 上有一模一样的题。

```go
func detectCycle(head *ListNode) *ListNode {
    if p := detect(head); p != nil {
        return start(p, head)
    } else {
        return nil
    }
}

// Floyd 算法检测环是否存在
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

如果更进一步要求求出环中节点的数量，那也很容易，因为已经知道了环的入口。

## 反转链表

定义一个函数，输入一个链表的头节点，反转该链表并输出反转后链表的头节点。

### 分析

经典问题，一定要背下模板。

（1）非递归版

```go
func ReverseListIteration(head *ListNode) *ListNode {
	if head == nil || head.Next == nil {
		// 链表为空或者只有一个节点
		return head
	}
	pre, cur := head, head.Next
	var tmp *ListNode
	for cur != nil {
		tmp = cur.Next // 记录当前节点的位置
		cur.Next = pre // 当前节点指向前一个节点
		pre, cur = cur, tmp
	}
	head.Next = nil // 记得让原来的首结点指向 nil
	return pre
}
```

时间复杂度是$$O(n)$$，空间复杂度是$$O(1)$$。

（2）递归版

```go
func ReverseListRecursion(head *ListNode) *ListNode {
	if head == nil || head.Next == nil {
		return head
	}
	// 从最后两个节点开始向前反转
	newHead := ReverseListRecursion(head.Next)
	// 让当前节点的下一个节点指向当前节点，当前节点指向 nil
	head.Next.Next, head.Next = head, nil
	return newHead
}
```

时间复杂度是$$O(n)$$，空间复杂度是$$O(n)$$。

## 合并两个排序的链表

输入两个递增排序的链表，合并这两个链表并使新链表中的节点依然是递增有序的。

### 分析

采用归并排序的思想即可。不过要注意链表为空的情况。

```go
func MergeTwoLists(l1, l2 *ListNode) *ListNode {
	if l1 == nil || l2 == nil {
        // 存在链表为空的情况
		if l1 != nil {
			return l1
		} else if l2 != nil {
			return l2
        } else {
            return nil
        }
	}
	dump := new(ListNode)
	cur := dump
	p, q := l1, l2
	for p != nil && q != nil {
		if p.Key < q.Key {
			cur.Next = p
			p = p.Next
		} else {
			cur.Next = q
			q = q.Next
		}
		cur = cur.Next
	}
	for p != nil {
		cur.Next = p
		cur, p = cur.Next, p.Next
	}
	for q != nil {
		cur.Next = q
		cur, q = cur.Next, q.Next
	}
	return dump.Next
}
```

时间复杂度$$O(n)$$，空间复杂度$$O(1)$$。

## 树的子结构

输入两棵二叉树 A 和 B，判断 B 是不是 A 的子树。二叉树节点定义为：

```pseudocode
struct BinaryTreeNode {
    double Value
    BinaryTreeNode Left
    BinaryTreeNode Right
}
```

### 分析

 首先判断根节点，然后再依次往下继续判断。

这道题还有一个需要注意的地方就是浮点数的比较：在计算机内部浮点数是有误差的，不能用等号判断浮点数是否相等，**而要用差的绝对值是否足够小来判断**。

```go
func FindSubtree(r1, r2 *BinaryTreeNode) bool {
	res := false
	if r1 != nil && r2 != nil {
		// 先找出 r1 的那一部分和 r2 的根节点相同
		if floatEqual(r1.Value, r2.Value) {
			res = findSubtreeHelper(r1, r2)
		}
		if !res {
			res = FindSubtree(r1.Left, r2)
		}
		if !res {
			res = FindSubtree(r1.Right, r2)
		}
	}
	return res
}

func findSubtreeHelper(r1 *BinaryTreeNode, r2 *BinaryTreeNode) bool {
	if r2 == nil {
		return true
	}
	if r1 == nil {
		return true
	}
	if !floatEqual(r1.Value, r2.Value) {
		return false
	}
	// 继续判断向下比较左右子树
	return findSubtreeHelper(r1.Left, r2.Right) && findSubtreeHelper(r1.Right, r2.Right)
}

// 比较浮点数的模板
func floatEqual(f1 float64, f2 float64) bool {
	EPSILON := 0.00000001
	if math.Abs(f1-f2) < EPSILON && math.Abs(f2-f1) < EPSILON {
		return true
	} else {
		return false
	}
}
```

