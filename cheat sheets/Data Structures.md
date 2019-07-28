### Array

#### Definition

- Stores data based on a sequential, most commonly 0 based, index.

#### Key Points

- Optimal for indexing; bad at searching, inserting, and deleting (except at the end).
- Linear arrays or one dimensional arrays, are the most basic.
  - Are static in size, meaning that they are declared with a fixed size.
- Dynamic arrays are like one dimensional arrays, but have reserved space for additional elements.
  - If a dynamic array is full, it copies it's contents to a larger array.
- Two dimensional arrays have x and y indices like a grid or nested arrays.

#### Big O Efficiency

- Indexing
  - Linear array: $O(1)$, Dynamic array: $O(1)$
- Search
  - Linear array:$ O(n)$, Dynamic array: $O(n)$
- Optimized Search 
  - Linear array: $O(log n)$, Dynamic array: $O(log n)$
- Insertion
  - Linear array: $N/A$ Dynamic array:$O(n)$

### Linked List

#### Definition

- Stores data with **nodes** that point to other nodes.

#### Key Points

- Designed to optimize insertion and deletion, slow at indexing and searching.
- **Doubly linked list** has nodes that reference the previous node.
- **Circularly linked list** is simple linked list whose **tail**, the last node, references the **head**, the first node.
- Stack, commonly implemented with linked lists but can be made from arrays too.
  - Stacks are **last in, first out** (LIFO) data structures.
  - Made with a linked list by having the head be the only place for insertion and removal.
- Queues, too can be implemented with a linked list or an array.
  - Queues are a **first in, first out** (FIFO) data structure.
  - Made with a doubly linked list that only removes from head and adds to tail.

#### Big O Efficiency

- Indexing: $O(n)$
- Search: $O(n)$
- Optimized Search: $O(n)$
- Insertion: $O(1)$

### Hash Table

#### Definition

- Stores data with key value pairs.
- Hash functions accept a key and return an output unique only to that specific key.
  - This is known as **hashing**, which is the concept that an input and an output have a one-to-one correspondence to map information.
  - Hash functions return a unique address in memory for that data.

#### Key Points

- Designed to optimize searching, insertion, and deletion.
- Hash collisions are when a hash function returns the same output for two distinct inputs.
  - All hash functions have this problem.
  - This is often accommodated for by having the hash tables be very large or using overflow buckets.
- Hashes are important for associative arrays and database indexing.

#### Big O Efficiency

- Indexing: $O(1)$
- Search: $O(1)$
- Insertion: $O(1)$

### Binary Tree

#### Definition

- Is a tree like data structure where every node has at most two children.

#### Key Points

- Designed to optimize searching and sorting.
- A **degenerate tree** is an unbalanced tree, which if entirely one-sided is a essentially a linked list.
- They are comparably simple to implement than other data structures.
- Used to make binary search trees
  - A binary tree that uses comparable keys to assign which direction a child is.
  - Left child has a key smaller than it's parent node.
  - Right child has a key greater than it's parent node.
  - There can be no duplicate node.

#### Big O Efficiency

- Indexing: $O(logn)$
- Search: $O(logn)$
- Insertion: $O(logn)$

### Trie

#### Definition

- Is a kind of search tree which aims to search words quickly.

#### Key Points

-  All the descendants of a node have a common prefix of the string associated with that node, and the root is associated with the empty string.
- A trie can also be used to replace a hash table.
- A common application of a trie is storing a predictive text or autocomplete dictionary.

#### Big O Efficiency

- Insertion: $O(logn)$
- Search: $O(logn)$
- Delete: $O(logn)$

#### Code Template

```go
type TrieNode struct {
    Children map[rune]*TrieNode
    Value    interface{}
}

func Find(node *TrieNode, key string) interface{} {
    for _, r := range key {
        if _, ok := node.Children[r]; !ok {
            return nil
        }
        node = node.Children[r]
    }
    return node.Value
}

func insert(node *TrieNode, key string, value interface{}) {
    for _, r := range key {
        if _, ok := node.Children[r]; !ok {
            node.Children[r] = &TrieNode{Children: make(map[rune]*TrieNode)}
        }
        node = node.Children[r]
    }
    node.Value = value
}
```

### Union Find

#### Definition

- Is a data structure which tracks a set of elements partitioned into a number of disjoint (non-overlapping) subsets. 

#### Key Points

- It provides near-constant-time operations (bounded by the inverse Ackermann function) to add new sets, to merge existing sets, and to determine whether elements are in the same set.

- Plays a key role in Kruskal's algorithm for finding the minimum spanning tree of a graph.

- Consists of a number of elements each of which stores an id, a parent pointer, and, in efficient algorithms, either a size or a "rank" value.

#### Big O Efficiency

- Search
  
  - Average: $O(\alpha(n))$, Worst: $O(\alpha(n))$

- Merge

  - Average: $O(\alpha(n))$, Worst: $O(\alpha(n))$

#### Code Template

```go
// Union by rank
type UnionFind struct {
    Root []int
    Rank  []int
}

// n is the number of elements
func Init(n int) *UnionFind {
    root := make([]int, n)
    for i := range root {
        root[i] = i
    }
    rank := make([]int, n)
    return &UnionFind{root, rank}
}

func (uf *UnionFind) Search(x int) int {
    if x != uf.Root[x] {
        uf.Root[x] = uf.Search(uf.Root[x])
	}
	return uf.Root[x]
}

func (uf *UnionFind) Merge(x, y int) {
	rx, ry := uf.Search(x), uf.Search(y)
	if rx != ry {
		if uf.Rank[rx] < uf.Rank[ry] {
			rx, ry = ry, rx
		}
		uf.Root[ry] = rx
		if uf.Rank[rx] == uf.Rank[ry] {
			uf.Rank[rx]++
		}
	}
}
```


### Graph

#### Definition

- Is a net like data structure where every node has several neighboring nodes

#### Key Points

- Designed to model complex situations such as transportation system and social network.
- Tree is a special kind of graph.
- Graphs can be divided into two groups: directed and undirected graphs.

#### Big O Efficiency

- Traversal: $O(E+V)$
- Shortest Path
  - Dijkstra: $O(ElogV)$, Floyd-Warshall: $O(n^3)$, Bellman-Ford/SPFA: $O(V(V+E))$
- Minimum Spanning Tree
  - Kruskal: $O(ElogE)$, Prim: $O(ElogV)$

### Heap

#### Definition

- Is a specialized tree-based data structure which is essentially an almost complete tree that satisfies the heap property

#### Key Points

- In a max heap, for any given node C, if P is a parent node of C, then the key (the value) of P is greater than or equal to the key of C.
- In a *min heap*, the key of P is less than or equal to the key of C.
- The heap is one maximally efficient implementation of a [priority queue](https://en.wikipedia.org/wiki/Priority_queue)
- In Go's heap package, a heap is a min-heap "by default".

#### Big O Efficiency

- Insertion: $O(logn)$
- Find-min/max: $O(1)$
- Delete-min/max: $O(logn)$

### String

#### Definition

- Is a traditionally a sequence of characters.

#### Key Points

- Is generally considered as a data type and is often implemented as an [array of bytes/words that stores a sequence of elements, typically characters, using some character encoding.
- Some languages, such as C++ and Ruby, normally allow the contents of a string to be changed after it has been created; these are termed *mutable* strings. 
- In other languages, such as Java and Python, the value is fixed and a new string must be created if any alteration is to be made; these are termed *immutable* strings.





