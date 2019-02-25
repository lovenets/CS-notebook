### 1.[Two Sum](https://leetcode.com/problems/two-sum/)

Given an array of integers, return **indices** of the two numbers such that they add up to a specific target.

You may assume that each input would have **exactly** one solution, and you may not use the *same* element twice.

**Example:**

Given `nums = [2, 7, 11, 15]`, `target = 9`,Because `nums[0] + nums[1] = 2 + 7 = 9`,return `[0, 1]`.

**My Solution**

Time complexity: O(n^2^)

Space complexity: O(1)

```java
// brute force
class Solution {
    public int[] twoSum(int[] nums, int target) {
        int[] solution = new int[2];
        
        for(int i = 0; i < nums.length - 1; i++){
            for(int j = i + 1; j <= nums.length - 1; j++){
                if(nums[i] + nums[j] == target){
                    solution[0] = i;
                    solution[1] = j;
                    break;
                }
            }
        }  
        
        return solution;
    }
}
```

**Optimization**

Time complexity: O(n)

Space complexity: O(n)

```go
func twoSum(nums []int, target int) []int {
    amap := make(map[int]int)
    
    var solution []int
    
    // while we iterate and insert elements into map,we look back 
    // to check if current element's complement already exists in the map
    for index1,num := range nums {
        complement := target - num
        if index2,present := amap[complement]; present {
            solution = append(solution,index1)
            solution = append(solution,index2)
        }
        amap[num] = index1
    }
    
    return solution
}
```

### 2.[Jewels and Stones](https://leetcode.com/problems/jewels-and-stones/)

You're given strings `J` representing the types of stones that are jewels, and `S` representing the stones you have.  Each character in `S` is a type of stone you have.  You want to know how many of the stones you have are also jewels.

The letters in `J` are guaranteed distinct, and all characters in `J` and `S` are letters. Letters are case sensitive, so `"a"` is considered a different type of stone from `"A"`.

**Example:**

Input: `J = "aA", S = "aAAbbbb"`
Output: 3

**My solution**:

```go
func numJewelsInStones(J string, S string) int {
    // calculate total types of jewels
    var types = make(map[uint8]int)
	for i := 0; i < len(J); i++ {
		types[J[i]] = 0
	}

    // calculate how many jewels I have
	for i := 0; i < len(S); i++ {
		if _,found := types[S[i]]; found  {
			types[S[i]] += 1
		}
	}
	count := 0
	for _,v := range types  {
		if v != 0 {
			count += v
		}
	}
	return count
}
```

**Improvement:**

```go
func numJewelsInStones(J string, S string) int {
    count := 0
    for i := 0; i < len(S); i++ {
        if strings.Contains(J,string(S[i])) {
            count++
        }
    }
    return count
}
```

```go
func numJewelsInStones(J string, S string) int {
    sum := 0
    for _, v := range J {
        sum += strings.Count(S, string(v))
    }
    return sum
}
```

**Trick**:

```java
public int numJewelsInStones(String J, String S) {
    // with the help of regular expression
    // replace all Stone characters with empty string 
    return S.replaceAll("[^" + J + "]", "").length();
}
```

### 3.[Sort Array By Parity](https://leetcode.com/problems/sort-array-by-parity/)

Given an array `A` of non-negative integers, return an array consisting of all the even elements of `A`, followed by all the odd elements of `A`.

You may return any answer array that satisfies this condition

**Example:**

```
Input: [3,1,2,4]
Output: [2,4,3,1]
```

**My Solution**

Go:

```go
func sortArrayByParity(A []int) []int {
    odds := make([]int,0)
    evens := make([]int,0)
    for _,v := range A {
        if (v % 2 == 0){
            evens = append(evens,v)
        } else {
            odds = append(odds,v)
        }
    }
    return append(evens,odds...)
}
```

Time complexity: O(n), n is the length of array.

Scala:

```scala
object Solution {
    def sortArrayByParity(A: Array[Int]): Array[Int] = {
        A.filter(_ % 2 == 0) ++ A.filter(_ % 2 != 0)
    }
}
```

**Improvement**

1.sort

```java
// custom sort
class Solution {
    public int[] sortArrayByParity(int[] A) {
        Integer[] B = new Integer[A.length];
        for (int t = 0; t < A.length; ++t)
            B[t] = A[t];

        Arrays.sort(B, (a, b) -> Integer.compare(a%2, b%2));

        for (int t = 0; t < A.length; ++t)
            A[t] = B[t];
        return A;

        /* Alternative:
        return Arrays.stream(A)
                     .boxed()
                     .sorted((a, b) -> Integer.compare(a%2, b%2))
                     .mapToInt(i -> i)
                     .toArray();
        */
    }
}
```

Time complexity: O(nlogn),n is the length of array

2.In-Place

```java
// just like quick sort
// everything below i is even and everything above j is odd
class Solution {
    public int[] sortArrayByParity(int[] A) {
        int i = 0, j = A.length - 1;
        while (i < j) {
            if (A[i] % 2 > A[j] % 2) {
                int tmp = A[i];
                A[i] = A[j];
                A[j] = tmp;
            }

            if (A[i] % 2 == 0) i++;
            if (A[j] % 2 == 1) j--;
        }

        return A;
    }
}
```

Time complexity: O(n)

### 4.[Flipping an Image](https://leetcode.com/problems/flipping-an-image/)

Given a binary matrix `A`, we want to flip the image horizontally, then invert it, and return the resulting image.

To flip an image horizontally means that each row of the image is reversed.  For example, flipping `[1, 1, 0]` horizontally results in `[0, 1, 1]`.

To invert an image means that each `0` is replaced by `1`, and each `1` is replaced by `0`. For example, inverting `[0, 1, 1]` results in `[1, 0, 0]`.

**Example:**

```
Input: [[1,1,0],[1,0,1],[0,0,0]]
Output: [[1,0,0],[0,1,0],[1,1,1]]
Explanation: First reverse each row: [[0,1,1],[1,0,1],[0,0,0]].
Then, invert the image: [[1,0,0],[0,1,0],[1,1,1]]
```

**My Solution**

```go
func flipAndInvertImage(A [][]int) [][]int {
    rows := len(A[0])
    cols := len(A)
    
    // flip
    for i := 0; i < rows; i++ {
        for j := 0; j < cols / 2; j++ {
            temp := A[i][j]
            A[i][j] = A[i][cols - j - 1]
            A[i][cols - j - 1] = temp
        } 
    }
    
    // invert
    for i := 0; i < rows; i++ {
        for j := 0; j < cols; j++ {
            if A[i][j] == 0 {
                A[i][j] = 1
            } else {
                A[i][j] = 0
            }
        } 
    }
    
    return A
}
```

**Improvement**

Go:

```java
func flipAndInvertImage(A [][]int) [][]int {
    for i := 0; i < len(A); i++ {
        row := A[i]
        last := len(row) - 1;
        for j := 0; j <= last / 2; j ++ {
            // flip and invert at then same time
            // 1 ^ 1 == 0
            // 0 ^ 1 == 1
            row[j], row[last - j] = row[last - j] ^ 1, row[j] ^ 1
        }
    }
    return A
}
```

Java:

```java
// reverse a row and then toggle every value
// why not just toggle only symmetric equal values in a row
class Solution {
    public int[][] flipAndInvertImage(int[][] A) {
        int n = A.length;
        for (int[] row : A) {
            // compare row[i] and row[n - i - 1] in a row
            // if same, only invert both. Otherwise do nothing
            for (int i = 0; i * 2 < n; i++) {
                if (row[i] == row[n - i - 1]) {
                    row[i] = row[n - i - 1] ^= 1;  
                }
            }
        }
        return A;
    }
}
```

Time complexity: O(n^2^)

### 5. [Find All Duplicates Array](https://leetcode.com/problems/find-all-duplicates-in-an-array)

Given an array of integers, 1 ≤ a[i] ≤ *n* (*n* = size of array), some elements appear **twice** and others appear **once**.

Find all the elements that appear **twice** in this array.

Could you do it without extra space and in O(*n*) runtime?

**Example:**

```
Input:
[4,3,2,7,8,2,3,1]

Output:
[2,3]
```

**My Solution**

```go
// Time complexity is O(n) but need more space
func findDuplicates(nums []int) []int {
    result := make([]int,0)
	
	count := make(map[int]int)
	for _,num := range nums {
		if _,ok := count[num]; !ok {
			count[num] = 1
		} else {
            // this number has appeard twice
			result = append(result,num)
		}
	}
	return result
}
```

**Improvement**

```go
// Time complexity is O(n)
func findDuplicates(nums []int) []int {
    counts := make([]int, len(nums) + 1)
    ret := []int{}
    // since a[i] is not greater than n 
    // so you can do this
    for _, v := range nums {
        counts[v]++
    }
    for i, v := range counts {
        if v > 1 {
            ret = append(ret, i)
        }
    }
    return ret
}
```

A fantastic solution in Java, using no extra space and running in O(n) time:

```java
public class Solution {
    // when find a number i, flip the number nums[i - 1] to negative. 
    // if the number at position nums[i - 1] is already negative, i is the number that occurs twice.
    // Of course the prerequisite of this solution is that a[i] <= n(n = size of array)
    
    public List<Integer> findDuplicates(int[] nums) {
        List<Integer> res = new ArrayList<>();
        for (int i = 0; i < nums.length; ++i) {
            int index = Math.abs(nums[i])-1;
            if (nums[index] < 0)
                res.add(Math.abs(nums[i]));
            nums[index] = -nums[index];
        }
        return res;
    }
}
```

### 6. [Max Area of  Island](https://leetcode.com/problems/max-area-of-island/)

Given a non-empty 2D array `grid` of 0's and 1's, an **island** is a group of `1`'s (representing land) connected 4-directionally (horizontal or vertical.) You may assume all four edges of the grid are surrounded by water.

Find the maximum area of an island in the given 2D array. (If there is no island, the maximum area is 0.)

**Example 1:**

```
[[0,0,1,0,0,0,0,1,0,0,0,0,0],
 [0,0,0,0,0,0,0,1,1,1,0,0,0],
 [0,1,1,0,1,0,0,0,0,0,0,0,0],
 [0,1,0,0,1,1,0,0,1,0,1,0,0],
 [0,1,0,0,1,1,0,0,1,1,1,0,0],
 [0,0,0,0,0,0,0,0,0,0,1,0,0],
 [0,0,0,0,0,0,0,1,1,1,0,0,0],
 [0,0,0,0,0,0,0,1,1,0,0,0,0]]
```

Given the above grid, return 

```
6
```

. Note the answer is not 11, because the island must be connected 4-directionally.

**My Solution**

Well, I learn from the standard solution...

```go
func maxAreaOfIsland(grid [][]int) int {
    rows := len(grid)
	cols := len(grid[0])
    // Why does Go make 2D array so inconvinient?
	visited := make([][]bool,rows)
	for col := range visited {
		visited[col] = make([]bool,cols)
	}

    // visit every cell once
	area := 0
	for r := 0; r < rows; r++ {
		for c := 0; c < cols; c++ {
			res := calArea(r, c, grid, visited)
			if area < res {
				area = res
			}
		}
	}

	return area
}

func calArea(r int, c int, grid [][]int, visited [][]bool) int {
	if r < 0 || r >= len(grid) || c < 0 || c >= len(grid[0]) || visited[r][c] || grid[r][c] == 0 {
		return 0
	}
	visited[r][c] = true
    // DFS
	return 1 + calArea(r+1, c, grid, visited) + calArea(r-1, c, grid, visited) +
		calArea(r, c+1, grid, visited) + calArea(r, c-1, grid, visited)
}
```

### 7. Container With Most Water

Given *n* non-negative integers *a1*, *a2*, ..., *an* , where each represents a point at coordinate (*i*, *ai*). *n* vertical lines are drawn such that the two endpoints of line *i* is at (*i*, *ai*) and (*i*, 0). Find two lines, which together with x-axis forms a container, such that the container contains the most water.

**Note:** You may not slant the container and *n* is at least 2.

![Container With Most Water](img\Container With Most Water.jpg)

**Example:**

```
Input: [1,8,6,2,5,4,8,3,7]
Output: 49
```

**My Solution**

```go
// brute-force
func maxArea(height []int) int {
	len := len(height)
	max, temp := 0, 0

	// try every possible combination
	for i := 0; i < len-1; i++ {
		for j := i + 1; j < len; j++ {
			if height[i] < height[j] {
				temp = height[i] * (j - i)
			} else {
				temp = height[j] * (j - i)
			}
			if temp > max {
				max = temp
			}
		}
	}

	return max
}
```

Time Complexity: O(n^2^)

**Improvement**

Initially we consider the area constituting the exterior most lines. Now, to maximize the area, we need to consider the area between the lines of larger lengths. If we try to move the pointer at the longer line inwards, we won't gain any increase in area, since it is limited by the shorter line. But moving the shorter line's pointer could turn out to be beneficial because you may find a longer line. This is done since a relatively longer line obtained by ==moving the shorter line's pointer might overcome the reduction in area caused by the width reduction==.

```go
// Two Pointer Approach
func maxArea(height []int) int {
	maxarea, l, r, temp := 0, 0, len(height)-1, 0

	// move the pointer which points to the shorter line
	// to the another end by one step
	for l < r {
		if height[l] < height[r] {
			temp = height[l] * (r - l)
			l++
		} else {
			temp = height[r] * (r - l)
			r--
		}
		if maxarea < temp {
			maxarea = temp
		}
	}

	return maxarea
}
```

Time Complexity: O(n)

### 8. [Word Search](https://leetcode.com/problems/word-search/)  

Given a 2D board and a word, find if the word exists in the grid.

The word should be constructed from letters of sequentially adjacent cell, where "adjacent" cells are those horizontally or vertically neighboring. The same letter cell may not be used more than once.

**Example:**

```
board =
[
  ['A','B','C','E'],
  ['S','F','C','S'],
  ['A','D','E','E']
]

Given word = "ABCCED", return true.
Given word = "SEE", return true.
Given word = "ABCB", return false.
```

**solution**

Obviously, this problem is related to DFS because **The word should be constructed from letters of sequentially adjacent cell**. Sounds like adjacent graph, right? 

```go
func exist(board [][]byte, word string) bool {
	if len(word) == 0 {
		return false
	}

	b := []byte(word)
	// x and y are index
	for x := range board {
		for y := range board[x] {
			if dfs(x, y, board, b) {
				return true
			}
		}
	}
	return false
}

func dfs(x, y int, board [][]byte, word []byte) bool {
	if x < 0 || x >= len(board) {
		return false
	}

	if y < 0 || y >= len(board[x]) {
		return false
	}

	if board[x][y] != word[0] {
		return false
	}

	if len(word) == 1 {
		return true
	}

	// mark the searched cell
	tmp := board[x][y]
	board[x][y] = '-'

    // After a successful round, found letters will be  
    // removed from word
	found := dfs(x+1, y, board, word[1:]) ||
		dfs(x-1, y, board, word[1:]) ||
		dfs(x, y+1, board, word[1:]) ||
		dfs(x, y-1, board, word[1:])

	// recover the searched cell for next round
	board[x][y] = tmp

	return found
}
```

In every round, in order to avoid repeat work, we need to mark searched cell. The native solution is to create a bool matrix but it will cost extra space. Since the content in every cell should be an English letter, we can change the contents of searched cells into '-', which is not an English letter. But when we finish a round, we should recover the contents of searched cells preparing for next round.

### 9. [First Missing Positive](https://leetcode.com/problems/first-missing-positive)

Given an unsorted integer array, find the smallest missing positive integer.

**Example 1:**

```
Input: [1,2,0]
Output: 3
```

**Example 2:**

```
Input: [3,4,-1,1]
Output: 2
```

**Example 3:**

```
Input: [7,8,9,11,12]
Output: 1
```

**Note:**

Your algorithm should run in *O*(*n*) time and uses constant extra space.

**Solution**

(1) $$O(n)$$ time but not $$O(1)$$ space

```go
func firstMissingPositive(nums []int) int {
    m := make(map[int]bool)
    for i := range nums {
        if nums[i] > 0 {
            m[nums[i]] = true
        }
    }
    min := 1
    for ; m[min]; min++ {}
    return min
}
```

(2) $$O(n)$$ time and $$O(1)$$ space

If we can put each number in order, we can find the first missing positive easily by traversing the array. For example, if we encounter 5, we put it in `nums[4]`. And if we find out that `5 != nums[4]`, 5 is the answer.

```c++
class Solution {
public:
    int firstMissingPositive(vector<int>& nums) {
        int size = nums.size();
        for (int i = 0; i < size; i++) {
            // while-loop is executed n times at most because 
            // because there are n numbers in total
            while (nums[i] > 0 && nums[i] <= size && nums[i] != nums[nums[i] - 1]) {
                swap(nums[i], nums[nums[i] - 1]);
            }
        }
        for (int i = 0; i < size; i++) {
            if (nums[i] != i + 1) {
                return i + 1;
            }
        }
        return size + 1;
    }
};
```



