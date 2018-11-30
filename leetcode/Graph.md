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

