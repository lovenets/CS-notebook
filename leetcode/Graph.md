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
	// the in degree of vertices,which also means
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

