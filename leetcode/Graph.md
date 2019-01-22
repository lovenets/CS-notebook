#### 1.[Course Schedule](https://leetcode.com/problems/course-schedule/)

There are a total of *n* courses you have to take, labeled from `0` to `n-1`.

Some courses may have prerequisites, for example to take course 0 you have to first take course 1, which is expressed as a pair: `[0,1]`

Given the total number of courses and a list of prerequisite **pairs**, is it possible for you to finish all courses?

**Example 1:**

```
Input: 2, [[1,0]] 
Output: true
Explanation: There are a total of 2 courses to take. 
             To take course 1 you should have finished course 0. So it is possible.
```

**Example 2:**

```
Input: 2, [[1,0],[0,1]]
Output: false
Explanation: There are a total of 2 courses to take. 
             To take course 1 you should have finished course 0, and to take course 0 you should
             also have finished course 1. So it is impossible.
```

**Note:**

1. The input prerequisites is a graph represented by **a list of edges**, not adjacency matrices. Read more about [how a graph is represented](https://www.khanacademy.org/computing/computer-science/algorithms/graph-representation/a/representing-graphs).
2. You may assume that there are no duplicate edges in the input prerequisites.

**My Solution**

Obviously, this is a topological sorting problem. If the directed graph contains any cycles, topological sorting can not be completed so courses can not be finished.

```go
var marked []bool
var onStack []bool
var cycle []int
func canFinish(numCourses int, prerequisites [][]int) bool {
	if len(prerequisites) == 0 {
		return true
	}

	marked = make([]bool, numCourses)
	onStack = make([]bool, numCourses)
	DG := constructDG(numCourses, prerequisites)
	for v := range DG {
		if !marked[v] {
			dfs(DG, v)
		}
	}
	return len(cycle) == 0
}

func dfs(DG map[int][]int, v int) {
	marked[v] = true
	onStack[v] = true
	for _, w := range DG[v] {
		if len(cycle) > 0 {
			return
		} else if !marked[w] {
			dfs(DG, w)
		} else if onStack[w] {
			cycle = make([]int, 0)
			cycle = append(cycle, w)
			cycle = append(cycle, v)
		}
	}
	onStack[v] = false
}

func constructDG(numCourses int, prerequisites [][]int) map[int][]int {
	DG := make(map[int][]int, numCourses)
	for _, val := range prerequisites {
		DG[val[1]] = append(DG[val[1]], val[0])
	}
	return DG
}
```

Time complexity: $$O(v + e)$$, v is the number of vertices and e is the number of edges. 

**Other**

```go
func canFinish(numCourses int, prerequisites [][]int) bool {
	// adjacency matrix
	matrix := make([][]int, numCourses)
	for key, _ := range matrix {
		matrix[key] = make([]int,numCourses)
	}
	// the in degree of vertices, which also means
	// the number of prerequisites
	indegree := make([]int, numCourses)

	// construct the directed graph
	for i := 0; i < len(prerequisites); i++ {
		ready := prerequisites[i][0]
		pre := prerequisites[i][1]
		// in case there are duplicate edges in the input
		if matrix[pre][ready] == 0 {
			indegree[ready]++
		}
		matrix[pre][ready] = 1
	}

	// the number of finished courses
	finished := 0
	// BFS
	// the queue stores courses which can be finished
	queue := make([]int, 0)
	for i, v := range indegree {
		if v == 0 {
			queue = append(queue, i)
		}
	}
	for len(queue) > 0 {
		c := queue[0]
		queue = queue[1:]
		finished++
		for i := 0; i < numCourses; i++ {
			if matrix[c][i] != 0 {
				indegree[i]--
				// now the course i can be finished
				if indegree[i] == 0 {
					queue = append(queue, i)
				}
			}
		}
	}
	return finished == numCourses
}
```

Time complexity: $$O(n^2)$$, n is the number of vertices.

#### 2.[Course Schedule II](https://leetcode.com/problems/course-schedule-ii)

There are a total of *n* courses you have to take, labeled from `0` to `n-1`.

Some courses may have prerequisites, for example to take course 0 you have to first take course 1, which is expressed as a pair: `[0,1]`

Given the total number of courses and a list of prerequisite **pairs**, return the ordering of courses you should take to finish all courses.

There may be multiple correct orders, you just need to return one of them. If it is impossible to finish all courses, return an empty array.

**Example 1:**

```
Input: 2, [[1,0]] 
Output: [0,1]
Explanation: There are a total of 2 courses to take. To take course 1 you should have finished   
             course 0. So the correct course order is [0,1] .
```

**Example 2:**

```
Input: 4, [[1,0],[2,0],[3,1],[3,2]]
Output: [0,1,2,3] or [0,2,1,3]
Explanation: There are a total of 4 courses to take. To take course 3 you should have finished both     
             courses 1 and 2. Both courses 1 and 2 should be taken after you finished course 0. 
             So one correct course order is [0,1,2,3]. Another correct ordering is [0,2,1,3] .
```

**Note:**

1. The input prerequisites is a graph represented by **a list of edges**, not adjacency matrices. Read more about [how a graph is represented](https://www.khanacademy.org/computing/computer-science/algorithms/graph-representation/a/representing-graphs).
2. You may assume that there are no duplicate edges in the input prerequisites.

**Solution**

(1) BFS-based topological sorting

```go
func findOrder(numCourses int, prerequisites [][]int) []int {
	// all vertices's in-degree
	in := make([]int, numCourses, numCourses)
	// the key is the prerequisite
	pres := make(map[int][]int)
	for _, val := range prerequisites {
		in[val[0]]++
		pres[val[1]] = append(pres[val[1]], val[0])
	}

	// those can be finished
	queue := make([]int, 0)
	for v, i := range in {
		if i == 0 {
			queue = append(queue, v)
		}
	}

	// finished courses
	finished := make([]int, 0)
	for len(queue) > 0 {
		// pop the queue
		p := queue[0]
		queue = queue[1:]
		finished = append(finished, p)
		// find courses which can be finished now
		for _, v := range pres[p] {
			in[v]--
			if in[v] == 0 {
				queue = append(queue, v)
			}
		}
	}
	if len(finished) == numCourses {
		return finished
	} else {
		return []int{}
	}
}
```

Time complexity: $$O(V+E)$$, $$V$$ is the number of vertices and $$E$$ is the number of edges.

(2) DFS-based topological sorting

```java
public int[] findOrder(int numCourses, int[][] prerequisites) {
    int[] incLinkCounts = new int[numCourses];
    List<List<Integer>> adjs = new ArrayList<>(numCourses);
    initialiseGraph(incLinkCounts, adjs, prerequisites);
    return solveByDFS(adjs);
}

private void initialiseGraph(int[] incLinkCounts, List<List<Integer>> adjs, int[][] prerequisites){
    int n = incLinkCounts.length;
    while (n-- > 0) adjs.add(new ArrayList<>());
    for (int[] edge : prerequisites) {
        incLinkCounts[edge[0]]++;
        adjs.get(edge[1]).add(edge[0]);
    }
}

private int[] solveByDFS(List<List<Integer>> adjs) {
    BitSet hasCycle = new BitSet(1);
    BitSet visited = new BitSet(adjs.size());
    BitSet onStack = new BitSet(adjs.size());
    Deque<Integer> order = new ArrayDeque<>();
    for (int i = adjs.size() - 1; i >= 0; i--) {
        if (!visited.get(i) && !hasOrder(i, adjs, visited, onStack, order)) return new int[0];
    }
    int[] orderArray = new int[adjs.size()];
    for (int i = 0; !order.isEmpty(); i++) orderArray[i] = order.pop();
    return orderArray;
}

private boolean hasOrder(int from, List<List<Integer>> adjs, BitSet visited, BitSet onStack, Deque<Integer> order) {
    visited.set(from);
    onStack.set(from);
    for (int to : adjs.get(from)) {
        if (!visited.get(to)) {
            if (!hasOrder(to, adjs, visited, onStack, order)) return false;
        } else if (onStack.get(to)) {
            // detect a cycle
            return false;
        }
    }
    onStack.clear(from);
    order.push(from);
    return true;
}
```

Time complexity: $$O(V+E)$$, $$V$$ is the number of vertices and $$E$$ is the number of edges.

#### 3.[Evaluate Division](https://leetcode.com/problems/evaluate-division/)

Equations are given in the format `A / B = k`, where `A` and `B`are variables represented as strings, and `k` is a real number (floating point number). Given some queries, return the answers. If the answer does not exist, return `-1.0`.

**Example:**
Given `a / b = 2.0, b / c = 3.0.` 
queries are: `a / c = ?, b / a = ?, a / e = ?, a / a = ?, x / x = ? .` 
return `[6.0, 0.5, -1.0, 1.0, -1.0 ].`

The input is: `vector<pair<string, string>> equations, vector<double>& values, vector<pair<string, string>> queries `, where `equations.size() == values.size()`, and the values are positive. This represents the equations. Return `vector<double>`.

According to the example above:

```
equations = [ ["a", "b"], ["b", "c"] ],
values = [2.0, 3.0],
queries = [ ["a", "c"], ["b", "a"], ["a", "e"], ["a", "a"], ["x", "x"] ]. 
```

The input is always valid. You may assume that evaluating the queries will result in no division by zero and there is no contradiction.

**Solution**

```go
func calcEquation(equations [][]string, values []float64, queries [][]string) []float64 {
	// construct a directed graph
    // a directed edge's starting vertex is the dividend and end its vertex is divisor 
    // the weight of an edge is quotient
	graph := constructGraph(equations, values)

	res := make([]float64, 0)
	for _, v := range queries {
		res = append(res, pathLength(graph, v[0], v[1]))
	}
	return res
}

// key are the vertices
// values are maps whose keys are adjacent vertices
// and values are corresponding weights 
func constructGraph(equations [][]string, values []float64) map[string]map[string]float64 {
	graph := make(map[string]map[string]float64)

	addEdge := func(f, t string, w float64) {
		if edges, ok := graph[f]; ok {
			edges[t] = w
		} else {
			graph[f] = make(map[string]float64)
			graph[f][t] = w
		}
	}

	for i := 0; i < len(values); i++ {
		f, t := equations[i][0], equations[i][1]
		w := values[i]
		addEdge(f, t, w)
		addEdge(t, f, 1.0/w)
	}

	return graph
}

func pathLength(g map[string]map[string]float64, f string, t string) float64 {
	if _, ok := g[f]; !ok {
		return -1.0
	}
	if f == t {
		return 1.0
	}

	// BFS
	queue := make([][]interface{}, 0)
	queue = append(queue, []interface{}{f, 1.0})
	visited := make(map[string]bool)
	for len(queue) > 0 {
		poll := queue[0]
		queue = queue[1:]
		v := poll[0].(string)
		if v == t {
			return poll[1].(float64)
		}
		visited[v] = true
		for n, w := range g[v] {
			if !visited[n] {
				queue = append(queue, []interface{}{n, poll[1].(float64) * w})
			}
		}
	}

	return -1.0
}
```

Time complexity: $$O(e+v)$$, e is the number of edges and v is the number of vertices.

#### 4.[Find Eventual Safe States](https://leetcode.com/problems/find-eventual-safe-states/)

In a directed graph, we start at some node and every turn, walk along a directed edge of the graph.  If we reach a node that is terminal (that is, it has no outgoing directed edges), we stop.

Now, say our starting node is *eventually safe* if and only if we must eventually walk to a terminal node.  More specifically, there exists a natural number `K` so that for any choice of where to walk, we must have stopped at a terminal node in less than `K` steps.

Which nodes are eventually safe?  Return them as an array in sorted order.

The directed graph has `N` nodes with labels `0, 1, ..., N-1`, where `N` is the length of `graph`.  The graph is given in the following form: `graph[i]` is a list of labels `j` such that `(i, j)`is a directed edge of the graph.

```
Example:
Input: graph = [[1,2],[2,3],[5],[0],[5],[],[]]
Output: [2,4,5,6]
Here is a diagram of the above graph.
```

![Illustration of graph](https://s3-lc-upload.s3.amazonaws.com/uploads/2018/03/17/picture1.png)

**Note:**

- `graph` will have length at most `10000`.
- The number of edges in the graph will not exceed `32000`.
- Each `graph[i]` will be a sorted list of different integers, chosen within the range `[0, graph.length - 1]`.

**Solution**

(1) remove 0 out degree nodes

1. Find nodes with out degree 0, they are terminal nodes, we remove them from graph and they are added to result
2. For nodes who are connected terminal nodes, since terminal nodes are removed, we decrease in-nodes' out degree by 1 and if its out degree equals to 0, it become new terminal nodes
3. Repeat 2 until no terminal nodes can be found.

```go
func eventualSafeNodes(graph [][]int) []int {
	// outDegree[i]: the out degree of vertex i
	outDegree := make([]int, len(graph), len(graph))
	// inVertices[i]: the list of vertices which point to vertex i
	inVertices := make(map[int][]int)
	res := make([]int, 0)
	queue := make([]int, 0)
    
    // find the initial terminal vertices
	for f, e := range graph {
		outDegree[f] = len(e)
		if outDegree[f] == 0 {
			queue = append(queue, f)
		}
		for _, t := range graph[f] {
			inVertices[t] = append(inVertices[t], f)
		}
	}
    // remove the terminal vertices and then update the graph
    // keep looking for terminal vertices until we can't find any more
	for len(queue) > 0 {
		t := queue[0]
		queue = queue[1:]
		res = append(res, t)
		for _, f := range inVertices[t] {
			outDegree[f]--
			if outDegree[f] == 0 {
				queue = append(queue, f)
			}
		}
	}
	sort.Ints(res)
	return res
}
```

Time complexity: $$O(n^2)$$,n is the number of nodes.

(2) DFS

Actually, we are looking for nodes which will not form a circle.

For DFS, we need to do some optimization.When we travel a path, we mark the node with 2 which represents having been visited, and when we encounter a node which results in a cycle, we return false, all node in the path stays 2 and it represents unsafe. And in the following traveling, whenever we encounter a node which points to a node marked with 2, we know it will results in a cycle, so we can stop traveling. On the contrary, when a node is safe, we can mark it with 1 and whenever we encounter a safe node, we know it will not results in a cycle.

```go
const NOT_VISITED = 0
const SAFE = 1
const UNSAFE = 2

func eventualSafeNodes(graph [][]int) []int {
	res := make([]int, 0)
	if len(graph) == 0 {
		return res
	}
	nodeState := make([]int, len(graph), len(graph))
	for i := 0; i < len(graph); i++ {
		if dfs(graph, i, nodeState) {
			res = append(res, i)
		}
	}
	sort.Ints(res)
	return res
}

func dfs(graph [][]int, f int, nodeState []int) bool {
	if nodeState[f] != NOT_VISITED {
		return nodeState[f] == SAFE
	}
	nodeState[f] = UNSAFE
	for _, t := range graph[f] {
		if !dfs(graph, t, nodeState) {
			return false
		}
	}
	nodeState[f] = SAFE
	return true
}
```

Time complexity: $$O(v+e)$$, v is the number of vertices and e is the number of edges.

#### 5. [Is Graph Bipartite?](https://leetcode.com/problems/is-graph-bipartite/)

Given an undirected `graph`, return `true` if and only if it is bipartite.

Recall that a graph is *bipartite* if we can split it's set of nodes into two independent subsets A and B such that every edge in the graph has one node in A and another node in B.

The graph is given in the following form: `graph[i]` is a list of indexes `j` for which the edge between nodes `i` and `j` exists.  Each node is an integer between `0` and `graph.length - 1`.  There are no self edges or parallel edges: `graph[i]` does not contain `i`, and it doesn't contain any element twice.

```
Example 1:
Input: [[1,3], [0,2], [1,3], [0,2]]
Output: true
Explanation: 
The graph looks like this:
0----1
|    |
|    |
3----2
We can divide the vertices into two groups: {0, 2} and {1, 3}.
Example 2:
Input: [[1,2,3], [0,2], [0,1,3], [0,2]]
Output: false
Explanation: 
The graph looks like this:
0----1
| \  |
|  \ |
3----2
We cannot find a way to divide the set of nodes into two independent subsets.
```

**Note:**

- `graph` will have length in range `[1, 100]`.
- `graph[i]` will contain integers in range `[0, graph.length - 1]`.
- `graph[i]` will not contain `i` or duplicate values.
- The graph is undirected: if any element `j` is in `graph[i]`, then `i` will be in `graph[j]`.

**Solution**

(1) DFS

Try to use two colors to color the graph and see if there are any adjacent nodes having the same color.

```java
class Solution {
    public boolean isBipartite(int[][] graph) {
        int n = graph.length;
        int[] colors = new int[n];			
        
        //This graph might be a disconnected graph. So check each unvisited node.
        for (int i = 0; i < n; i++) {              
            if (colors[i] == 0 && !validColor(graph, colors, 1, i)) {
                return false;
            }
        }
        return true;
    }
    
    public boolean validColor(int[][] graph, int[] colors, int color, int node) {
        if (colors[node] != 0) {
            return colors[node] == color;
        }       
        colors[node] = color;       
        for (int next : graph[node]) {
            if (!validColor(graph, colors, -color, next)) {
                return false;
            }
        }
        return true;
    }
}

```

Time complexity: $$O(V+E)$$

(2) BFS

```go
func isBipartite(graph [][]int) bool {
	// 0: not colored, 1: white, -1: black
	color := make([]int, len(graph), len(graph))
	queue := make([]int, 0)
	queue = append(queue, 0)

	for i := range graph {
		if color[i] == 0 {
			color[i] = 1
			queue := make([]int, 0)
			queue = append(queue, i)
			for len(queue) > 0 {
				cur := queue[0]
				queue = queue[1:]
				for _, adj := range graph[cur] {
					if color[adj] == 0 {
						// color the adjacent vertex different color
						color[adj] = -color[cur]
						queue = append(queue, adj)
					} else if color[cur] == color[adj] {
						// if two adjacent vertices have the same color,
						// then return false
						return false
					}
				}
			}
		}
	}
	return true
}
```

Time complexity: $$O(V+E)$$

#### 6. [Keys and Rooms](https://leetcode.com/problems/keys-and-rooms/)

There are `N` rooms and you start in room `0`.  Each room has a distinct number in `0, 1, 2, ..., N-1`, and each room may have some keys to access the next room. 

Formally, each room `i` has a list of keys `rooms[i]`, and each key `rooms[i][j]` is an integer in `[0, 1, ..., N-1]` where `N = rooms.length`.  A key `rooms[i][j] = v` opens the room with number `v`.

Initially, all the rooms start locked (except for room `0`). 

You can walk back and forth between rooms freely.

Return `true` if and only if you can enter every room.

**Example 1:**

```
Input: [[1],[2],[3],[]]
Output: true
Explanation:  
We start in room 0, and pick up key 1.
We then go to room 1, and pick up key 2.
We then go to room 2, and pick up key 3.
We then go to room 3.  Since we were able to go to every room, we return true.
```

**Example 2:**

```
Input: [[1,3],[3,0,1],[2],[0]]
Output: false
Explanation: We can't enter the room with number 2.
```

**Note:**

1. `1 <= rooms.length <= 1000`
2. `0 <= rooms[i].length <= 1000`
3. The number of keys in all rooms combined is at most `3000`.

**Solution**

This problem equals whether the graph is connected i.e. whether we can start from a vertex to visit the rest of vertices. 

(1) DFS

```java
class Solution {
    
    private Set<Integer> visitedRooms = new HashSet<>();
    
    public boolean canVisitAllRooms(List<List<Integer>> rooms) {
        dfs(0, rooms);
        return visitedRooms.size() == rooms.size();
    }
    
    private void dfs(int room, List<List<Integer>> rooms) {
        visitedRooms.add(room);
        List<Integer> keys = rooms.get(room);
        for (int k : keys) {
            // Be careful: the graph may contain circles.
            if (!visitedRooms.contains(k)) {
                dfs(k, rooms);
            }
        }
    }
}
```

 (2) BFS

```java
class Solution {
    
    private Set<Integer> visitedRooms = new HashSet<>();
    
    public boolean canVisitAllRooms(List<List<Integer>> rooms) {
        Queue<Integer> queue = new LinkedList<>();
        queue.offer(0);
        int room;
        while(!queue.isEmpty()) {
            room = queue.poll();
            visitedRooms.add(room);
            for (int k : rooms.get(room)) {
                if(!visitedRooms.contains(k)) {
                    queue.offer(k);
                }
            }
        }
        return visitedRooms.size() == rooms.size();
    }
}
```

#### 7. [Minimum Height Trees](https://leetcode.com/problems/minimum-height-trees)

For an undirected graph with tree characteristics, we can choose any node as the root. The result graph is then a rooted tree. Among all possible rooted trees, those with minimum height are called minimum height trees (MHTs). Given such a graph, write a function to find all the MHTs and return a list of their root labels.

**Format**
The graph contains `n` nodes which are labeled from `0` to `n - 1`. You will be given the number `n` and a list of undirected `edges`(each edge is a pair of labels).

You can assume that no duplicate edges will appear in `edges`. Since all edges are undirected, `[0, 1]` is the same as `[1, 0]`and thus will not appear together in `edges`.

**Example 1 :**

```
Input: n = 4, edges = [[1, 0], [1, 2], [1, 3]]

        0
        |
        1
       / \
      2   3 

Output: [1]
```

**Example 2 :**

```
Input: n = 6, edges = [[0, 3], [1, 3], [2, 3], [4, 3], [5, 4]]

     0  1  2
      \ | /
        3
        |
        4
        |
        5 

Output: [3, 4]
```

**Note**:

- According to the [definition of tree on Wikipedia](https://en.wikipedia.org/wiki/Tree_(graph_theory)): “a tree is an undirected graph in which any two vertices are connected by *exactly* one path. In other words, any connected graph without simple cycles is a tree.”
- The height of a rooted tree is the number of edges on the longest downward path between the root and a leaf.

**Solution**

It is easy to see that the root of an MHT has to be the middle point (or two middle points) of the longest path of the tree.

We start from every leaf node. We let the pointers move the same speed. When two pointers meet, we keep only one of them, until the last two pointers meet or one step away we then find the roots. It is easy to see that the last two pointers are from the two ends of the longest path in the graph.

The actual implementation is similar to the BFS topological sort. Remove the leaves, update the degrees of inner vertexes. Then remove the new leaves. Doing so until there are 2 or 1 nodes left.

```go
func findMinHeightTrees(n int, edges [][]int) []int {
    if n == 1 {
        return []int{0}
    }
    
    // adj[i]: adjacent vertices of i
    adj := make([][]int, n, n)
    for _, edge := range edges {
        adj[edge[0]] = append(adj[edge[0]], edge[1])
        adj[edge[1]] = append(adj[edge[1]], edge[0])
    }
    
    // leaves: current leaf nodes
    leaves := make([]int, 0)
    for v, e:= range adj {
        if len(e) == 1 {
            leaves = append(leaves, v)
        }
    }
    
    for n > 2 {
        n -= len(leaves)
        newLeaves := make([]int, 0)
        for _, l := range leaves {
            v := adj[l][0]
            // remove leaf nodes and remove them from adj too
            for i, j := range adj[v] {
                if j == l {
                    adj[v] = append(adj[v][:i], adj[v][i+1:]...)
                    break
                }
            }
            // update current leaf nodes
            if len(adj[v]) == 1 {
                newLeaves = append(newLeaves, v)
            }
        }
        leaves = newLeaves
    }
    return leaves
}
```

#### 8. [Redundant Connection](https://leetcode.com/problems/redundant-connection/)

In this problem, a tree is an **undirected** graph that is connected and has no cycles.

The given input is a graph that started as a tree with N nodes (with distinct values 1, 2, ..., N), with one additional edge added. The added edge has two different vertices chosen from 1 to N, and was not an edge that already existed.

The resulting graph is given as a 2D-array of `edges`. Each element of `edges` is a pair `[u, v]` with `u < v`, that represents an **undirected** edge connecting nodes `u` and `v`.

Return an edge that can be removed so that the resulting graph is a tree of N nodes. If there are multiple answers, return the answer that occurs last in the given 2D-array. The answer edge `[u, v]` should be in the same format, with `u < v`.

**Example 1:**

```
Input: [[1,2], [1,3], [2,3]]
Output: [2,3]
Explanation: The given undirected graph will be like this:
  1
 / \
2 - 3
```

**Example 2:**

```
Input: [[1,2], [2,3], [3,4], [1,4], [1,5]]
Output: [1,4]
Explanation: The given undirected graph will be like this:
5 - 1 - 2
    |   |
    4 - 3
```

**Note:**

The size of the input 2D-array will be between 3 and 1000.

Every integer represented in the 2D-array will be between 1 and N, where N is the size of the input array.

**Solution**

(1) Union Find

We can make use of [Disjoint Sets (Union Find)](https://en.wikipedia.org/wiki/Disjoint-set_data_structure).
If we regard a node as an element, a connected component is actually a disjoint set.

For example,

```
Given edges [1, 2], [1, 3], [2, 3],
  1
 / \
2 - 3
```

Initially, there are 3 disjoint sets: 1, 2, 3.
Edge [1,2] connects 1 to 2, i.e., 1 and 2 are winthin the same connected component.
Edge [1,3] connects 1 to 3, i.e., 1 and 3 are winthin the same connected component.
Edge [2,3] connects 2 to 3, but 2 and 3 have been within the same connected component already, so [2, 3] is redundant.

```go
func findRedundantConnection(edges [][]int) []int {
	u := newUnionFind(len(edges))
	for _, e := range edges {
		if !u.union(e[0], e[1]) {
			return e
		}
	}
	return nil
}

// Union Find
type UnionFind struct {
	Parent []int
	Rank   []int  
}

func newUnionFind(n int) *UnionFind {
	u := &UnionFind{make([]int, n+1), make([]int, n+1)}
	for i := 1; i < n+1; i++ {
		u.Parent[i] = i
		u.Rank[i] = 1
	}
	return u
}

func (u *UnionFind) find(x int) int {
	if u.Parent[x] == x {
		return x
	}
	u.Parent[x] = u.find(u.Parent[x])
	return u.Parent[x]
}

func (u *UnionFind) union(x, y int) bool {
	rootX := u.find(x)
	rootY := u.find(y)
	if rootX == rootY {
		return false
	}
	if u.Rank[rootX] < u.Rank[rootY] {
		u.Parent[rootX] = rootY
		u.Rank[rootY] += u.Rank[rootX]
	} else {
		u.Parent[rootY] = rootX
		u.Rank[rootX] += u.Rank[rootY]
	}
	return true
}
```

Time complexity: $$O(N\alpha(N)) \approx O(N)$$, where $$N$$ is the number of vertices (and also the number of edges) in the graph, and $$\alpha$$ is the *Inverse-Ackermann* function. We make up to $$N$$ queries of `union`, which takes (amortized) $$O(\alpha(N))$$ time.

(2) DFS

For each edge `(u, v)`, traverse the graph with a depth-first search to see if we can connect `u` to `v`. If we can, then it must be the duplicate edge.

```go
func findRedundantConnection(edges [][]int) []int {
	MAX_EDGE := 1000
	graph := make([][]int, MAX_EDGE+1)
	for i := range graph {
		graph[i] = make([]int, 0)
	}
	for _, e := range edges {
		visited := make(map[int]bool)
		if len(graph[e[0]]) > 0 && len(graph[e[1]]) > 0 && dfs(graph, e[0], e[1], visited) {
			return e
		}
		graph[e[0]] = append(graph[e[0]], e[1])
		graph[e[1]] = append(graph[e[1]], e[0])
	}
	return nil
}

func dfs(graph [][]int, u int, v int, visited map[int]bool) bool {
	if !visited[u] {
		visited[u] = true
		if u == v {
			return true
		}
		for _, adj := range graph[u] {
			if dfs(graph, adj, v, visited) {
				return true
			}
		}
	}
	return false
}
```

Time Complexity: $$O(N^2)$$ where $$N$$ is the number of vertices (and also the number of edges) in the graph. In the worst case, for every edge we include, we have to search every previously-occurring edge of the graph.

#### 9. [Reconstruct Itinerary](https://leetcode.com/problems/reconstruct-itinerary/)

Given a list of airline tickets represented by pairs of departure and arrival airports `[from, to]`, reconstruct the itinerary in order. All of the tickets belong to a man who departs from `JFK`. Thus, the itinerary must begin with `JFK`.

**Note:**

1. If there are multiple valid itineraries, you should return the itinerary that has the smallest lexical order when read as a single string. For example, the itinerary `["JFK", "LGA"]` has a smaller lexical order than `["JFK", "LGB"]`.
2. All airports are represented by three capital letters (IATA code).
3. You may assume all tickets form at least one valid itinerary.

**Example 1:**

```
Input: [["MUC", "LHR"], ["JFK", "MUC"], ["SFO", "SJC"], ["LHR", "SFO"]]
Output: ["JFK", "MUC", "LHR", "SFO", "SJC"]
```

**Example 2:**

```
Input: [["JFK","SFO"],["JFK","ATL"],["SFO","ATL"],["ATL","JFK"],["ATL","SFO"]]
Output: ["JFK","ATL","JFK","SFO","ATL","SFO"]
Explanation: Another possible reconstruction is ["JFK","SFO","ATL","JFK","ATL","SFO"].
             But it is larger in lexical order.
```

**Solution**

This problem is equal to finding a _Eulerian cycle_. An **Eulerian cycle**,[[3\]](https://en.wikipedia.org/wiki/Eulerian_path#cite_note-pathcycle-3) **Eulerian circuit** or **Euler tour** in an undirected graph is a [cycle](https://en.wikipedia.org/wiki/Cycle_(graph_theory)) that uses each edge exactly once. What we need to do is finding a Eulerian cycle according to lexical order. 

First keep going forward until you get stuck. That's a good main path already. Remaining tickets form cycles which are found on the way back and get merged into that main path. By writing down the path backwards when retreating from recursion, merging the cycles into the main path is easy - the end part of the path has already been written, the start part of the path hasn't been written yet, so just write down the cycle now and then keep backwards-writing the path.

Example:

![enter image description here](http://www.stefan-pochmann.info/misc/reconstruct-itinerary.png)

From JFK we first visit JFK -> A -> C -> D -> A. There we're stuck, so we write down A as the end of the route and retreat back to D. There we see the unused ticket to B and follow it: D -> B -> C -> JFK -> D. Then we're stuck again, retreat and write down the airports while doing so: Write down D before the already written A, then JFK before the D, etc. When we're back from our cycle at D, the written route is D -> B -> C -> JFK -> D -> A. Then we retreat further along the original path, prepending C, A and finally JFK to the route, ending up with the route JFK -> A -> C -> D -> B -> C -> JFK -> D -> A.

```kotlin
class Solution {
    // Use a priority queue to maintain the lexical order
    private var targets = mutableMapOf<String, java.util.PriorityQueue<String>>()

    private var route = mutableListOf<String>()

    fun findItinerary(tickets: Array<Array<String>>): List<String> {
        tickets.forEach { it ->
            targets.computeIfAbsent(it[0]) {java.util.PriorityQueue() }.add(it[1])
        }
        visit("JFK")
        return route
    }

    private fun visit(v: String) {
        while (targets.containsKey(v) && targets[v]!!.isNotEmpty()) {
            visit(targets[v]!!.poll())
        }
        route.add(0, v)
    }
}
```

Time complexity: $$O(ElogE)$$, where E is the number of edges.

#### 10. [Couples Holding Hands](https://leetcode.com/problems/couples-holding-hands/)

N couples sit in 2N seats arranged in a row and want to hold hands. We want to know the minimum number of swaps so that every couple is sitting side by side. A *swap* consists of choosing **any** two people, then they stand up and switch seats.

The people and seats are represented by an integer from `0` to `2N-1`, the couples are numbered in order, the first couple being `(0, 1)`, the second couple being `(2, 3)`, and so on with the last couple being `(2N-2, 2N-1)`.

The couples' initial seating is given by `row[i]` being the value of the person who is initially sitting in the i-th seat.

**Example 1:**

```
Input: row = [0, 2, 1, 3]
Output: 1
Explanation: We only need to swap the second (row[1]) and third (row[2]) person.
```

**Example 2:**

```
Input: row = [3, 2, 0, 1]
Output: 0
Explanation: All couples are already seated side by side.
```

**Note:**

1. `len(row)` is even and in the range of `[4, 60]`.
2. `row` is guaranteed to be a permutation of `0...len(row)-1`.

**Solution**

First we need to dive into N integers problem and then we can generalize it to N couples problem.

(1) N integers problem 

Assume we have an integer array `row` of length `N`, which contains integers from `0` up to `N-1` but in random order. You are free to choose any two numbers and swap them. What is the minimum number of swaps needed so that we have `i == row[i]` for `0 <= i < N` (or equivalently, to sort this integer array)?

**First**, to apply the cyclic swapping algorithm, we need to divide the `N` indices into mutually exclusive index groups, where indices in each group form a cycle: `i0 --> i1 --> ... --> ik --> i0`. Here we employ the notation `i --> j` to indicate that we expect the element at index `i` to appear at index `j` at the end of the swapping process.

Before we dive into the procedure for building the index groups, here is a simple example to illustrate the ideas, assuming we have the following integer array and corresponding indices:

```
row: 2, 3, 1, 0, 5, 4
idx: 0, 1, 2, 3, 4, 5
```

Starting from index `0`, what is the index that the element at index `0` is expected to appear? The answer is `row[0]`, which is `2`. Using the above notation, we have `0 --> 2`. Then starting from index `2`, what is the index that the element at index `2` is expected to appear? The answer will be `row[2]`, which is `1`, so we have `2 --> 1`. We can continue in this fashion until the indices form a cycle, which indicates an index group has been found: `0 --> 2 --> 1 --> 3 --> 0`. Then we choose another start index that has not been visited and repeat what we just did. This time we found another group: `4 --> 5 --> 4`, after which all indices are visited so we conclude there are two index groups.

Now for an arbitrary integer array, we can take similar approaches to build the index groups. Starting from some unvisited index `i0`, we compute the index `i1` at which the element at `i0` is expected to appear. In this case, `i1 = row[i0]`. We then continue from index `i1` and compute the index `i2` at which the element at `i1` is expected to appear. Similarly we have, `i2 = row[i1]`. We continue in this fashion to construct a list of indices: `i0 --> i1 --> i2 --> ... --> ik`. Next we will show that eventually the list will repeat itself from index `i0`, which has two implications:

1. Eventually the list will repeat itself.
2. It will repeat itself from index `i0`, not other indices.

**Next** suppose we have produced two such index groups, `g1` and `g2`, which are not identical (there exists at least one index contained in `g1` but not in `g2` and at least one index contained in `g2` but not in `g1`). We will show that all indices in `g1` cannot appear in `g2`, and vice versa - - `g1` and `g2` are mutually exclusive. The proof is straightforward: if there is some index `j` that is common to both `g1` and `g2`, then both `g1` and `g2` can be constructed by starting from index `j` and following the aforementioned procedure. Since each index is only dependent on its predecessor, the groups generated from the same start index will be identical to each other, contradicting the assumption. Therefore `g1` and `g2` will be mutually exclusive. This also implies the union of all groups will cover all the `N` indices exactly once.

**Lastly**, we will show that the minimum number of swaps needed to resolve an index group of size `k` is given by `k - 1`. Here we define the size of a group as the number of distinct indices contained in the group, for example:

1. Size **1** groups: `0 --> 0`, `2 --> 2`, etc.
2. Size **2** groups: `0 --> 3 --> 0`, `2 --> 1 --> 2`, etc.
   ......
3. Size **k** groups: `0 --> 1 --> 2 --> ... --> (k-1) --> 0`, etc.

And by saying "resolving a group", we mean placing the elements at each index contained in the group to their expected positions at the end of the swapping process. In this case, we want to put the element at index `i`, which is `row[i]`, to its expected position, which is `row[i]` again (the fact that the element itself coincides with its expected position is a result of the placement requirement `row[i] == i`).

**In conclusion**, the minimum number of swaps needed to resolve the whole array can be obtained by summing up the minimum number of swaps needed to resolve each of the index groups. To resolve each index group, we are free to choose any two distinct indices in the group and swap them so as to reduce the group to two smaller disjoint groups. In practice, we can always choose a pivot index and continuously swap it with its expected index until the pivot index is the same as its expected index, meaning the entire group is resolved and all placement requirements within the group are satisfied.

```java
public int miniSwapsArray(int[] row) {
    int res = 0, N = row.length;

    for (int i = 0; i < N; i++) {
		for (int j = row[i]; i != j; j = row[i]) {
			swap(row, i, j);
			res++;
		}
    }

    return res;
}

private void swap(int[] arr, int i, int j) {
    int t = arr[i];
    arr[i] = arr[j];
    arr[j] = t;
}
```

(2) N couples problem 

The `N` couples problem can be solved using exactly the same idea as the `N` integers problem, except now we have different placement requirements: instead of `i == row[i]`, we require `i == ptn[pos[ptn[row[i]]]]`, where we have defined two additional arrays `ptn` and `pos`:

1. `ptn[i]` denotes the partner of label `i` (`i` can be either a seat or a person) - - `ptn[i] = i + 1` if `i` is even; `ptn[i] = i - 1` if `i` is odd.

   

2. `pos[i]` denotes the index of the person with label `i` in the `row` array - - `row[pos[i]] == i`.

The meaning of `i == ptn[pos[ptn[row[i]]]]` is as follows:

1. The person sitting at seat `i` has a label `row[i]`, and we want to place him/her next to his/her partner.

   

2. So we first find the label of his/her partner, which is given by `ptn[row[i]]`.

   

3. We then find the seat of his/her partner, which is given by `pos[ptn[row[i]]]`.

   

4. Lastly we find the seat next to his/her partner's seat, which is given by `ptn[pos[ptn[row[i]]]]`.

Therefore, for each pivot index `i`, its expected index `j` is given by `ptn[pos[ptn[row[i]]]]`. As long as `i != j`, we swap the two elements at index `i` and `j`, and continue until the placement requirement is satisfied. A minor complication here is that for each swapping operation, we need to swap both the `row` and `pos` arrays.

Note that there are several optimizations we can do, just to name a few:

1. The `ptn` array can be replaced with a simple function that takes an index `i` and returns `i + 1` or `i - 1` depending on whether `i` is even or odd.

   

2. We can check every other seat instead of all seats. This is because we are matching each person to his/her partners, so technically speaking there are always half of the people sitting at the right seats.

   

3. There is an alternative way for building the index groups which goes in backward direction, that is instead of building the cycle like `i0 --> i1 --> ... --> jk --> i0`, we can also build it like `i0 <-- i1 <-- ... <-- ik <-- i0`, where `i <-- j` means the element at index `j` is expected to appear at index `i`. In this case, the pivot index will be changing along the cycle as the swapping operations are applied. The benefit is that we only need to do swapping on the `row` array.

```go
func minSwapsCouples(row []int) int {
    count, N := 0, len(row)
    ptn, pos := make([]int, N), make([]int, N)
    
    for i := range row {
        if i % 2 == 0 {
            ptn[i] = i + 1
        } else {
            ptn[i] = i - 1
        }
        pos[row[i]] = i
    }
    
    for i := range row {
        for j := ptn[pos[ptn[row[i]]]]; i != j; j = ptn[pos[ptn[row[i]]]] {
            row[i], row[j] = row[j], row[i]
            pos[row[i]], pos[row[j]] = pos[row[j]], pos[row[i]]
            count++
        }
    }
    
    return count 
}
```

Time complexity: $$O(n)$$

#### 11. [K-Similar Strings](https://leetcode.com/problems/k-similar-strings/)

Strings `A` and `B` are `K`-similar (for some non-negative integer `K`) if we can swap the positions of two letters in `A` exactly `K` times so that the resulting string equals `B`.

Given two anagrams `A` and `B`, return the smallest `K` for which `A` and `B` are `K`-similar.

**Example 1:**

```
Input: A = "ab", B = "ba"
Output: 1
```

**Example 2:**

```
Input: A = "abc", B = "bca"
Output: 2
```

**Example 3:**

```
Input: A = "abac", B = "baca"
Output: 2
```

**Example 4:**

```
Input: A = "aabc", B = "abca"
Output: 2
```

**Note:**

1. `1 <= A.length == B.length <= 20`
2. `A` and `B` contain only lowercase letters from the set `{'a', 'b', 'c', 'd', 'e', 'f'}`

**Solution**

**When it comes to shortest step, you should keep BFS in mind**. The trick in this problem is that you should just swap the first incorrect pair in each level, instead of trying each pair, which causes TLE, because the other swap is not important. 

```go
func kSimilarity(A string, B string) int {
    if A == B {
        return 0
    }
    
    visited := make(map[string]bool)
    queue := make([]string, 0)
    visited[A] = true
    queue = append(queue, A)
    res := 0
    for len(queue) > 0 {
        res++
        for sz := len(queue); sz > 0; sz-- {
            s := queue[0]
            queue = queue[1:]
            i := 0
            // find the first position where we need to swap characters 
            for s[i] == B[i] {
                i++
            }
            // BFS
            for j := i + 1; j < len(s); j++ {
                // find the character s[j] which should replace s[i]
                // if s[j] == B[j], which means the j-th character is correct, 
                //the swapping should not be taken
                // if s[i] != B[j], which means i-th character is not corresponding to 
                //j-th character, the swapping should not be taken either
                if s[j] == B[j] || s[i] != B[j] {
                    continue
                }
                temp := swap(s, i, j)
                if temp == B {
                    return res
                }
                if !visited[temp] {
                    visited[temp] = true
                    queue = append(queue, temp)
                }
            }
        }
    }
    return res
}

func swap(s string, i, j int) string {
    chars := make([]rune, 0)
    for _, c := range s { 
        chars = append(chars, c) 
    }
    chars[i], chars[j] = chars[j], chars[i]
    return string(chars)
}
```

#### 12. [Similar String Groups](https://leetcode.com/problems/similar-string-groups/)

Two strings `X` and `Y` are similar if we can swap two letters (in different positions) of `X`, so that it equals `Y`.

For example, `"tars"` and `"rats"` are similar (swapping at positions `0` and `2`), and `"rats"` and `"arts"` are similar, but `"star"` is not similar to `"tars"`, `"rats"`, or `"arts"`.

Together, these form two connected groups by similarity: `{"tars", "rats", "arts"}` and `{"star"}`.  Notice that `"tars"` and `"arts"` are in the same group even though they are not similar.  Formally, each group is such that a word is in the group if and only if it is similar to at least one other word in the group.

We are given a list `A` of strings.  Every string in `A` is an anagram of every other string in `A`.  How many groups are there?

**Example 1:**

```
Input: ["tars","rats","arts","star"]
Output: 2
```

**Note:**

1. `A.length <= 2000`
2. `A[i].length <= 1000`
3. `A.length * A[i].length <= 20000`
4. All words in `A` consist of lowercase letters only.
5. All words in `A` have the same length and are anagrams of each other.
6. The judging time limit has been increased for this question.

**Solution**

(1) DFS

```go
func numSimilarGroups(A []string) int {
    if len(A) < 2 {
        return len(A)
    }
    
    res := 0
    for i := range A {
        // avoid testing a string more than once 
        if A[i] == "" {
            continue
        }
        str := A[i]
        A[i] = ""
        res++
        dfs(A, str)
    }
    return res
}

func dfs(A []string, s string) {
    for i := range A {
        if A[i] == "" {
            continue
        }
        if isSimilar(A[i], s) {
            str := A[i]
            A[i] = ""
            dfs(A, str)
        }
    }
}

func isSimilar(s, t string) bool {
    swap := 0
    for i := range s {
        if swap > 2 {
            return false 
        }
        if s[i] != t[i] {
            swap++
        }
    }
    return swap == 2 || swap == 0
}
```

Time complexity: $$O(kn^2)$$, n is the length of array and k is the length of string. 

(2) Disjoint Set

```kotlin
class Solution {
    fun numSimilarGroups(A: Array<String>): Int {
        val set = DisjointSet(A.size)
        for (i in A.indices) {
            for (j in i + 1 until A.size) {
                if (isSimilar(A[i], A[j])) {
                    set.join(i, j)
                }
            }
        }
        return set.size()
    }

    private fun isSimilar(s: String, t: String): Boolean {
        var swap = 0
        for (i in 0 until s.length) {
            if (s[i] != t[i]) {
                swap++
                if (swap > 2) {
                    return false
                }
            }
        }
        return swap == 2
    }
}

class DisjointSet(n: Int) {
    private val elements = mutableListOf<Int>()

    private var size = 0

    init {
        elements.addAll(0 until n)
        size = n
    }

    private fun find(i: Int): Int {
        if (i != elements[i]) {
            elements[i] = find(elements[i])
        }
        return elements[i]
    }

    fun join(i: Int, j: Int): Unit {
        val ri = find(i)
        val rj = find(j)
        if (ri != rj) {
            elements[ri] = rj
            size--
        }
    }

    fun size(): Int {
        return size
    }
}
```

