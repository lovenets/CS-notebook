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

## Graph

### Traversal

#### Definition

- Is the process of visiting (checking and/or updating) each vertex in a graph.
  - Such traversals are classified by the order in which the vertices are visited.

#### Key Points

- Tree traversal is a special case of graph traversal.
- Unlike tree traversal, graph traversal may require that some vertices be visited more than once, since it is not necessarily known before transitioning to a vertex that it has already been explored.
  
  - But usually we record each vertex we have visited and by doing so we can assure each vertex is visited exactly once and we can also avoid stucking in any cycles. 

- Traversals are classified by the order in which the vertices are visited.

  - DFS visits the child vertices before visiting the sibling vertices;
  - BFS visits the sibling bertices before visiting the child vertices.

#### Big O Efficiency

- DFS
  
  - Adjacent list: $O(E+V)$, Adjacent matrix: $O(V^2)$

- BFS
  
  - Adjacent list: $O(E+V)$, Adjacent matrix: $O(V^2)$  

#### Code Template

```go
// g is the adjacent matrix
// g[x][y] == 1 means there is an edge 
// from vertex i to j
func dfs(g [][]int, start int) {
    visited := make([]bool, len(g))

    var helper func(int)
    helper = func(i int) {
        visited[j] = true
        for j := range g[i] {
            if g[i][j] == 1 && !visited[j] {
                helper(j)
            }
        }
    }

    helper(start)
}

func bfs(g [][]int, start int) {
    visited := make([]bool, len(g))
    queue := []int{start}
    visited[start] = true
    for len(queue) > 0 {
        i := queue[0]
        queue = queue[1:]
        for j := range g[i] {
            if g[i][j] == 1 && !visited[j] {
                queue = append(queue, j)
                visited[j] = true
            }
        }
    }
}
```

### Minimum Spanning Tree

#### Definition

- Is a subset of the edges of a connected, edge-weighted undirected graph that connects all the vertices together, without any cycles and with the minimum possible total edge weight.

#### Key Points

- If there are n vertices in the graph, then each spanning tree has n âˆ’ 1 edges.

- If each edge has a distinct weight then there will be only one, unique minimum spanning tree.

- If the weights are positive, then a minimum spanning tree is in fact a minimum-cost subgraph connecting all vertices. This property can be applied to solve many problems in real world.

  - For example, A minimum spanning tree would be one with the lowest total cost, representing the least expensive path for laying the cable.

- If the minimum cost edge e of a graph is unique, then this edge is included in any MST.

#### Big O Efficiency

- Prim's algorithm

  - Adjacency list: $O(ElogV)$, Adjacency matrix: $O(V^2)$

- Kruskal

  - Adjacency list: $O(ElogV)$, Adjacency matrix: $O(V^2)$

#### Code Template

##### Prim's Algorithm

1. Initialize a tree with a single vertex, chosen arbitrarily from the graph.
2. Grow the tree by one edge: of the edges that connect the tree to vertices not yet in the tree, find the minimum-weight edge, and transfer it to the tree.
3. Repeat step 2 (until all vertices are in the tree).

```go
type Edge struct {
    From   int
    To     int
    Weight int
}

func kruskal(g []Edge) []Edge {
    inMST := make([]bool ,len(g)) // Record whether a vertex is already in MST
    inMST[0] = true
    res := make([]Edge, 0)
    for len(res) < len(g)-1 {
        // Find the edge which connects the tree to a vertex not yet in the tree and has the minimum weight at the same time
        min := Edge{Weight: math.MaxInt64}
        for _, e := range g {
            if inMST[e.From] && !inMST[e.To] && e.Weight < min.Weight {
                min = e
            }
        }
        res = append(res, min)
        inMST[e.To] = true
    }
    return res
}
```

##### Kruskal's Algorithm

1. create a forest F (a set of trees), where each vertex in the graph is a separate tree
2. create a set S containing all the edges in the graph
3. while S is nonempty and F is not yet spanning
  - remove an edge with minimum weight from S
  - if the removed edge connects two different trees then add it to the forest F, combining two trees into a single tree

```go
type Edge struct {
    From   int
    To     int
    Weight int
}

type Edges []Edge

func (e Edges) Len() int {
    return len(e)
}

func (e Edges) Less(i, j int) bool {
    return e[i].Weight < e[j].Weight
}

func (e Edges) Swap(i, j int) {
    e[i], e[j] = e[j], e[i]
}

// Let's say all vertices are marked from 0 to n-1
func Kruskal(g []Edge) []Edge {
    // Sort edges ascendingly by their weights
    sort.Sort(Edges(g))
    // We will use union find to determine 
    // if two vertices are already in the same tree
    uf := union.Init(len(g))
    
    mst := make([]Edge, 0, len(g)-1)
    for _, e := range g {
        if r1, r2 := uf.Search(e.From), uf.Search(e.To); r1 != r2 {
            // This edge connects two different trees, 
            // combine two trees into a single tree
            uf.Merge(r1, r2)
            if mst = append(mst, e); len(mst) == len(g)-1 {
                // If mst has E-1 edges, then we finish the task.
                break
            }
        }
    }
    return mst
}
```

### Shortest Path

#### Definition

- Is the problem of finding a path between two vertices (or nodes) in a graph such that the sum of the weights of its constituent edges is minimized.

#### Key Points

- There are 3 kinds of SP problems:

  - The **single-source shortest path** problem, in which we have to find shortest paths from a source vertex v to all other vertices in the graph.
  - The **single-destination shortest path** problem, in which we have to find shortest paths from all vertices in the directed graph to a single destination vertex v. This can be reduced to the single-source shortest path problem by reversing the arcs in the directed graph.
  - The **all-pairs shortest path** problem, in which we have to find shortest paths between every pair of vertices v, v' in the graph.

- Dijkstra's algorithm solves the single-source shortest path problem with **non-negative** edge weight.

- Floyd–Warshall algorithm solves all pairs shortest paths in graphs which contain **no negative cycles**.

- Bellman–Ford algorithm/SPFA solves the single-source problem if edge weights **may be negative**.

- Don't forget BFS can find shortest paths in **unweighted** graphs.

#### Big O Efficiency

- Dijkstra's algorithm: $O(V^2)$

- Floyd-Warshall algorithm: $O(V^3)$

- Bellman-Ford algorithm: $O(VE)$

### Topological Sort

#### Definition

- Is a linear ordering of its vertices such that for every directed edge <u, v> from vertex u to vertex v, u comes before v in the ordering.

#### Key Points

- A topological ordering is possible if and only if the graph has no directed cycles.

- Any DAG has at least one topological ordering.

- The canonical application of topological sorting is in scheduling a sequence of jobs or tasks based on their dependencies.

#### Big O Efficiency

- Kahn's algorithm: $O(V+E)$

#### Code Template

```go
func KahnTopologicalSort(g [][]int) []int {
    // Find all "start nodes" which have no incoming edges
    indegree := make([]int, len(g))
    for i := range g {
        for j := range g[i] {
            if g[i][j] == 1 {
                indegree[j]++
            }
        }
    }
    start := make([]int, 0) // set of all nodes with no incoming edge
    for i := range indegree {
        if indegree[i] == 0 {
            start = append(start, i)
        }
    }

    sorted := make([]int, 0, len(g)) 
    for len(start) > 0 {
        n := start[0]
        set = start[1:]
        sorted = append(sorted, n)
        for m := range g[n] {
            if g[n][m] == 1 {
                // Remove edge n to m
                if indegree[m]--; indegree[m] == 0 {
                    // If m has no other incoming edges
                    // then it becomes a start node
                    start = append(start, m)
                }
            }
        } 
    }

    if len(sorted) < len(g) {
        // Graph has at least one cycle
        return nil
    } else {
        return sorted
    }
}
```

## Dynamic Programming

### Definition

- It refers to simplifying a complicated problem by breaking it down into simpler sub-problems in a recursive manner. 

- If a problem can be solved optimally by breaking it into sub-problems and then recursively finding the optimal solutions to the sub-problems, then it is said to have optimal substructure.

### Key Points

- Most of DP problems can be solved by following steps:

  - Breaking the whole problem into sub-problems

    - Find the connection between them

  - Finding the optimal solutions to the sub-problems

    - Memoization

- Top-down vs. Bottom-up

  - Top-down: Assuming that we have solved the whole problem, we figure out how we can get answers to sub-problems. Usually done recursively.
  - Bottom-up: We firstly slove sub-problems and then use answers to sub-problems to solve the whole problem. Usually done iteratively.
  - For efficiency, both need memoization.