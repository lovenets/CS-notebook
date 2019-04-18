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







