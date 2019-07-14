## Sort

### Merge Sort

#### Definition

- A comparison based sorting algorithm
  - Divides entire dataset into groups of at most two.
  - Compares each number one at a time, moving the smallest number to left of the pair.
  - Once all pairs sorted it then compares left most elements of the two leftmost pairs creating a sorted group of four with the smallest numbers on the left and the largest ones on the right.
  - This process is repeated until there is only one set.

#### Key Points

- Know that it divides all the data into as small possible sets then compares them.
- Stable.
- Use extra space.

#### Big O Efficiency

- Best Case Sort: $O(n)$
- Average Case Sort: $O(n log n)$
- Worst Case Sort: $O(nlog n)$

#### Code Template

```go
func MergeSort(arr []int) []int {
    if n := len(arr); n < 2 {
        return arr
    } else {
        mid := n / 2
        return merge(MergeSort(arr[:mid]), MergeSort(arr[mid:]))
    }
}

func merge(arr1, arr2 []int) []int {
    n := len(arr1) + len(arr2)
    tmp := make([]int, n, n)
    for i, j, k := 0, 0, 0; k < n; k++ {
        if i >= len(arr1) && j < len(arr2) {
            tmp[k] = arr2[j]
            j++
        } else if j >= len(arr2) && i < len(arr1) {
            tmp[k] = arr1[i]
            i++
        } else if arr1[i] < arr2[j] {
            tmp[k] = arr1[i]
            i++
        } else {
            tmp[k] = arr2[j]
            j++
        }
    } 
    return tmp
} 
```

### Quicksort

#### Definition

- A comparison based sorting algorithm
  - Divides entire dataset in half by selecting the average element and putting all smaller elements to the left of the average.
  - It repeats this process on the left side until it is comparing only two elements at which point the left side is sorted.
  - When the left side is finished sorting it performs the same operation on the right side.
- Computer architecture favors the quicksort process.

#### Key Points

- While it has the same Big O as (or worse in some cases) many other sorting algorithms it is often faster in practice than many other sorting algorithms, such as merge sort.
- Know that it halves the data set by the average continuously until all the information is sorted.
- In place.
- Not stable.

#### Big O Efficiency

- Best Case Sort: $O(logn)$
- Average Case Sort: $O(nlogn)$
- Worst Case Sort: $O(n^2)$

#### Code Template

```go
func QuickSort(arr []int) {
    if len(arr) < 2 {
        return arr
    }
    // Partition
    left, right := 0, len(arr)-1
    pivot := rand.Intn(len(arr))
    // Move pivot to the end
    arr[pivot], arr[right] = arr[right], arr[pivot]
    for i := range arr {
        if arr[i] < arr[right] {
            arr[left], arr[i] = arr[i], arr[left]
            left++
        }
    }
    // Move pivot to the correct postion
    arr[left], arr[right] = arr[right], arr[left]
    // Recursion
    QuickSort(arr[:left])
    QuickSort(arr[left+1:])
    return a
}
```

### Heapsort

#### Definition

- A comparison based algorithm.
  -  It divides its input into a sorted and an unsorted region, and it iteratively shrinks the unsorted region by extracting the largest element and moving that to the sorted region.

#### Key Points

- The heapsort algorithm can be divided into two parts.
  - In the first step, a heap is built out of the data.
  - In the second step, a sorted array is created by repeatedly removing the largest element from the heap (the root of the heap), and inserting it into the array.
- In place.
- Not stable.

#### Big O Efficiency

- Best Case: $O(n)$
- Average Case: $O(nlogn)$
- Worst Case: $O(nlogn)$

#### Code Template

```go
func Heapsort(arr []int) []int {
    for i := len(arr)/2; i < len(arr); i++ {
        heapify(arr, i)
    }
    for i := len(arr)-1; i > 0; i-- {
        arr[0], arr[i] = arr[i], arr[0]
        heapify(arr[:i], 0)
    } 
    return arr
}

// heapify will build a max-heap
func heapify(arr []int, root int) []int {
    for parent, child := root, root*2+1; child < len(arr); child = parent*2+1 {
        if child+1 < len(arr) && arr[child] < arr[child+1] {
            child++
        }
        if arr[parent] > arr[child] {
            break
        }
        arr[parent], arr[child] = arr[child], arr[parent]
        parent = child
    }
}
```

### Summary

| ALGORITHM   | IN PLACE           | STABLE             | BEST         | AVERAGE    | WORST              |
|-------------|--------------------|--------------------|--------------|------------|--------------------|
| Merge Sort  | :x:                | :heavy_check_mark: | $O(n)$       | $O(nlogn)$ | $O(nlogn)$         |
| Quicksort   | :heavy_check_mark: | :x:                | $O(nlogn)$   | $O(nlogn)$ | $O(n^2)$           |
| Heapsort    | :heavy_check_mark: | :x:                | $O(n)$       | $O(nlogn)$ | $O(nlogn)$         |
| Bubble Sort | :heavy_check_mark: | :heavy_check_mark: | $O(n)$       | $O(n^2)$   | $O(n^2)$           |
| Shellsort   | :heavy_check_mark: | :x:                | $O(nlog_3n)$ | $N/A$      | $O(n^\frac{3}{2})$ |

## Search

### Binary Search

#### Definition

- An algorithm finds the position of a target value within a sorted array.

  - Binary search compares the target value to the middle element of the array. 

    - If the target value matches the middle element, its position in the array is returned.
    - If the target value is less than the middle element, the search continues in the lower half of the array. 
    - If the target value is greater than the middle element, the search continues in the upper half of the array. 

  - By doing this, the algorithm eliminates the half in which the target value cannot lie in each iteration.

#### Key Points

- It works on sorted arrays.

- Even though the idea is simple, implementing binary search correctly requires attention to some subtleties about its exit conditions and midpoint calculation.

#### Big O Efficiency

- Best Case: $O(1)$
- Average Case: $O(logn)$
- Worst Case: $O(logn)$

#### Code Template

```go
// BinarySearch returns the index of target if it's present in the array
// if not, return -1
func BinarySearch(nums []int, target int) int {
    for low, high := 0, len(nums)-1; low <= high; {
        if mid := low + (high-low)>>1; nums[mid] == target {
            return mid
        } else if nums[mid] < target {
            low = mid + 1
        } else {
            high = mid - 1
        }
    }
    return -1
}
```

### Binary Search Tree

#### Definition

- Is a rooted binary tree. The tree additionally satisfies the binary search property, which states that the key in each node must be greater than or equal to any key stored in the left sub-tree, and less than or equal to any key stored in the right sub-tree.

#### Key Points

- Binary search trees are a fundamental data structure used to construct more abstract data structures such as sets, multisets, and associative arrays.

- The major advantage of binary search trees over other data structures is that the related sorting algorithms and search algorithms such as in-order traversal can be very efficient.

- The shape of the binary search tree depends entirely on the order of insertions and deletions, and can become degenerate.

#### Big O Efficiency

- Worst Case:

  - Search: $O(n)$, Insertion: $O(n)$

- Average Case:

  - Seach: $O(logn)$, Insertion: $O(logn)$

#### Code Template

```go
type BSTNode struct {
    Key   int
    Val   int
    Left  *BSTNode
    Right *BSTNode
}

func search(key int, root *BSTNode) int {
    cur := root
    for cur != nil {
        if cur.Key == key {
            return cur.Val
        } else if cur.Key < key {
            cur = cur.Right
        } else {
            cur = cur.Left
        }
    }
    return cur
}

func insert(root *BSTNode, key int, value int) *BSTNode {
    if root == nil {
        return &BSTNode{key, value, nil, nil}
    }
    if key == root.key {
        root.Val = value
    } else if key < root.key {
        root.Left =  insert(root.Left, key ,value)
    } else {
        root.Right = insert(root.Right, key, value)
    }
    return root
}
```

### AVL Tree

#### Definition

- Is a self-balancing binary search tree.

  - The heights of the two child subtrees of any node differ by at most one; if at any time they differ by more than one, rebalancing is done to restore this property.

#### Key Points

- Lookup, insertion, and deletion all take $O(log n)$ time in both the average and worst cases, where n is the number of nodes in the tree prior to the operation. 

- Insertions and deletions may require the tree to be rebalanced by one or more tree rotations.

#### Big O Efficiency

- Worst Case: $O(logn)$
- Average Case: $O(logn)$

#### Summary

| DATA STRUCTURE     | WORST CASE | AVERAGE CASE |
|--------------------|------------|--------------|
| Binary Search      | $O(logn)$  | $O(logn)$    |
| Binary Search Tree | $O(n)$     | $O(logn)$    |
| AVL                | $O(logn)$  | $O(logn)$    |

## Tree

### Traversal

#### Definition

- Is a form of graph traversal and refers to the process of visiting (checking and/or updating) each node in a tree data structure, exactly once.

#### Key Points

- Such traversals are classified by the order in which the nodes are visited.

  - Pre-order, in-oder, post-order, level-order

- Pre-order traversal while duplicating nodes and edges can make a complete duplicate of a binary tree. It can also be used to make a prefix expression (Polish notation) from expression trees: traverse the expression tree pre-orderly. For example, traversing the depicted arithmetic expression in pre-order yields "+ * 1 - 2 3 + 4 5".

- In-order traversal is very commonly used on binary search trees because it returns values from the underlying set in order, according to the comparator that set up the binary search tree.

- Post-order traversal while deleting or freeing nodes and values can delete or free an entire binary tree. It can also generate a postfix representation (Reverse Polish notation) of a binary tree. Traversing the depicted arithmetic expression in post-order yields "1 2 3 - * 4 5 + +".

#### Big O Efficiency

- $O(n)$

#### Code Template

```go
type TreeNode struct {
    Val   interface{}
    Left  *TreeNode
    Right *TreeNode
}

func preorder(root *TreeNode) {
    if root == nil {
        return
    }
    visit(root)
    preorder(root.Left)
    preorder(root.Right)
}

func inorder(root *TreeNode) {
    if root == nil {
        return
    }
    inorder(root.Left)
    visit(root)
    inorder(root.Right)
}

func postorder(root *TreeNode) {
    if root == nil {
        return
    }
    postorder(root.Left)
    postorder(root.Right)
    visit(root)
}

func levelOrder(root *TreeNode) {
    if root == nil {
        return
    }
    queue := []*TreeNode{root}
    for len(queue) > 0 {
        node := queue[0]
        queue = queue[1:]
        visit(node)
        if node.Left != nil {
            queue = append(queue, node.Left)
        }
        if node.Right != nil {
            queue = append(queue, node.Right)
        }
    }
}
```

