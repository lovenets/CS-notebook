# Array

## [152. Maximum Product Subarray](<https://leetcode.com/problems/maximum-product-subarray/>)

Given an integer array `nums`, find the contiguous subarray within an array (containing at least one number) which has the largest product.

**Example 1:**

```
Input: [2,3,-2,4]
Output: 6
Explanation: [2,3] has the largest product 6.
```

**Example 2:**

```
Input: [-2,0,-1]
Output: 0
Explanation: The result cannot be 2, because [-2,-1] is not a subarray.
```

**Solution**

(1) Wrong Answer 

Stuck in local optimization so failed to find the global optimized result.

```go
func maxProduct(nums []int) int {
    if len(nums) == 0 {
        return 0
    }
    dp := make([]int, len(nums))
    dp[0] = nums[0]
    max := dp[0]
    for i := 1; i < len(nums); i++ {
        if tmp := dp[i-1]*nums[i]; tmp > nums[i] {
            dp[i] = tmp
        } else {
            dp[i] = nums[i]
        }
        if dp[i] > max {
            max = dp[i]
        }
    }
    return max
}
```

```
Input:
[-2,3,-4]
Output:
3
Expected:
24
```

(2) Accepted

Here we keep 2 values: the max cumulative product UP TO current element starting from SOMEWHERE in the past, and the minimum cumulative product UP TO current element. At each new element, we could either add the new element to the existing product, or start fresh the product from current index (wipe out previous results). If we see a negative number, the "candidate" for max should instead become the previous min product, because a bigger number multiplied by negative becomes smaller, hence the swap them.

```go
func maxProduct(nums []int) int {
    if len(nums) == 0 {
        return 0
    }
    max := nums[0]
    curMax, curMin := nums[0], nums[0]
    for i := 1; i < len(nums); i++ {
        if nums[i] < 0 {
            curMax, curMin = curMin, curMax
        }
        if tmp := curMax*nums[i]; tmp > nums[i] {
            curMax = tmp
        } else {
            curMax = nums[i]
        }
        if tmp := curMin*nums[i]; tmp < nums[i] {
            curMin = tmp
        } else {
            curMin = nums[i]
        }
        if max < curMax {
            max = curMax
        }
    }
    return max
}
```

It would be easier to see the DP structure if we store these 2 values for each index, like `maxProduct[i]`, `minProduct[i]` .

- Time complexity: $$O(n)$$
- Space complexity: $$O(1)$$

**Recap**

This is a typical DP problem. We get the final result from the results of sub problems.

## [169. Majority Element](<https://leetcode.com/problems/majority-element/>)

Given an array of size *n*, find the majority element. The majority element is the element that appears **more than** `⌊ n/2 ⌋`times.

You may assume that the array is non-empty and the majority element always exist in the array.

**Example 1:**

```
Input: [3,2,3]
Output: 3
```

**Example 2:**

```
Input: [2,2,1,1,1,2,2]
Output: 2
```

**Solution**

(1) Accepted

```go
func majorityElement(nums []int) int {
    n := len(nums)
    count := make(map[int]int)
    for i := range nums {
        if count[nums[i]]++; count[nums[i]] > n/2 {
            return nums[i]
        }
    }
    return 0
}
```

- Time complexity: $$O(n)$$
- Space complexity: $$O(m)$$ where m is the number of different numbers in the array.

(2) Accepted

```go
func majorityElement(nums []int) int {
    sort.Ints(nums)
    return nums[len(nums)/2]
}
```

- Time complexity: kind of $$O(nlogn)$$
- Space complexity: $$O(1)$$

(3) Accepted 

```go
func majorityElement(nums []int) int {
	var count, res int
	for i := range nums {
		if count == 0 {
			res = nums[i]
		}
		if nums[i] == res {
			count++
		} else {
			count--
		}
	}
	return res
}
```

The key point is "the majority element always exist in the array". 

When `count != 0` , it means `nums[1...i]` has a majority,which is major in the solution.
When `count == 0`, it means `nums[1...i ]` doesn't have a majority, so `nums[1...i]` will not help.

Actually, this is called [Boyer-Moore Majority Vote Algorithm]([https://en.wikipedia.org/wiki/Boyer%E2%80%93Moore_majority_vote_algorithm](https://en.wikipedia.org/wiki/Boyer–Moore_majority_vote_algorithm)).

<https://www.zhihu.com/question/49973163/answer/235921864>

- Time complexity: $$O(n)$$
- Space complexity: $$O(1)$$

## [189. Rotate Array](<https://leetcode.com/problems/rotate-array/>)

Given an array, rotate the array to the right by *k* steps, where *k* is non-negative.

**Example 1:**

```
Input: [1,2,3,4,5,6,7] and k = 3
Output: [5,6,7,1,2,3,4]
Explanation:
rotate 1 steps to the right: [7,1,2,3,4,5,6]
rotate 2 steps to the right: [6,7,1,2,3,4,5]
rotate 3 steps to the right: [5,6,7,1,2,3,4]
```

**Example 2:**

```
Input: [-1,-100,3,99] and k = 2
Output: [3,99,-1,-100]
Explanation: 
rotate 1 steps to the right: [99,-1,-100,3]
rotate 2 steps to the right: [3,99,-1,-100]
```

**Note:**

- Try to come up as many solutions as you can, there are at least 3 different ways to solve this problem.
- Could you do it in-place with O(1) extra space?

**Solution**

(1) Accepted

It's very straightforward.

```go
func rotate(nums []int, k int) {
	if len(nums) == 0 {
		return
	}
	for ; k > 0; k-- {
		last := nums[len(nums)-1]
		for i := len(nums) - 2; i >= 0; i-- {
			nums[i+1] = nums[i]
		}
		nums[0] = last
	}
}
```

- Time complexity: $$O(kn)$$
- Space complexity: $$O(1)$$

(2) Accepted

This approach is based on the fact that when we rotate the array k times, k%n*k* elements from the back end of the array come to the front and the rest of the elements from the front shift backwards.

In this approach, we firstly reverse all the elements of the array. Then, reversing the first k elements followed by reversing the rest n-k elements gives us the required result.

```go
k = 3
Original List                   : 1 2 3 4 5 6 7
After reversing all numbers     : 7 6 5 4 3 2 1
After reversing first k numbers : 5 6 7 4 3 2 1
After revering last n-k numbers : 5 6 7 1 2 3 4 --> Result
```

```go
func rotate(nums []int, k int) {
	if len(nums) == 0 {
		return
	}
	reverse := func(i, j int) {
		for ; i < j; i, j = i+1, j-1 {
			nums[i], nums[j] = nums[j], nums[i]
		}
	}
    k %= len(nums) // Handle the cases where k > len(nums)
	reverse(0, len(nums)-1)
	reverse(0, k-1)
	reverse(k, len(nums)-1)
}
```

- Time complexity: $$O(n)$$
- Space complexity: $$O(1)$$

(3) Accepted

We use an extra array in which we place every element of the array at its correct position.

```go
func rotate(nums []int, k int) {
	if len(nums) == 0 {
		return
	}
	n := len(nums)
	arr := make([]int, n)
	for i := range nums {
		arr[(i+k)%n] = nums[i]
	}
	for i  := range arr {
		nums[i] = arr[i]
	}
}
```

- Time complexity: $$O(n)$$
- Space complexity: $$O(n)$$

## [217. Contains Duplicates](<https://leetcode.com/problems/contains-duplicate/>)

Given an array of integers, find if the array contains any duplicates.

Your function should return true if any value appears at least twice in the array, and it should return false if every element is distinct.

**Example 1:**

```
Input: [1,2,3,1]
Output: true
```

**Example 2:**

```
Input: [1,2,3,4]
Output: false
```

**Solution**

(1) Accepted

```go
func containsDuplicate(nums []int) bool {
    if len(nums) == 0 {
        return false
    }
    sort.Ints(nums)
    for i := 0; i < len(nums)-1; i++ {
        if nums[i] == nums[i+1] {
            return true
        }
    }
    return false
}
```

- Time complexity: average $$O(nlogn)$$
- Space complexity: $$O(1)$$

(2) Accepted

```go
func containsDuplicate(nums []int) bool {
    if len(nums) == 0 {
        return false
    }
    count := make(map[int]int)
    for i := range nums {
        count[nums[i]]++
    }
    for n := range count {
        if count[n] > 1 {
            return true
        }
    }
    return false
}
```

- Time complexity: $$O(n)$$
- Space complexity: $$O(n)$$

(3) Accepted

```go
func containsDuplicate(nums []int) bool {
    if len(nums) <= 1 {
        return false
    }
    present := make(map[int]bool)
    for i := range nums {
        if present[nums[i]] {
            return true
        } else {
            present[nums[i]] = true
        }
    }
    return false
}
```

- Time complexity: $$O(n)$$
- Space complexity: $$O(n)$$

**Recap**

Always try as hard as possible to find out a solution which can solve array problem in one pass.

## [283. More Zeroes](<https://leetcode.com/problems/move-zeroes/>)

Given an array `nums`, write a function to move all `0`'s to the end of it while maintaining the relative order of the non-zero elements.

**Example:**

```
Input: [0,1,0,3,12]
Output: [1,3,12,0,0]
```

**Note**:

1. You must do this **in-place** without making a copy of the array.
2. Minimize the total number of operations.

**Solution**

(1) Accepted

Two pointers.

```go
func moveZeroes(nums []int) {
	for i, j := len(nums)-1, len(nums)-1; i >= 0 && j >= 0; {
		// Find a zero 
		for ; i >= 0 && nums[i] != 0; i-- {
		}
		if i < 0 {
			return
		} else {
			// Move non-zero elements to the left
			for k := i; k < j; k++ {
				nums[k] = nums[k+1]
			}
			// Move zero to the end
			nums[j] = 0
			i, j = i-1, j-1
		}
	}
}
```

- Time complexity: $$O(n)$$?
- Space complexity: $$O(1)$$

(2) Accepted

```go
func moveZeroes(nums []int) {
	if len(nums) == 0 {
		return
	}
	insert := 0
    // Move non-zeros to the left as far as possible
	for i := range nums {
		if nums[i] != 0 {
			nums[insert] = nums[i]
			insert++
		}
	}
    // Fill remaining positions with 0s
	for ; insert < len(nums); insert++ {
		nums[insert] = 0
	}
}
```

- Time complexity: $$O(n)$$
- Space complexity: $$O(1)$$

(3) Accepted

```go
func moveZeroes(nums []int) {
	if len(nums) == 0 {
		return
	}
    // j is the index of left-most zero 
	for i, j := 0, 0; i < len(nums); i++ {
		if nums[i] != 0 {
			nums[i], nums[j] = nums[j], nums[i]
			j++
		}
	}
}
```

Improvement:

```go
func moveZeroes(nums []int) {
	if len(nums) == 0 {
		return
	}
	for i, j := 0, 0; i < len(nums); i++ {
		if nums[i] != 0 {
             // Avoid unnecessary operations 
			if i > j {
				nums[j] = nums[i]
				nums[i] = 0
			}
			j++
		}
	}
}
```

- Time complexity: $$O(n)$$
- Space complexity: $$O(1)$$

**Recap**

Shifting elements in a array one by one is always too slow.

## [384. Shuffle Array](<https://leetcode.com/problems/shuffle-an-array/>)

Shuffle a set of numbers without duplicates.

**Example:**

```
// Init an array with set 1, 2, and 3.
int[] nums = {1,2,3};
Solution solution = new Solution(nums);

// Shuffle the array [1,2,3] and return its result. Any permutation of [1,2,3] must equally likely to be returned.
solution.shuffle();

// Resets the array back to its original configuration [1,2,3].
solution.reset();

// Returns the random shuffling of array [1,2,3].
solution.shuffle();
```

**Solution**

(1) Memory Limited Exceeded

```go
type Solution struct {
	Data  []int
	Perms [][]int
}

func Constructor(nums []int) Solution {
	solution := Solution{}
	solution.Data = make([]int, len(nums))
	copy(solution.Data, nums)
	// Generate all permutations
	solution.Perms = perm(nums)
	return solution
}

func perm(data []int) (res [][]int) {
	var do func(int)
	do = func(i int) {
		if i == len(data) {
			tmp := make([]int, i)
			copy(tmp, data)
			res = append(res, tmp)
		} else {
			for j := i; j < len(data); j++ {
				data[j], data[i] = data[i], data[j]
				do(i + 1)
				data[j], data[i] = data[i], data[j]
			}
		}
	}
	do(0)
	return
}

/** Resets the array to its original configuration and return it. */
func (this *Solution) Reset() []int {
	return this.Data
}

/** Returns a random shuffling of the array. */
func (this *Solution) Shuffle() []int {
	return this.Perms[rand.Intn(len(this.Perms))]
}
```

We will need $$O(n!)$$  extra space to store all permutations which is too much.

(2) Accepted

[Fisher-Yates Algorithm]([https://en.wikipedia.org/wiki/Fisher%E2%80%93Yates_shuffle#The_modern_algorithm])

```go
type Solution struct {
	Data []int
}

func Constructor(nums []int) Solution {
	return Solution{nums}
}

/** Resets the array to its original configuration and return it. */
func (this *Solution) Reset() []int {
	return this.Data
}

/** Returns a random shuffling of the array. */
func (this *Solution) Shuffle() []int {
	arr := make([]int, len(this.Data))
	copy(arr, this.Data)
	// Fisher-Yates algorithm
	for i := len(arr) - 1; i >= 1; i-- {
		j := rand.Intn(i + 1)
		arr[i], arr[j] = arr[j], arr[i]
	}
	return arr
}
```

`Shuffle`complexity:

- Time complexity: $$O(n)$$
- Space complexity: $$O(n)$$

Improvement: We allocate an extra array to store one permutation in advance instead of allocating one every time we call `Shuffle`.

```go
type Solution struct {
	Data []int
	Perm []int
}

func Constructor(nums []int) Solution {
	perm := make([]int, len(nums))
    copy(perm, nums)
	return Solution{nums, perm}
}

/** Resets the array to its original configuration and return it. */
func (this *Solution) Reset() []int {
	return this.Data
}

/** Returns a random shuffling of the array. */
func (this *Solution) Shuffle() []int {
	// Fisher-Yates algorithm
	for i := len(this.Perm) - 1; i >= 1; i-- {
		j := rand.Intn(i + 1)
		this.Perm[i], this.Perm[j] = this.Perm[j], this.Perm[i]
	}
	return this.Perm
}
```

(3) Accepted

`rand.Shuffle`is a function of standard library which can shuffle pseudo-randomizes the order of elements. Actually, it's also based on Fisher-Yates algorithm.

```go
type Solution struct {
	Data []int
	Perm []int
}

func Constructor(nums []int) Solution {
	perm := make([]int, len(nums))
	copy(perm, nums)
	return Solution{nums, perm}
}

/** Resets the array to its original configuration and return it. */
func (this *Solution) Reset() []int {
	return this.Data
}

/** Returns a random shuffling of the array. */
func (this *Solution) Shuffle() []int {
	rand.Shuffle(len(this.Perm), func(i, j int) {
		this.Perm[i], this.Perm[j] = this.Perm[j], this.Perm[i]
	})
	return this.Perm
}
```

**Recap**

1. Fisher-Yates algorithm can generate every permutation equally likely. `rand.Shuffle`is based on it.
2. `rand.Intn`generates pseudo-random number in [0, n).

## [350. Intersection of Two Arrays II](<https://leetcode.com/problems/intersection-of-two-arrays-ii/>)

Given two arrays, write a function to compute their intersection.

**Example 1:**

```
Input: nums1 = [1,2,2,1], nums2 = [2,2]
Output: [2,2]
```

**Example 2:**

```
Input: nums1 = [4,9,5], nums2 = [9,4,9,8,4]
Output: [4,9]
```

**Note:**

- Each element in the result should appear as many times as it shows in both arrays.
- The result can be in any order.

**Follow up:**

- What if the given array is already sorted? How would you optimize your algorithm?
- What if *nums1*'s size is small compared to *nums2*'s size? Which algorithm is better?
- What if elements of *nums2* are stored on disk, and the memory is limited such that you cannot load all elements into the memory at once?

**Solution**

(1) Accepted

Quite straightforward.

```go
func intersect(nums1 []int, nums2 []int) []int {
    if len(nums1) == 0 || len(nums2) == 0 {
        return nil
    }
    countNums1, countNums2 := make(map[int]int), make(map[int]int)
    for i := range nums1 {
        countNums1[nums1[i]]++
    }
    for i := range nums2 {
        countNums2[nums2[i]]++
    }
    res := make([]int, 0)
    add := func(element, count int) {
        for i := 0; i < count; i++ {
            res = append(res, element)
        }
    }
    for n1 := range countNums1 {
        var min int
        if countNums1[n1] < countNums2[n1] {
            min = countNums1[n1]
        } else {
            min = countNums2[n1]
        }
        add(n1, min)
    }
    return res
}
```

- Time complexity: $$O(max\{len(nums1), len(nums2)\})$$
- Space complexity: $$O(n1+n2)$$ where n1 is the number of distinct numbers in `nums1`and n2 is the number of distinct numbers in `nums2`.

Improvement: only one map

```go
func intersect(nums1 []int, nums2 []int) []int {
    if len(nums1) == 0 || len(nums2) == 0 {
        return nil
    }
    count := make(map[int]int)
    for i := range nums1 {
        count[nums1[i]]++
    }
    res := make([]int, 0)
    for i := range nums2 {
        if count[nums2[i]] > 0 {
            res = append(res, nums2[i])
            count[nums2[i]]--
        }
    }
    return res
}
```

(2) Accepted

```go
func intersect(nums1 []int, nums2 []int) []int {
    if len(nums1) == 0 || len(nums2) == 0 {
        return nil
    }
    sort.Ints(nums1)
    sort.Ints(nums2)
    res := make([]int, 0)
    for i, j := 0, 0; i < len(nums1) && j < len(nums2); {
        if nums1[i] == nums2[j] {
            res = append(res, nums1[i])
            i, j = i+1, j+1
        } else if nums1[i] > nums2[j] {
            j++
        } else {
            i++
        }
    }
    return res
}
```

- Time complexity: $$O(n1log(n1)+n2log(n2))$$
- Space complexity: $$O(1)$$

(3) Solution to 3rd follow-up question?
If only nums2 cannot fit in memory, put all elements of nums1 into a map (like Solution 1), read chunks of array that fit into the memory, and record the intersections. 

**Solution**

Use as little memory as we can. In general, it's unnecessary to use more than one map.

## [334. Increasing Triplet Subsequence](<https://leetcode.com/problems/increasing-triplet-subsequence/>)

Given an unsorted array return whether an increasing subsequence of length 3 exists or not in the array.

Formally the function should:

> Return true if there exists *i, j, k* 
> such that *arr[i]* < *arr[j]* < *arr[k]* given 0 ≤ *i* < *j* < *k* ≤ *n*-1 else return false.

**Note:** Your algorithm should run in O(*n*) time complexity and O(*1*) space complexity.

**Example 1:**

```
Input: [1,2,3,4,5]
Output: true
```

**Example 2:**

```
Input: [5,4,3,2,1]
Output: false
```

**Solution**

(1) Time Limited Exceeded

Of course brute force ...

````go
func increasingTriplet(nums []int) bool {
    if len(nums) < 3 {
        return false
    }   
    for i := 0; i < len(nums)-2; i++ {
        for j := i+1; j < len(nums)-1; j++ {
            for k := j+1; k < len(nums); k++ {
                if nums[i] < nums[j] && nums[j] < nums[k] {
                    return true
                }
            }
        }
    }
    return false
}
````

- Time complexity: $$O(n^3)$$
- Space complexity: $$O(1)$$

(2) Accepted


```go
func increasingTriplet(nums []int) bool {
    if len(nums) < 3 {
        return false
    }   
    small, middle := math.MaxInt64, math.MaxInt64
    for i := range nums {
        if nums[i] <= small {
            small = nums[i]
        } else if nums[i] <= middle {
            middle = nums[i]
        } else {
            return true
        }
    }
    return false
}
```

Let's clarify what `small`and `middle`mean:

- `small`: so far best candidate of smallest one in the triplet subsequence
- `middle`: so far best candidate of middle one in the triplet subsequence

For this problem, above code does work well. Take `[1, 0, 2, 0, -1, 3]`for example:

```
Iteration One
small = 1 middle = INF
Iteration Two
small = 0 middle = INF
Iteration Three
small = 0 middle = 2
Iteration Four (Nothing Changes)
small = 0 middle = 2
Iteration Five (Confusing Part)
small = -1 middle = 2
Iteration Six
return true; Since 3 > 2 && 3 > -1
```

Setting `small= -1` is important, yet doesn't change the answer in this case since `middle= 2` implies that their existed a value that was previously smaller than `2`. Notice if we had a test case like this `[1,0,2,0,-1,0,1]` we now could see the importance of the updated lower bound for `small = -1`.

However, **if the problem requires us to return the index, then this code would not work**.

- Time complexity: $$O(n)$$
- Space complexity: $$O(1)$$

**Recap**

Obviously, this is another DP problem. We don't really need `dp`array stuff in every single DP problem.

## [240. Search a 2D Matrix II](<https://leetcode.com/problems/search-a-2d-matrix-ii/>)

Write an efficient algorithm that searches for a value in an *m* x *n* matrix. This matrix has the following properties:

- Integers in each row are sorted in ascending from left to right.
- Integers in each column are sorted in ascending from top to bottom.

**Example:**

Consider the following matrix:

```
[
  [1,   4,  7, 11, 15],
  [2,   5,  8, 12, 19],
  [3,   6,  9, 16, 22],
  [10, 13, 14, 17, 24],
  [18, 21, 23, 26, 30]
]
```

Given target = `5`, return `true`.

Given target = `20`, return `false`.

**Solution**

(1) Accepted

Since every row/column is sorted, we can do binary search to every row/column.

```go
func searchMatrix(matrix [][]int, target int) bool {
	if len(matrix) == 0 || len(matrix[0]) == 0 {
		return false
	}
	for i := range matrix {
		if target >= matrix[i][0] && target <= matrix[i][len(matrix[i])-1] {
			if j := sort.SearchInts(matrix[i], target); j < len(matrix[i]) && matrix[i][j] == target {
				return true
			}
		}
	}
	return false
}
```

- Time complexity: $$O(rows*log(columns))$$
- Space complexity: $$O(1)$$

(2) Accepted

```go
func searchMatrix(matrix [][]int, target int) bool {
	if len(matrix) == 0 || len(matrix[0]) == 0 {
		return false
	}
    for r, c := 0, len(matrix[0])-1; r < len(matrix) && c >= 0; {
        if matrix[r][c] == target {
            return true
        } else if matrix[r][c] < target {
            r++
        } else {
            c--
        }
    } 
	return false
}
```

- Time complexity: $$O(rows+columns)$$
- Space complexity: $$O(1)$$

**Recap**

1. `sort.Ints`will return the position where target **should be** in array.
2. Sometimes try to iterate an array reversely and it may help.

## [238. Product of Array Except Self](<https://leetcode.com/problems/product-of-array-except-self/>)

Given an array `nums` of *n* integers where *n*> 1,  return an array `output` such that `output[i]` is equal to the product of all the elements of `nums` except `nums[i]`.

**Example:**

```
Input:  [1,2,3,4]
Output: [24,12,8,6]
```

**Note:** Please solve it **without division** and in O(*n*).

**Follow up:**
Could you solve it with constant space complexity? (The output array **does not** count as extra space for the purpose of space complexity analysis.)

**Solution**

(1) Accepted

- `left[i]=nums[0]*...*nums[i-1], i > 0`
- `right[i]=nums[len(nums)-1]*...*nums[i+1], i >= 0`

```go
func productExceptSelf(nums []int) []int {
    n := len(nums)
    res, left, right := make([]int, n), make([]int, n), make([]int, n)
    for i := range left {
        left[i], right[i] = 1, 1
    }
    for i := 1; i < n; i++ {
        left[i] = left[i-1] * nums[i-1]
    }
    for i := n-2; i >= 0; i-- {
        right[i] = right[i+1] * nums[i+1]
    }
    for i := range nums {
        res[i] = left[i] * right[i]
    }
    return res
}
```

- Time complexity: $$O(n)$$
- Space complexity: $$O(n)$$

Improvement: no extra space

```go
func productExceptSelf(nums []int) []int {
    n := len(nums)
    res := make([]int, len(nums))
    res[0] = 1
    // res[i] = nums[0]*...*nums[i-1], i > 1
    for i := 1; i < n; i++ {
        res[i] = res[i-1] * nums[i-1]
    }
    // right = nums[n-1]*...*nums[1]
    // So multiply res[i] by corresponding right 
    // to complete the computation
    right := 1
    for i := n-1; i >= 0; i-- {
        res[i] *= right
        right *= nums[i]
    }
    return res
}
```

- Time complexity: $$O(n)$$
- Space complexity: $$O(1)$$

(2) Accepted

Same idea but one pass.

```go
func productExceptSelf(nums []int) []int {
	n := len(nums)
	res := make([]int, n)
	for i := range res {
		res[i] = 1
	}
	left, right := 1, 1
	for i := 0; i < n; i++ {
		res[i] *= left
		res[n-1-i] *= right
		left, right = left*nums[i], right*nums[n-1-i]
	}
	return res
}
```

- Time complexity: $$O(n)$$
- Space complexity: $$O(1)$$

**Recap**

Product or accumulation problem ?

# Linked List

## [138. Copy List with Random Pointer](<https://leetcode.com/problems/copy-list-with-random-pointer/>)

A linked list is given such that each node contains an additional random pointer which could point to any node in the list or null.

Return a [**deep copy**](https://en.wikipedia.org/wiki/Object_copying#Deep_copy) of the list.

**Example 1:**

**![img](https://discuss.leetcode.com/uploads/files/1470150906153-2yxeznm.png)**

```
Input:
{"$id":"1","next":{"$id":"2","next":null,"random":{"$ref":"2"},"val":2},"random":{"$ref":"2"},"val":1}

Explanation:
Node 1's value is 1, both of its next and random pointer points to Node 2.
Node 2's value is 2, its next pointer points to null and its random pointer points to itself.
```

**Note:**

1. You must return the **copy of the given head** as a reference to the cloned list.

**Solution**

```go
type RandomListNode struct {
    Label  int
    Next   *RandomListNode
    Random *RandomListNode
}

func copyRandomListNode(head *RandomListNode) *RandomListNode {   
    // 1st round: copy every node and 
    // make them linked to their original nodes
    var next *RandomListNode
    for iter := head; iter != nil; iter = iter.Next {
        next = iter.Next
        copyNode := &RandomListNode{Label: iter.Label}
        // make the copy one become its original node's next node
        iter.Next = copyNode
        copyNode.Next = next
    }
    
    // 2nd round: assign random poiters for the copy nodes
    // iter.Next.Next is the original one
    for iter := head; iter != nil; iter = iter.Next.Next {
        if iter.Random != nil {
            // Note that iter.Next is the copy one 
            // and iter.Random.Next is also a copy one
            iter.Next.Random = iter.Random.Next
        }
    }
    
    // 3rd round: restore the original list and 
    // extract the copy ones
    dummy := new(RandomListNode)
    for iter, copyIter := head, dummy; iter != nil; iter, copyIter = iter.Next, copyIter.Next {
        next, copyNode := iter.Next.Next, iter.Next
        // extarct the copy one
        copyIter.Next = copyNode
        // restore the original list
        iter.Next = next
    }
    return dummy.Next
}
```

Clarify:

![138. Copy List with Random Pointer](img/138. Copy List with Random Pointer.jpg)

- Time complexity: $$O(n)$$
- Space complexity: $$O(1)$$

## [141. Linked List Cycle](<https://leetcode.com/problems/linked-list-cycle/>)

Given a linked list, determine if it has a cycle in it.

To represent a cycle in the given linked list, we use an integer `pos` which represents the position (0-indexed) in the linked list where tail connects to. If `pos` is `-1`, then there is no cycle in the linked list.

**Example 1:**

```
Input: head = [3,2,0,-4], pos = 1
Output: true
Explanation: There is a cycle in the linked list, where tail connects to the second node.
```

![img](https://assets.leetcode.com/uploads/2018/12/07/circularlinkedlist.png)

**Example 2:**

```
Input: head = [1,2], pos = 0
Output: true
Explanation: There is a cycle in the linked list, where tail connects to the first node.
```

![img](https://assets.leetcode.com/uploads/2018/12/07/circularlinkedlist_test2.png)

**Example 3:**

```
Input: head = [1], pos = -1
Output: false
Explanation: There is no cycle in the linked list.
```

![img](https://assets.leetcode.com/uploads/2018/12/07/circularlinkedlist_test3.png)

**Follow up:**

Can you solve it using *O(1)* (i.e. constant) memory?

**Solution**

(1) Accepted

```go
func hasCycle(head *ListNode) bool {
    if head == nil || head.Next == nil {
        return false
    }
    // present records whether a node has been found in list
    present := make(map[*ListNode]bool)
    for p := head; p != nil; p = p.Next {
        if present[p] {
            return true
        } else {
            present[p] = true
        }
    }
    return false
}
```

- Time complexity: $$O(n)$$
- Space complexity: $$O(n)$$

(2) Accepted

A quite commonly used algorithm for detecting a cycle in a linked list is [Floyds's algorithm](<https://en.wikipedia.org/wiki/Cycle_detection#Floyd's_Tortoise_and_Hare>).

```go
func hasCycle(head *ListNode) bool {
    if head == nil || head.Next == nil {
        return false
    }
    for slow, fast := head, head; slow != nil && fast != nil && fast.Next != nil; slow, fast = slow.Next, fast.Next.Next {
        if slow == fast {
            return true
        }
    } 
    return false
}
```

- Time complexity: $$O(n)$$
- Space complexity: $$O(1)$$

**Recap**

1. Many linked list problems can be solved by two pointers.
2. Floyd's algorithm can also be used for looking for the entry point of cycle easily:

```go
// Let's say fast is already in cycle
slow = head
for slow != fast {
    slow, fast = slow.Next, fast.Next
}
return slow 
```

## [148. Sort List](<https://leetcode.com/problems/sort-list/>)

Sort a linked list in *O*(*n* log *n*) time using constant space complexity.

**Example 1:**

```
Input: 4->2->1->3
Output: 1->2->3->4
```

**Example 2:**

```
Input: -1->5->3->4->0
Output: -1->0->3->4->5
```

**Solution**

(1) Accepted

```go
func sortList(head *ListNode) *ListNode {
    if head == nil || head.Next == nil {
        return head
    }
    
    // Split the list into two parts 
    slow, fast, pre := head, head, head 
    for fast != nil && fast.Next != nil {
        pre = slow
        slow, fast = slow.Next, fast.Next.Next
    }
    pre.Next = nil
    // Sort each part 
    l1, l2 := sortList(head), sortList(slow)
    // Merge
    return merge(l1, l2)
}

func merge(l1, l2 *ListNode) *ListNode {
    l := new(ListNode)
    p := l
    for l1 != nil && l2 != nil {
        if l1.Val < l2.Val {
            p.Next = l1
            l1 = l1.Next
        } else {
            p.Next = l2
            l2 = l2.Next
        }
        p = p.Next
    }
    if l1 != nil {
        p.Next = l1
    }
    if l2 != nil {
        p.Next = l2
    }
    return l.Next
}
```

- Time complexity: $$O(nlogn)$$
- Space complexity: $$O(logn)$$ since the program needs to store stack frames.

(2) Accepted

```go
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */

// Merge Sort
func sortList(head *ListNode) *ListNode {
    if head == nil || head.Next == nil {
        return head
    }
    
    // Calculate the lenght of list
    length := 0
    for p := head; p != nil; p = p.Next {
        length++
    }
    
    // Bottom-up merge sort
    dummy := &ListNode{Next: head}
    for l := 1; l < length; l <<= 1 {
        cur, tail := dummy.Next, dummy
        for cur != nil {
            left := cur
            right := split(left, l)
            cur, tail = split(right, l), merge(left, right, tail)
        }
    }
    return dummy.Next
}

// split the list into two parts
// while the first part contains first n ndoes
// and return the second part's head
func split(head *ListNode, length int) *ListNode {
    for l := 1; head != nil && l < length; l++ {
        head = head.Next
    }
    if head == nil {
        return nil
    }
    second := head.Next
    head.Next = nil
    return second
}


// merge two sorted lists and append it to the head
// return the tail of merged list
func merge(l1, l2, head *ListNode) *ListNode {
    cur := head
    for l1 != nil && l2 != nil {
        if l1.Val > l2.Val {
            cur.Next = l2
            cur = l2
            l2 = l2.Next
        } else {
            cur.Next = l1
            cur = l1
            l1 = l1.Next
        }
    }
    if l1 != nil {
        cur.Next = l1
    } else if l2 != nil {
        cur.Next = l2
    }
    // Get the tail
    for cur.Next != nil {
        cur = cur.Next
    }
    return cur
}
```

- Time complexity: $$O(nlogn)$$
- Space complexity: $$O(1)$$

**Recap**

Bottom-up merge sort can save extra space.

## [160. Intersection of Two Linked Lists](<https://leetcode.com/problems/intersection-of-two-linked-lists/>)

Write a program to find the node at which the intersection of two singly linked lists begins.

For example, the following two linked lists:

![img](https://assets.leetcode.com/uploads/2018/12/13/160_statement.png)

begin to intersect at node c1.

**Notes:**

- If the two linked lists have no intersection at all, return `null`.
- The linked lists must retain their original structure after the function returns.
- You may assume there are no cycles anywhere in the entire linked structure.
- Your code should preferably run in O(n) time and use only O(1) memory.

**Solution**

(1) Accepted

Very straightforward.

```go
func getIntersectionNode(headA, headB *ListNode) *ListNode {
    if headA == nil || headB == nil {
        return nil
    }
    lenOfList := func(head *ListNode) int {
        l := 0
        for p := head; p != nil; p = p.Next {
            l++
        }
        return l
    }
    len1, len2 := lenOfList(headA), lenOfList(headB)
    p, q := headA, headB
    if gap := len1-len2; gap > 0 {
        for ; gap > 0; gap-- {
            p = p.Next
        }
    } else if gap < 0 {
        for ; gap < 0; gap++ {
            q = q.Next
        }
    }
    for p != nil && q != nil {
        if p == q {
            return p
        } else {
            p, q = p.Next, q.Next
        }
    }
    return nil
}
```

- Time complexity: $$O(len1+len2)$$
- Space complexity: $$O(1)$$

(2) Accepted but not satisfied

```go
func getIntersectionNode(headA, headB *ListNode) *ListNode {
    if headA == nil || headB == nil {
        return nil
    }
    present := make(map[*ListNode]bool)
    for p := headA; p != nil; p = p.Next {
        present[p] = true
    }
    for q := headB; q != nil; q = q.Next {
        if present[q] {
            return q
        }
    }
    return nil
}
```

- Time complexity: $$O(len1+len2)$$
- Space complexity: $$O(len1)$$

(3) Accepted

```go
func getIntersectionNode(headA, headB *ListNode) *ListNode {
    if headA == nil || headB == nil {
        return nil
    }
    a, b := headA, headB
    for a != b {
        if a == nil {
            a = headB
        } else {
            a = a.Next
        }
        if b == nil {
            b = headA
        } else {
            b = b.Next
        }
    }
    return a
}
```

In the for loop, we actually do two iterations. In the first iteration, we will reset the pointer of one linked list to the head of another linked list after it reaches the tail node. In the second iteration, we will move two pointers until they points to the same node. Our operations in first iteration will help us counteract the difference of lengths. 

So if two linked list intersects, the meeting point in second iteration must be the intersection point. If the two linked lists have no intersection at all, then the meeting pointer in second iteration must be the tail node of both lists, which is null.

- Time complexity: $$O(len1+len2)$$
- Space complexity: $$O(1)$$

## [206. Reverse Linked List](<https://leetcode.com/problems/reverse-linked-list/>)

Reverse a singly linked list.

**Example:**

```
Input: 1->2->3->4->5->NULL
Output: 5->4->3->2->1->NULL
```

**Follow up:**

A linked list can be reversed either iteratively or recursively. Could you implement both?

(1) Accepted

Iterative solution.

```go
func reverseList(head *ListNode) *ListNode {
    if head == nil || head.Next == nil {
        return head
    }
    pre, cur := head, head.Next
    var next *ListNode
    for cur != nil {
        next = cur.Next
        cur.Next = pre
        pre = cur
        cur = next
    }
    head.Next = nil
    return pre
}
```

- Time complexity: $$O(n)$$
- Space complexity: $$O(1)$$

(2) Accepted

Recursive solution.

The recursive version is slightly trickier and the key is to work backwards. Assume that the rest of the list had already been reversed, now how do I reverse the front part? Let's assume the list is: n1 → … → nk-1→ nk → nk+1 → … → nm → Ø

Assume from node nk+1 to nm had been reversed and you are at node nk.

n1 → … → nk-1 → **nk** → nk+1 ← … ← nm

We want nk+1’s next node to point to nk. So, `nk.next.next = nk`.

Be very careful that n1's next must point to Ø. If you forget about this, your linked list has a cycle in it. This bug could be caught if you test your code with a linked list of size 2.

```go
func reverseList(head *ListNode) *ListNode {
    if head == nil || head.Next == nil {
        return head
    }
    p := reverseList(head.Next)
    head.Next.Next = head
    head.Next = nil
    return p
}
```

- Time complexity: $$O(n)$$
- Space complexity: $$O(n)$$ for stack frames.

**Recap**

This problem is too classic and must be solved without doubt.

## [234. Palindrome Linked List](<https://leetcode.com/problems/palindrome-linked-list/>)

Given a singly linked list, determine if it is a palindrome.

**Example 1:**

```
Input: 1->2
Output: false
```

**Example 2:**

```
Input: 1->2->2->1
Output: true
```

**Follow up:**
Could you do it in O(n) time and O(1) space?

**Solution**

(1) Accepted

```go
func isPalindrome(head *ListNode) bool {
    if head == nil {
        return true
    }
    vals := make([]int, 0)
    for p := head; p != nil; p = p.Next {
        vals = append(vals, p.Val)
    }
    for i, j := 0, len(vals)-1; i < j; i, j = i+1, j-1 {
        if vals[i] != vals[j] {
            return false
        }
    }
    return true
}
```

- Time complexity: $$O(n)$$
- Space complexity: $$O(n)$$

(2) Accepted

Reverse the right half and compare two halves.

```go
func isPalindrome(head *ListNode) bool {
    if head == nil {
        return true
    }
    slow, fast := head, head
    for fast != nil && fast.Next != nil {
        slow, fast = slow.Next, fast.Next.Next
    }
    if fast != nil {
        // odd nodes: let right half smaller
        slow = slow.Next
    }
    for slow, fast = reverse(slow), head; slow != nil; slow, fast = slow.Next, fast.Next {
        if slow.Val != fast.Val {
            return false
        }
    }
    return true
}

func reverse(head *ListNode) *ListNode {
    if head == nil || head.Next == nil {
        return head
    }
    pre, cur := head, head.Next
    var next *ListNode
    for cur != nil {
        next = cur.Next
        cur.Next = pre
        pre = cur
        cur = next
    }
    head.Next = nil
    return pre
}
```

Note that this method modifies input. If that's not allowed, we need to restore the linked list at last.

- Time complexity: $$O(n)$$
- Space complexity: $$O(1)$$

**Recap**

Ask your interview whether you can modify input if you are not sure.

## [237. Delete Node in a Linked List](<https://leetcode.com/problems/delete-node-in-a-linked-list/>)

Write a function to delete a node (except the tail) in a singly linked list, given only access to that node.

Given linked list -- head = [4,5,1,9], which looks like following:

![img](https://assets.leetcode.com/uploads/2018/12/28/237_example.png)

**Example 1:**

```
Input: head = [4,5,1,9], node = 5
Output: [4,1,9]
Explanation: You are given the second node with value 5, the linked list should become 4 -> 1 -> 9 after calling your function.
```

**Example 2:**

```
Input: head = [4,5,1,9], node = 1
Output: [4,5,9]
Explanation: You are given the third node with value 1, the linked list should become 4 -> 5 -> 9 after calling your function.
```

**Note:**

- The linked list will have at least two elements.
- All of the nodes' values will be unique.
- The given node will not be the tail and it will always be a valid node of the linked list.
- Do not return anything from your function.

**Solution**

```go
func deleteNode(node *ListNode) {
    pre, cur := node, node.Next
    for ; cur != nil && cur.Next != nil; pre, cur = cur, cur.Next {
        pre.Val = cur.Val
    }
    pre.Val, pre.Next = cur.Val, nil
}
```

Actually, we just need to swap the current node with its next...

![img](https://leetcode.com/media/original_images/237_LinkedList3.png)

```go
func deleteNode(node *ListNode) {
    node.Val, node.Next = node.Next.Val, node.Next.Next
}
```

- Time complexity: $$O(1)$$
- Space complexity: $$O(1)$$

## [328. Odd Even Linked List](<https://leetcode.com/problems/odd-even-linked-list/>)

Given a singly linked list, group all odd nodes together followed by the even nodes. Please note here we are talking about the node number and not the value in the nodes.

You should try to do it in place. The program should run in O(1) space complexity and O(nodes) time complexity.

**Example 1:**

```
Input: 1->2->3->4->5->NULL
Output: 1->3->5->2->4->NULL
```

**Example 2:**

```
Input: 2->1->3->5->6->4->7->NULL
Output: 2->3->6->7->1->5->4->NULL
```

**Note:**

- The relative order inside both the even and odd groups should remain as it was in the input.
- The first node is considered odd, the second node even and so on ...

**Solution**

Just another two-pointer problem.

```go
func oddEvenList(head *ListNode) *ListNode {
	if head == nil {
		return head
	}
	// two pointers
	odd, even := head, head.Next
	evenFirst := head.Next
	for even != nil && even.Next != nil  {
		odd.Next = odd.Next.Next
		even.Next = even.Next.Next
		odd = odd.Next
		even = even.Next
	}
	odd.Next = evenFirst
	return head
}
```

- Time complexity: $$O(n)$$
- Space complexity: $$O(1)$$

# Heap & Stack

## [155. Min Stack](<https://leetcode.com/problems/min-stack/>)

Design a stack that supports push, pop, top, and retrieving the minimum element in constant time.

- push(x) -- Push element x onto stack.
- pop() -- Removes the element on top of the stack.
- top() -- Get the top element.
- getMin() -- Retrieve the minimum element in the stack.

**Example:**

```
MinStack minStack = new MinStack();
minStack.push(-2);
minStack.push(0);
minStack.push(-3);
minStack.getMin();   --> Returns -3.
minStack.pop();
minStack.top();      --> Returns 0.
minStack.getMin();   --> Returns -2.
```

**Solution**

(1) Accepted

```go
type MinStack struct {
    Data []int
    Mins []int
}


/** initialize your data structure here. */
func Constructor() MinStack {
    return MinStack{make([]int, 0), make([]int, 0)}
}


func (this *MinStack) Push(x int)  {
    this.Data = append(this.Data, x)
    if len(this.Mins) == 0 || this.Mins[len(this.Mins)-1] >= x {
        this.Mins = append(this.Mins, x)
    }
}


func (this *MinStack) Pop()  {
    if len(this.Data) == 0 {
        return
    }
    top := this.Data[len(this.Data)-1]
    this.Data = this.Data[:len(this.Data)-1]
    if len(this.Mins) > 0 && this.Mins[len(this.Mins)-1] == top {
        this.Mins = this.Mins[:len(this.Mins)-1]
    }
}


func (this *MinStack) Top() int {
    if len(this.Data) == 0 {
        panic("empty stack")
    }
    return this.Data[len(this.Data)-1]
}


func (this *MinStack) GetMin() int {
    if len(this.Data) == 0 {
        panic("empty stack")
    }
    return this.Mins[len(this.Mins)-1]
}
```

(2) Accepted

Only one stack.

```go
type MinStack struct {
    Data []int
    Min  int
}


/** initialize your data structure here. */
func Constructor() MinStack {
    return MinStack{make([]int, 0), math.MaxInt64}
}


func (this *MinStack) Push(x int)  {
    if x <= this.Min {
        // When updating the minimum, we need to push old minimum too
        // By doing so, we can resotre minimum to last value when we 
        // pop current minimum
        this.Data, this.Min = append(this.Data, this.Min), x
    }
    this.Data = append(this.Data, x)
}


func (this *MinStack) Pop()  {
    if len(this.Data) == 0 {
        return
    }
    top := this.Data[len(this.Data)-1]
    this.Data = this.Data[:len(this.Data)-1]
    if top == this.Min {
        this.Min = this.Data[len(this.Data)-1]
        this.Data = this.Data[:len(this.Data)-1]
    }
}


func (this *MinStack) Top() int {
    if len(this.Data) == 0 {
        panic("empty stack")
    }
    return this.Data[len(this.Data)-1]
}


func (this *MinStack) GetMin() int {
    if len(this.Data) == 0 {
        panic("empty stack")
    }
    return this.Min
}
```

(3) Accepted

Not so straightforward but it does work.

```go
type MinStack struct {
    Data []int
    Mins []int
}


/** initialize your data structure here. */
func Constructor() MinStack {
    return MinStack{make([]int, 0), make([]int, 0)}
}


func (this *MinStack) Push(x int)  {
    this.Data = append(this.Data, x)
    if len(this.Mins) == 0 || this.Mins[len(this.Mins)-1] > x {
        this.Mins = append(this.Mins, x)
    } else {
        // Push a placeholder number
        this.Mins = append(this.Mins, this.Mins[len(this.Mins)-1])
    }
}


func (this *MinStack) Pop()  {
    if len(this.Data) == 0 {
        return
    }
    this.Data = this.Data[:len(this.Data)-1]
    // If this.Mins[len(this.Mins)] == this.Data[len(this.Data)1], jsut pop it
    // if not, this.Mins[len(this.Mins)] is just a placeholder number, pop it too
    this.Mins = this.Mins[:len(this.Mins)-1]
}


func (this *MinStack) Top() int {
    if len(this.Data) == 0 {
        panic("empty stack")
    }
    return this.Data[len(this.Data)-1]
}


func (this *MinStack) GetMin() int {
    if len(this.Data) == 0 {
        panic("empty stack")
    }
    return this.Mins[len(this.Mins)-1]
}
```

**Recap**

Make use of the natures of `push`and `pop` to get constant run time.

## [215. Kth Largest Element in an Array](<https://leetcode.com/problems/kth-largest-element-in-an-array/>)

Find the **k**th largest element in an unsorted array. Note that it is the kth largest element in the sorted order, not the kth distinct element.

**Example 1:**

```
Input: [3,2,1,5,6,4] and k = 2
Output: 5
```

**Example 2:**

```
Input: [3,2,3,1,2,4,5,5,6] and k = 4
Output: 4
```

**Note:** 
You may assume k is always valid, 1 ≤ k ≤ array's length.

**Solution**

(1) Accepted

Very straightforward.

```go
func findKthLargest(nums []int, k int) int {
    if len(nums) == 0 {
        panic("invalid input")
    }
    sort.Ints(nums)
    i := len(nums)-1
    for j := 0; i >= 0 && j < k-1; i, j = i-1, j+1 {
    } 
    return nums[i]
}
```

- Time  complexity: $$O(nlog(n))$$
- Space complexity: $$O(1)$$

(2) Accepted

The smart approach for this problem is to use the selection algorithm (based on the partitioning method - the same one as used in quicksort). Notice that quick sort will cost $$O(n^2)$$ time in the worst case. To avoid this, just shuffle input array.

```go
func findKthLargest(nums []int, k int) int {
	if len(nums) == 0 {
		panic("invalid input")
	}
	rand.Shuffle(len(nums), func(i, j int) {
		nums[i], nums[j] = nums[j], nums[i]
	})
	k--
	for low, high := 0, len(nums)-1; low < high; {
		if tmp := partition(nums[low:high+1])+low; tmp == k {
			break
		} else if tmp > k {
			high = tmp - 1
		} else {
			low = tmp + 1
		}
	}
	return nums[k]
}

func partition(nums []int) int {
	i, j := 0, len(nums)
	for pivot := nums[0]; ; {
		for i++; i < len(nums) && nums[i] > pivot; i++ {
		}
		for j--; j >= 0 && nums[j] < pivot; j-- {
		}
		if i >= j {
			break
		} else {
			nums[i], nums[j] = nums[j], nums[i]
		}
	}
	nums[0], nums[j] = nums[j], nums[0]
	return j
}
```

- Time complexity: $$O(n)$$
- Space complexity: $$O(1)$$

**Recap**

In quick sort, every time we call `partition`, we will put one element in its right position in the sorted array.

## [295. Find Median from Data Stream](<https://leetcode.com/problems/find-median-from-data-stream/>)

Median is the middle value in an ordered integer list. If the size of the list is even, there is no middle value. So the median is the mean of the two middle value.

For example,

```
[2,3,4]`, the median is `3
[2,3]`, the median is `(2 + 3) / 2 = 2.5
```

Design a data structure that supports the following two operations:

- void addNum(int num) - Add a integer number from the data stream to the data structure.
- double findMedian() - Return the median of all elements so far.

 

**Example:**

```
addNum(1)
addNum(2)
findMedian() -> 1.5
addNum(3) 
findMedian() -> 2
```

 

**Follow up:**

1. If all integer numbers from the stream are between 0 and 100, how would you optimize it?
2. If 99% of all integer numbers from the stream are between 0 and 100, how would you optimize it?

**Solution**

(1) Time Limit Exceeded

```go
type MedianFinder struct {
    Data []int
}


/** initialize your data structure here. */
func Constructor() MedianFinder {
    return MedianFinder{make([]int, 0)}
}


func (this *MedianFinder) AddNum(num int)  {
    this.Data = append(this.Data, num)
}


func (this *MedianFinder) FindMedian() float64 {
    sort.Ints(this.Data)
    if len(this.Data)&1 == 0 {
        // Even 
        i := len(this.Data) >> 1
        j := i - 1
        return float64((this.Data[i]+this.Data[j])) / 2.0
    } else {
        // Odd
        return float64(this.Data[len(this.Data)>>1])
    }
}


/**
 * Your MedianFinder object will be instantiated and called as such:
 * obj := Constructor();
 * obj.AddNum(num);
 * param_2 := obj.FindMedian();
 */
```

- Time complexity: $$O(nlogn)$$
- Space complexity: $$O(n)$$

(2) 

Keep two heaps (or priority queues):

- Max-heap `small` has the smaller half of the numbers.
- Min-heap `large` has the larger half of the numbers.

```go
type PeekHeap interface {
    heap.Interface
    Peek() interface{}
}

type heapInt []int

func (h heapInt) Len() int {
	return len(h)
}

func (h heapInt) Less(i, j int) bool {
	return h[i] < h[j]
}

func (h heapInt) Swap(i, j int) {
	h[i], h[j] = h[j], h[i]
}

func (h *heapInt) Peek() interface{} {
	
	return (*h)[0]
}

func (h *heapInt) Push(x interface{}) {
	*h = append(*h, x.(int))
}

func (h *heapInt) Pop() interface{} {
	length := len(*h)
	res := (*h)[length - 1]

	*h = (*h)[0 : length - 1]
	return res
}

type reverse struct {
    PeekHeap
}

func (r reverse) Less(i, j int) bool {
	return r.PeekHeap.Less(j, i)
}

func Reverse(data PeekHeap) PeekHeap {
	return &reverse{data}
}

type MedianFinder struct {
	maxHeap PeekHeap
	minHeap PeekHeap
}


/** initialize your data structure here. */
func Constructor() MedianFinder {
    minHeap := &heapInt{}
	maxHeap := Reverse(&heapInt{})
	heap.Init(minHeap)
	heap.Init(maxHeap)
	return MedianFinder{maxHeap, minHeap}
}


func (this *MedianFinder) AddNum(num int)  {
    heap.Push(this.maxHeap, num)
    heap.Push(this.minHeap, heap.Pop(this.maxHeap))
    if this.maxHeap.Len() < this.minHeap.Len() {
        heap.Push(this.maxHeap, heap.Pop(this.minHeap))
    }
}


func (this *MedianFinder) FindMedian() float64 {
	if this.maxHeap.Len() == this.minHeap.Len() {
		return (float64(this.maxHeap.Peek().(int)) + float64(this.minHeap.Peek().(int))) / 2.0
	} else {
		return float64(this.maxHeap.Peek().(int))
	}
}


/**
 * Your MedianFinder object will be instantiated and called as such:
 * obj := Constructor();
 * obj.AddNum(num);
 * param_2 := obj.FindMedian();
 */
```

- Time complexity: `AddNum`costs $$O(logn)$$ time. `FindMedian` costs $$O(1)$$ time.
- Space complexity: $$O(n)$$

**Recap**

Try to feel comfortable with [heap pakcage](<https://golang.org/pkg/container/heap/#example__intHeap>). 

## [378. Kth Smallest Element in a Sorted Matrix](<https://leetcode.com/problems/kth-smallest-element-in-a-sorted-matrix/>)

Given a *n* x *n* matrix where each of the rows and columns are sorted in ascending order, find the kth smallest element in the matrix.

Note that it is the kth smallest element in the sorted order, not the kth distinct element.

**Example:**

```
matrix = [
   [ 1,  5,  9],
   [10, 11, 13],
   [12, 13, 15]
],
k = 8,

return 13.
```

**Note:** 
You may assume k is always valid, 1 ≤ k ≤ n^2.

**Solution**

(1) Wrong Answer

This solution fails because there may be duplicates in matrix.

```go
func kthSmallest(matrix [][]int, k int) int {
    if len(matrix) == 0 || len(matrix[0]) == 0 || len(matrix) != len(matrix[0]) {
        panic("invalid input")
    }
    n := len(matrix)
    k--
    r, c := k/n, k%n
    return matrix[r][c]
}
```

```
Input
[[1,2],[1,3]]
2
Output
2
Expected
1
```

(2) Accepted

Two steps:

1. Build a min-heap of elements from the first row.
2. Do the following operations k-1 times :
   Every time when you poll out the root of heap, you need to know the row number and column number of that element(so we can create a tuple class here), replace that root with the next element from the same column.

In this approach, we actually flat the matrix into an sorted array.

```go
type MatrixItem struct {
	Row    int
	Column int
	Val    int
}

type Matrix []*MatrixItem

func (m Matrix) Len() int {
	return len(m)
}

func (m Matrix) Less(i, j int) bool {
	return m[i].Val < m[j].Val
}

func (m Matrix) Swap(i, j int) {
	m[i], m[j] = m[j], m[i]
}

func (m *Matrix) Push(x interface{}) {
	*m = append(*m, x.(*MatrixItem))
}

func (m *Matrix) Pop() interface{} {
	peek := (*m)[len(*m)-1]
	*m = (*m)[:len(*m)-1]
	return peek
}

func kthSmallest(matrix [][]int, k int) int {
	if len(matrix) == 0 || len(matrix[0]) == 0 || len(matrix) != len(matrix[0]) {
		panic("invalid input")
	}
	m := new(Matrix)
	heap.Init(m)
	n := len(matrix)
	for i := 0; i < n; i++ {
		heap.Push(m, &MatrixItem{0, i, matrix[0][i]})
	}
	for k--; k > 0; k-- {
		peek := heap.Pop(m).(*MatrixItem)
		if peek.Row < n-1 {
			heap.Push(m, &MatrixItem{peek.Row + 1, peek.Column, matrix[peek.Row+1][peek.Column]})
		}
	}
	return heap.Pop(m).(*MatrixItem).Val
}
```

- Time complexity: $$O(klogn)$$
- Space complexity: $$O(n)$$

**Recap**

Similar problem: [373. Find K Pairs with Smallest Sums](373. Find K Pairs with Smallest Sums)

## [373. Find K Pairs with Smallest Sums](373. Find K Pairs with Smallest Sums)

You are given two integer arrays **nums1 **and **nums2** sorted in ascending order and an integer **k**.

Define a pair **(u,v)** which consists of one element from the first array and one element from the second array.

Find the k pairs **(u1,v1),(u2,v2) ...(uk,vk)**with the smallest sums.

**Example 1:**

```
Input: nums1 = [1,7,11], nums2 = [2,4,6], k = 3
Output: [[1,2],[1,4],[1,6]] 
Explanation: The first 3 pairs are returned from the sequence: 
             [1,2],[1,4],[1,6],[7,2],[7,4],[11,2],[7,6],[11,4],[11,6]
```

**Example 2:**

```
Input: nums1 = [1,1,2], nums2 = [1,2,3], k = 2
Output: [1,1],[1,1]
Explanation: The first 2 pairs are returned from the sequence: 
             [1,1],[1,1],[1,2],[2,1],[1,2],[2,2],[1,3],[1,3],[2,3]
```

**Example 3:**

```
Input: nums1 = [1,2], nums2 = [3], k = 3
Output: [1,3],[2,3]
Explanation: All possible pairs are returned from the sequence: [1,3],[2,3]
```

**Solution**

Generate possible pairs and store them in a min heap/priority queue. Return the first k pairs.

```go
type Pair struct {
    IdxInNums1 int
    IdxInNums2 int
    Val        int
}

type PairHeap []Pair

func (ph PairHeap) Len() int {
    return len(ph)
}

func (ph PairHeap) Less(i, j int) bool {
    return ph[i].Val < ph[j].Val
}

func (ph PairHeap) Swap(i, j int) {
    ph[i], ph[j] = ph[j], ph[i]
}

func (ph *PairHeap) Push(x interface{}) {
    *ph = append(*ph, x.(Pair))
}

func (ph *PairHeap) Pop() interface{} {
    root := (*ph)[len(*ph)-1]
    *ph = (*ph)[:len(*ph)-1]
    return root
}


func kSmallestPairs(nums1 []int, nums2 []int, k int) [][]int {
    if len(nums1) == 0 || len(nums2) == 0 {
        return nil
    }
    // Init heap
    ph := new(PairHeap)
    heap.Init(ph)
    for i := 0; i < len(nums2); i++ {
        heap.Push(ph, Pair{0, i, nums1[0]+nums2[i]})
    }
    // Generate ohther pairs and get first k smallest paris
    res := make([][]int, 0)
    if k > len(nums1)*len(nums2) {
        k = len(nums1) * len(nums2)
    }
    for i := 0; i < k; i++ {
        pair := heap.Pop(ph).(Pair)
        res = append(res, []int{nums1[pair.IdxInNums1], nums2[pair.IdxInNums2]})
        if pair.IdxInNums1 < len(nums1)-1 {
            heap.Push(ph, Pair{pair.IdxInNums1+1, pair.IdxInNums2, nums1[pair.IdxInNums1+1]+nums2[pair.IdxInNums2]})
        }
    }
    return res
}
```

We can also consider this approach as a multiway merge sort:

![](https://pbs.twimg.com/media/Dg-5jocU0AAI-cC.jpg:small)

- Time complexity: $$O(klogk)$$
- Space complexity: $$O(len(nums1))$$ or $$O(len(nums2))$$

## [347. Top K Frequent Elements](<https://leetcode.com/problems/top-k-frequent-elements/>)

Given a non-empty array of integers, return the **k** most frequent elements.

**Example 1:**

```
Input: nums = [1,1,1,2,2,3], k = 2
Output: [1,2]
```

**Example 2:**

```
Input: nums = [1], k = 1
Output: [1]
```

**Note:**

- You may assume *k* is always valid, 1 ≤ *k* ≤ number of unique elements.
- Your algorithm's time complexity **must be** better than O(*n* log *n*), where *n* is the array's size.

**Solution**

```go
func topKFrequent(nums []int, k int) []int {
    // Count the frequency of every element
    numToFreq := make(map[int]int)
    for i := range nums {
        numToFreq[nums[i]]++
    }
    // Group elements by their frequencies
    maxFreq := 0
    freqToNum := make(map[int][]int)
    for n, f := range numToFreq {
        if _, ok := freqToNum[f]; !ok {
            freqToNum[f] = make([]int, 0)
        } 
        freqToNum[f] = append(freqToNum[f], n)
        if f > maxFreq {
            maxFreq = f
        }
    }
    // Get k most frequent elements
    res := make([]int, 0)
    for i := maxFreq; i > 0; i-- {
        if _, ok := freqToNum[i]; ok {
            res = append(res, freqToNum[i]...)
            if len(res) == k {
                break
            }
        }
    }
    return res
}
```

- Time complexity: $$O(n)$$
- Space complexity: $$O(m)$$ where m is the number of distinct numbers

## [150. Evaluate Reverse Polish Notation](<https://leetcode.com/problems/evaluate-reverse-polish-notation/>)

Evaluate the value of an arithmetic expression in [Reverse Polish Notation](http://en.wikipedia.org/wiki/Reverse_Polish_notation).

Valid operators are `+`, `-`, `*`, `/`. Each operand may be an integer or another expression.

**Note:**

- Division between two integers should truncate toward zero.
- The given RPN expression is always valid. That means the expression would always evaluate to a result and there won't be any divide by zero operation.

**Example 1:**

```
Input: ["2", "1", "+", "3", "*"]
Output: 9
Explanation: ((2 + 1) * 3) = 9
```

**Example 2:**

```
Input: ["4", "13", "5", "/", "+"]
Output: 6
Explanation: (4 + (13 / 5)) = 6
```

**Example 3:**

```
Input: ["10", "6", "9", "3", "+", "-11", "*", "/", "*", "17", "+", "5", "+"]
Output: 22
Explanation: 
  ((10 * (6 / ((9 + 3) * -11))) + 17) + 5
= ((10 * (6 / (12 * -11))) + 17) + 5
= ((10 * (6 / -132)) + 17) + 5
= ((10 * 0) + 17) + 5
= (0 + 17) + 5
= 17 + 5
= 22
```

**Solution**

Accepted

```go
func evalRPN(tokens []string) int {
    if len(tokens) == 0 {
        return 0
    }
    calculate := func(a int, b int, op string) int {
        switch op {
            case "+":
                return a + b
            case "-":
                return a - b
            case "*":
                return a * b
            case "/":
                return a / b
            default:
                panic("invalid operation")
        }
    }
    stack := make([]int, 0)
    for _, t := range tokens {
        if t != "+" && t != "-" && t != "*" && t != "/" {
            num, _ := strconv.Atoi(t)
            stack = append(stack, num)
        } else {
            tmp := calculate(stack[len(stack)-2], stack[len(stack)-1], t)
            stack = stack[:len(stack)-2]
            stack = append(stack, tmp)
        }
    }
    return stack[0]
}
```

- Time complexity: $$O(n)$$ where n is the length of `tokens`.
- Space complexity: $$O(m)$$ where m is the number of operands.

## [227. Basic Calculator II](<https://leetcode.com/problems/basic-calculator-ii/>)

Implement a basic calculator to evaluate a simple expression string.

The expression string contains only **non-negative** integers, `+`, `-`, `*`, `/` operators and empty spaces ``. The integer division should truncate toward zero.

**Example 1:**

```
Input: "3+2*2"
Output: 7
```

**Example 2:**

```
Input: " 3/2 "
Output: 1
```

**Example 3:**

```
Input: " 3+5 / 2 "
Output: 5
```

**Note:**

- You may assume that the given expression is always valid.
- **Do not** use the `eval` built-in library function.

**Solution**

(1) Wrong Answer

```go
func calculate(s string) int {
	// Convert infix expression to postfix expression
	var sb strings.Builder
	postfix := make([]string, 0)
	ops := make([]string, 0)
	precedence := func(op string) int {
		switch op {
		case "+", "-":
			return 1
		default:
			return 2
		}
	}
	for _, r := range strings.TrimSpace(s) {
		if unicode.IsDigit(r) {
			sb.WriteRune(r)
		} else {
			if sb.Len() > 0 {
				postfix = append(postfix, sb.String())
				sb.Reset()
			}
			if r != ' ' {
				if len(ops) > 0 && precedence(string(r)) <= precedence(ops[len(ops)-1]) {
					postfix = append(postfix, ops[len(ops)-1])
					ops = ops[:len(ops)-1]
				}
				ops = append(ops, string(r))
			}
		}
	}
	if sb.Len() > 0 {
		postfix = append(postfix, sb.String())
	}
	for i := len(ops) - 1; i >= 0; i-- {
		postfix = append(postfix, ops[i])
	}
	// Evaluate
	return eval(postfix)
}

func eval(tokens []string) int {
    if len(tokens) == 0 {
        return 0
    }
    calculate := func(a int, b int, op string) int {
        switch op {
            case "+":
                return a + b
            case "-":
                return a - b
            case "*":
                return a * b
            case "/":
                return a / b
            default:
                panic("invalid operation")
        }
    }
    stack := make([]int, 0)
    for _, t := range tokens {
        if t != "+" && t != "-" && t != "*" && t != "/" {
            num, _ := strconv.Atoi(t)
            stack = append(stack, num)
        } else {
            tmp := calculate(stack[len(stack)-2], stack[len(stack)-1], t)
            stack = stack[:len(stack)-2]
            stack = append(stack, tmp)
        }
    }
    return stack[0]
}
```

```
Input
"1*2-3/4+5*6-7*8+9/10"
Output
28
Expected
-24
```

There are not any parentheses in the expression so we can not use [Shunting-yard algorithm](<https://en.wikipedia.org/wiki/Shunting-yard_algorithm>) to generate a postfix expression.

(2) Accepted

```go
func calculate(s string) int {
	if len(s) == 0 {
		return 0
	}
	var sb strings.Builder
	preSign := '+' // leading symbol of previous number
	stack := make([]int, 0)
	s = strings.TrimSpace(s)
	for i, r := range s {
		if unicode.IsDigit(r) {
			sb.WriteRune(r)
		}
		if (!unicode.IsDigit(r) && r != ' ') || i == len(s)-1 {
			num, _ := strconv.Atoi(sb.String())
			sb.Reset()
			switch preSign {
			case '+':
				stack = append(stack, num)
			case '-':
				stack = append(stack, -num)
			case '*':
				top := stack[len(stack)-1]
				stack = stack[:len(stack)-1]
				stack = append(stack, top*num)
			case '/':
				top := stack[len(stack)-1]
				stack = stack[:len(stack)-1]
				stack = append(stack, top/num)
			}
			preSign = r
		}
	}
	res := 0
	for i := range stack {
		res += stack[i]
	}
	return res
}
```

- Time complexity: $$O(n)$$
- Space complexity: $$O(n)$$

## [341. Flatten Nested List Iterator](<https://leetcode.com/problems/flatten-nested-list-iterator/>)

Given a nested list of integers, implement an iterator to flatten it.

Each element is either an integer, or a list -- whose elements may also be integers or other lists.

**Example 1:**

```
Input: [[1,1],2,[1,1]]
Output: [1,1,2,1,1]
Explanation: By calling next repeatedly until hasNext returns false, 
             the order of elements returned by next should be: [1,1,2,1,1].
```

**Example 2:**

```
Input: [1,[4,[6]]]
Output: [1,4,6]
Explanation: By calling next repeatedly until hasNext returns false, 
             the order of elements returned by next should be: [1,4,6].
```

**Solution**

```java
/**
 * // This is the interface that allows for creating nested lists.
 * // You should not implement it, or speculate about its implementation
 * public interface NestedInteger {
 *
 *     // @return true if this NestedInteger holds a single integer, rather than a nested list.
 *     public boolean isInteger();
 *
 *     // @return the single integer that this NestedInteger holds, if it holds a single integer
 *     // Return null if this NestedInteger holds a nested list
 *     public Integer getInteger();
 *
 *     // @return the nested list that this NestedInteger holds, if it holds a nested list
 *     // Return null if this NestedInteger holds a single integer
 *     public List<NestedInteger> getList();
 * }
 */
public class NestedIterator implements Iterator<Integer> {
    // all single integers
    private List<Integer> singleIntegers;
    
    private Iterator<Integer> iter;

    public NestedIterator(List<NestedInteger> nestedList) {
        singleIntegers = new LinkedList<>();
        flatten(nestedList);
        iter = singleIntegers.iterator();
    }

    @Override
    public boolean hasNext() {
        return iter.hasNext();
    }

    @Override
    public Integer next() {
        return iter.next();
    }

    // get every single integer from nestedList
    private void flatten(List<NestedInteger> nestedList) {
        for (NestedInteger n : nestedList) {
            if (n.isInteger()) {
                // if we find an integer, just add it into list
                singleIntegers.add(n.getInteger());
            } else {
                // if we find a nested list, resolve it recursively
                flatten(n.getList());
            }
        }
    }
}

/**
 * Your NestedIterator object will be instantiated and called as such:
 * NestedIterator i = new NestedIterator(nestedList);
 * while (i.hasNext()) v[f()] = i.next();
 */
```

**Recap**

Solve this problem iteratively using stack?

# Queue

## [239. Sliding Window Maximum](<https://leetcode.com/problems/sliding-window-maximum/>)

Given an array *nums*, there is a sliding window of size *k* which is moving from the very left of the array to the very right. You can only see the *k* numbers in the window. Each time the sliding window moves right by one position. Return the max sliding window.

**Example:**

```
Input: nums = [1,3,-1,-3,5,3,6,7], and k = 3
Output: [3,3,5,5,6,7] 
Explanation: 

Window position                Max
---------------               -----
[1  3  -1] -3  5  3  6  7       3
 1 [3  -1  -3] 5  3  6  7       3
 1  3 [-1  -3  5] 3  6  7       5
 1  3  -1 [-3  5  3] 6  7       5
 1  3  -1  -3 [5  3  6] 7       6
 1  3  -1  -3  5 [3  6  7]      7
```

**Note:** 
You may assume *k* is always valid, 1 ≤ k ≤ input array's size for non-empty array.

**Follow up:**
Could you solve it in linear time?

**Solution**

(1) Accepted

```go
func maxSlidingWindow(nums []int, k int) []int {
	if len(nums) == 0 || k <= 0 || k > len(nums) {
		return nil
	}
	// Find the maximum in the first window
	dp := make([][]int, 0)
	curMax, idx := nums[0], 0
	for i := 1; i < k; i++ {
		if curMax < nums[i] {
			curMax, idx = nums[i], i
		}
	}
	dp = append(dp, []int{curMax, idx})
	// Find the maximums in following windows
	max := func(l, r int) []int {
		m, i := nums[l], l
		for j := l+1; j <= r; j++ {
			if nums[j] > m {
				m, i = nums[j], j
			}
		}
		return []int{m, i}
	}
	res := []int{dp[0][0]}
	for l, r := 1, k; r < len(nums); l, r = l+1, r+1 {
		preCur, idx := dp[l-1][0], dp[l-1][1]
		if idx >= l && idx <= r {
            // The max in previous windows is still in current window
			if nums[r] > preCur {
				dp = append(dp, []int{nums[r], r})
			} else {
				dp = append(dp, dp[l-1])
			}
		} else {
			dp = append(dp, max(l, r))
		}
		res = append(res, dp[l][0])
	}
	return res
}
```

- Time complexity: worst case will cost $$O(mk)$$ where m is the number of windows.
- Space complexity: $$O(m)$$ where m is the number of windows.

(2) Accepted

At each i, we keep "promising" elements, which are potentially max number in window `[i-(k-1), i]` or any subsequent window. This means

- If an element in the deque and it is out of range, we discard them. We just need to poll from the head, as we are using a deque and elements are ordered as the sequence in the array

- Now only those elements within `[i-(k-1), i]` are in the deque. We then discard elements smaller than `a[i]` from the tail. This is because if `a[x] < a[i] && x < i`, then `a[x]` has no chance to be the "max" in `[i-(k-1), i]`, or any other subsequent window: `a[i]` would always be a better candidate.

- As a result elements in the deque are ordered in both sequence in array and their value. At each step the head of the deque is the max element in `[i-(k-1), i]`.

```go
func maxSlidingWindow(nums []int, k int) []int {
	if len(nums) == 0 || k <= 0 || k > len(nums) {
		return nil
	}
    res := make([]int, 0, len(nums)-k+1)
    deque := make([]int, 0)
    for i := range nums {
        l := i - k + 1 // Left-most index of window whose right-most index is i
        for len(deque) > 0 && deque[0] < l {
            // Discard out-of-range index
            deque = deque[1:]
        }
        for len(deque) > 0 && nums[deque[len(deque)-1]] < nums[i] {
            // Discard index at which the element can not be the max in current window
            deque = deque[:len(deque)-1]
        }
        deque = append(deque, i)
        if l >= 0 {
            res = append(res, nums[deque[0]])
        }
    }
    return res
}
```

- Time complexity: $$O(n)$$
- Space complexity: $$O(n)$$

**Recap**

A good way to solve sliding window problems is **utilizing what we've already known**. 

# String

## [125. Valid Palindrome](<https://leetcode.com/problems/valid-palindrome/>)

Given a string, determine if it is a palindrome, considering only alphanumeric characters and ignoring cases.

**Note:** For the purpose of this problem, we define empty string as valid palindrome.

**Example 1:**

```
Input: "A man, a plan, a canal: Panama"
Output: true
```

**Example 2:**

```
Input: "race a car"
Output: false
```

**Solution**

(1) Accepted

```go
func isPalindrome(s string) bool {
    if len(s) == 0 {
        return true
    }
    runes := make([]rune, 0)
    // Extract all letters or digits
    for _, r := range strings.ToLower(s) {
        if unicode.IsLetter(r) || unicode.IsDigit(r) {
            runes = append(runes, r)
        }
    }
    for i, j := 0, len(runes)-1; i < j; i, j = i+1, j-1 {
        if runes[i] != runes[j] {
            return false
        }
    }
    return true
}
```

- Time complexity: $$O(n)$$ n is the number of alphanumeric characters.
- Space complexity: $$O(n)$$

(2) Accepted

```go
func isPalindrome(s string) bool {
    if len(s) == 0 {
        return true
    }
    for i, j := 0, len(s)-1; i < j; i, j = i+1, j-1 {
        for i < j && !unicode.IsDigit(rune(s[i])) && !unicode.IsLetter(rune(s[i])) {
            i++
        }
        for i < j && !unicode.IsDigit(rune(s[j])) && !unicode.IsLetter(rune(s[j])) {
            j--
        }
        if s[i] != s[j] {
            if unicode.IsLetter(rune(s[i])) && unicode.IsLetter(rune(s[j])) {
                if math.Abs(float64(rune(s[i])-rune(s[j]))) != 32.0 {
                    return false
                }
            } else {
                return false
            }
        }
    }
    return true
}
```

More concise:

```go
func isPalindrome(s string) bool {
    if len(s) == 0 {
        return true
    }
    for i, j := 0, len(s)-1; i < j; i, j = i+1, j-1 {
        for i < j && !unicode.IsDigit(rune(s[i])) && !unicode.IsLetter(rune(s[i])) {
            i++
        }
        for i < j && !unicode.IsDigit(rune(s[j])) && !unicode.IsLetter(rune(s[j])) {
            j--
        }
        if s[i] != s[j] {
            // Just do ToLower whatever it is an letter or a digit
            if unicode.ToLower(rune(s[i])) != unicode.ToLower(rune(s[j])) {
                return false
            }
        }
    }
    return true
}
```

- Time complexity: $$O(len(s))$$
- Space complexity: $$O(1)$$

## [131. Palindrome Partitioning](<https://leetcode.com/problems/palindrome-partitioning/>)

Given a string *s*, partition *s* such that every substring of the partition is a palindrome.

Return all possible palindrome partitioning of *s*.

**Example:**

```
Input: "aab"
Output:
[
  ["aa","b"],
  ["a","a","b"]
]
```

**Solution**

First,  here if you want to get all the possible palindrome partition, first a nested for loop to get every possible partitions for a string, then a scanning for all the partitions. That's a $$O(n^2)$$ for partitioning and $$O(n^2)$$ for the scanning of string, totaling at $$O(n^4)$$ just for the partition. However, if we use a 2D array to keep track of any string we have scanned so far, with an addition pair, we can determine whether it's palindrome or not by looking at that pair. This way, the 2D array `dp` contains the possible palindrome partition among all.

Second, based on the prescanned palindrome partitions saved in `dp` array, a simple backtrack/DFS does the job.

```go
func partition(s string) [][]string {
	if len(s) == 0 {
		return nil
	}
	dp := make([][]bool, len(s))
	for i := range dp {
		dp[i] = make([]bool, len(s))
	}
    // Find palindrome
	for i := 0; i < len(s); i++ {
		for j := 0; j <= i; j++ {
			if s[i] == s[j] && (i-j <= 2 || dp[j+1][i-1]) {
				dp[j][i] = true
			}
		}
	}
    // Generate partitions
	res := make([][]string, 0)
	partition := make([]string, 0)
	dfs(s, 0, partition, dp, &res)
	return res
}

func dfs(s string, pos int, partition []string, dp [][]bool, res *[][]string) {
	if pos == len(s) {
        // Note that we need a copy to avoid following recursion's pollution
		tmp := make([]string, len(partition))
		copy(tmp, partition)
		*res = append(*res, tmp)
		return
	}
	for i := pos; i < len(s); i++ {
		if dp[pos][i] {
			partition = append(partition, s[pos:i+1])
			dfs(s, i+1, partition, dp, res)
			partition = partition[:len(partition)-1]
		}
	}
}
```

- Time complexity: `dfs` costs $$O(2^n)$$?
- Space complexity: $$O(n^2)$$

## [139. Word Break](<https://leetcode.com/problems/word-break/>)

Given a **non-empty** string *s* and a dictionary *wordDict* containing a list of **non-empty** words, determine if *s* can be segmented into a space-separated sequence of one or more dictionary words.

**Note:**

- The same word in the dictionary may be reused multiple times in the segmentation.
- You may assume the dictionary does not contain duplicate words.

**Example 1:**

```
Input: s = "leetcode", wordDict = ["leet", "code"]
Output: true
Explanation: Return true because "leetcode" can be segmented as "leet code".
```

**Example 2:**

```
Input: s = "applepenapple", wordDict = ["apple", "pen"]
Output: true
Explanation: Return true because "applepenapple" can be segmented as "apple pen apple".
             Note that you are allowed to reuse a dictionary word.
```

**Example 3:**

```
Input: s = "catsandog", wordDict = ["cats", "dog", "sand", "and", "cat"]
Output: false
```

**Solution**

A typical DP problem.

```go
func wordBreak(s string, dict []string) bool {
	if len(s) == 0 || len(dict) == 0 {
		return false
	}
	// dp[i]: can s[:i] be segmented?
	dp := make([]bool, len(s)+1)
	dp[0] = true
	for i := 1; i <= len(s); i++ {
		for _, word := range dict {
			if l := len(word); l <= i {
				if dp[i-l] && s[i-l:i] == word {
					dp[i] = true
					break
				}
			}
		}
	}
	return dp[len(s)]
}
```

- Time complexity: $$O(len(s)*len(dict))$$
- Space complexity: $$O(len(s))$$

Or can be done in this way:

```go
func wordBreak(s string, dict []string) bool {
	if len(s) == 0 || len(dict) == 0 {
		return false
	}
    contains := func(word string) bool {
        for i := range dict {
            if dict[i] == word {
                return true
            }
        }
        return false
    } 
	// dp[i]: can s[:i] be segmented?
	dp := make([]bool, len(s)+1)
	dp[0] = true
	for i := 1; i <= len(s); i++ {
        for j := 0; j < i; j++ {
            if dp[j] && contains(s[j:i]) {
                dp[i] = true
                break
            }
        }
	}
	return dp[len(s)]
}
```

- Time complexity: $$O(len(s)^2)$$
- Space complexity: $$O(len(s))$$

**Recap**

For DP problems about *sequences* like arrays or strings, try to think in this way: if we have solved the subproblem in subsequence `[:i]`, can we keep moving on to solve the entire problem?

## [140. Word Break II](<https://leetcode.com/problems/word-break-ii/>)

Given a **non-empty** string *s* and a dictionary *wordDict* containing a list of **non-empty** words, add spaces in *s* to construct a sentence where each word is a valid dictionary word. Return all such possible sentences.

**Note:**

- The same word in the dictionary may be reused multiple times in the segmentation.
- You may assume the dictionary does not contain duplicate words.

**Example 1:**

```
Input:
s = "catsanddog"
wordDict = ["cat", "cats", "and", "sand", "dog"]
Output:
[
  "cats and dog",
  "cat sand dog"
]
```

**Example 2:**

```
Input:
s = "pineapplepenapple"
wordDict = ["apple", "pen", "applepen", "pine", "pineapple"]
Output:
[
  "pine apple pen apple",
  "pineapple pen apple",
  "pine applepen apple"
]
Explanation: Note that you are allowed to reuse a dictionary word.
```

**Example 3:**

```
Input:
s = "catsandog"
wordDict = ["cats", "dog", "sand", "and", "cat"]
Output:
[]
```

**Solution**

This is a DFS problem. Find the first word, then start here to find next... Normal DFS will cause TLE so do memoization. By doing so, we are solving this problem using DP at the same time.

```go
func wordBreak(s string, wordDict []string) []string {
    return dfs(s, wordDict, make(map[string][]string))
}

// dfs segments the whole string s
// memo is the mapping from a string to its segmentations
func dfs(s string, wordDict []string, memo map[string][]string) []string {
    if segs, ok := memo[s]; ok {
        return segs
    }
    segs := make([]string, 0)
    if len(s) == 0 {
        segs = append(segs, "")
        return segs
    }
    for _, word := range wordDict {
        if strings.HasPrefix(s, word) {
            // Start here to segment subtring 
            tmp := dfs(s[len(word):], wordDict, memo)
            for _, seg := range tmp {
                if len(seg) > 0 {
                    segs = append(segs, word + " " + seg)
                } else {
                    segs = append(segs, word)
                }
            }
        }
    }
    memo[s] = segs
    return segs
}
```

## [208. Implement Trie (Prefix Tree)](<https://leetcode.com/problems/implement-trie-prefix-tree/>)

Implement a trie with `insert`, `search`, and `startsWith` methods.

**Example:**

```
Trie trie = new Trie();

trie.insert("apple");
trie.search("apple");   // returns true
trie.search("app");     // returns false
trie.startsWith("app"); // returns true
trie.insert("app");   
trie.search("app");     // returns true
```

**Note:**

- You may assume that all inputs are consist of lowercase letters `a-z`.
- All inputs are guaranteed to be non-empty strings.

**Solution**

(1) Accepted

```go
type Trie struct {
    IsWord   bool // If this node is the end of a word
	Children map[uint8]*Trie
}

func Constructor() Trie {
    return Trie{Children: make(map[uint8]*Trie)}
}


/** Inserts a word into the trie. */
func (this *Trie) Insert(word string)  {
	if len(word) == 0 {
        this.IsWord = true
		return
	}
	if _, ok := this.Children[word[0]]; !ok {
        this.Children[word[0]] = &Trie{Children: make(map[uint8]*Trie)}
	}
	this.Children[word[0]].Insert(word[1:])
}


/** Returns if the word is in the trie. */
func (this *Trie) Search(word string) bool {
    if len(word) == 0 {
        return this.IsWord
    }
    if _, ok := this.Children[word[0]]; !ok {
        return false
    } else {
        return this.Children[word[0]].Search(word[1:])
    }
}


/** Returns if there is any word in the trie that starts with the given prefix. */
func (this *Trie) StartsWith(prefix string) bool {
	if len(prefix) == 0 {
		return true
	}
	if _, ok := this.Children[prefix[0]]; !ok {
		return false
	} else {
		return this.Children[prefix[0]].StartsWith(prefix[1:])
	}
}
```

- Time complexity: $$O(m)$$ where m is the length of word/prefix/
- Space complexity: `INsert` costs $$O(m)$$ in the worst case.

(2) Accepted

Iteratively.

```go
type Trie struct {
    IsWord   bool // If this node is the end of a word
	Children map[uint8]*Trie
}

func Constructor() Trie {
    return Trie{Children: make(map[uint8]*Trie)}
}


/** Inserts a word into the trie. */
func (this *Trie) Insert(word string)  {
	if len(word) == 0 {
		return
	}
    node := this
    for i := range word {
        if _, ok := node.Children[word[i]]; !ok {
            node.Children[word[i]] = &Trie{Children: make(map[uint8]*Trie)}
        }
        node = node.Children[word[i]]
    }
    node.IsWord = true
}


/** Returns if the word is in the trie. */
func (this *Trie) Search(word string) bool {
    if len(word) == 0 {
        return false
    }
    node := this
    for i := range word {
        if _, ok := node.Children[word[i]]; !ok {
            return false
        }
        node = node.Children[word[i]]
    }
    return node.IsWord
}


/** Returns if there is any word in the trie that starts with the given prefix. */
func (this *Trie) StartsWith(prefix string) bool {
	if len(prefix) == 0 {
		return true
	}
    node := this
    for i := range prefix {
        if _, ok := node.Children[prefix[i]]; !ok {
            return false
        }
        node = node.Children[prefix[i]]
    }
    return true
}
```

## [212. Word Search II](<https://leetcode.com/problems/word-search-ii/>)

Given a 2D board and a list of words from the dictionary, find all words in the board.

Each word must be constructed from letters of sequentially adjacent cell, where "adjacent" cells are those horizontally or vertically neighboring. The same letter cell may not be used more than once in a word.

 

**Example:**

```
Input: 
board = [
  ['o','a','a','n'],
  ['e','t','a','e'],
  ['i','h','k','r'],
  ['i','f','l','v']
]
words = ["oath","pea","eat","rain"]

Output: ["eat","oath"]
```

 

**Note:**

1. All inputs are consist of lowercase letters `a-z`.
2. The values of `words` are distinct.

**Solution**

(1) Time Limit Exceeded

The most straightforward way is DFS. The code below waste much time on useless work. For example, it will continue searching even though no word in dictionary starts with current string `tmp`.

```go
func findWords(board [][]byte, words []string) []string {
    if len(board) == 0 || len(board[0]) == 0 || len(words) == 0 {
        return nil
    }
    dict := make(map[string]bool)
    for i := range words {
        dict[words[i]] = true
    }
    visited := make([][]bool, len(board))
    for i := range visited {
        visited[i] = make([]bool, len(board[0]))
    }
    set := make(map[string]bool)
    for i := range board {
        for j := range board[i] {
            dfs(board, dict, i, j, &visited, "", set)
        }
    }
    res := make([]string, 0)
    for k := range set {
        res = append(res, k)
    }
    return res
}

func dfs(board [][]byte, dict map[string]bool, i int, j int, visited *[][]bool, tmp string, set map[string]bool) {
    tmp += string(board[i][j])
    if dict[tmp] {
        set[tmp] = true
    }
    (*visited)[i][j] = true
    check := func(x, y int) bool {
        return x >= 0 && x < len(board) && y >= 0 && y < len(board[0]) && !(*visited)[x][y]
    } 
    dir := [][]int{{0, 1}, {0, -1}, {1, 0}, {-1, 0}}
    for _, d := range dir {
        x, y := i+d[0], j+d[1]
        if check(x, y) {
            dfs(board, dict, x, y, visited, tmp, set)
        }
    }
    (*visited)[i][j] = false
}
```

(2) Accepted

Since this is a word search problem, trie may help. 

The code below has some interesting optimizations:

- No `visited`array
- Store the whole word in trie node so no string concatenation
- No map/set to avoid duplicates in final answer

```go
func findWords(board [][]byte, words []string) []string {
    if len(board) == 0 || len(board[0]) == 0 || len(words) == 0 {
        return nil
    }
    // Build trie
    trie := &Trie{Next: make(map[byte]*Trie)}
    for _, w := range words {
        trie.insert(w)
    }
    res := make([]string, 0)
    for i := range board {
        for j := range board[i] {
            dfs(&board, i, j, trie, &res)
        }
    }
    return res
}

func dfs(board *[][]byte, i int, j int, trie *Trie, res *[]string) {
    char := (*board)[i][j]
    next, ok := trie.Next[char]
    if char == '#' || !ok {
        return
    }
    if len(next.Word) != 0 {
        *res = append(*res, next.Word)
        next.Word = "" // de-duplicate
    }
    (*board)[i][j] = '#'
    if i > 0 {
        dfs(board, i-1, j, next, res)
    }
    if j > 0 {
        dfs(board, i, j-1, next, res)
    }
    if i < len(*board)-1 {
        dfs(board, i+1, j, next, res)
    }
    if j < len((*board)[0])-1 {
        dfs(board, i, j+1, next, res)
    }
    (*board)[i][j] = char
}

type Trie struct {
    Next map[byte]*Trie
    Word string
}

func (t *Trie) insert(word string) {
	if len(word) == 0 {
		return
	}
    node := t
    for i := range word {
        if _, ok := node.Next[word[i]]; !ok {
            node.Next[word[i]] = &Trie{Next: make(map[byte]*Trie)}
        }
        node = node.Next[word[i]]
    }
    node.Word = word
}
```

**Recap**

In word search problems, trie is a helpful data structure which can optimize time complexity.

## [242. Valid Anagram](<https://leetcode.com/problems/valid-anagram/>)

Given two strings *s* and *t* , write a function to determine if *t* is an anagram of *s*.

**Example 1:**

```
Input: s = "anagram", t = "nagaram"
Output: true
```

**Example 2:**

```
Input: s = "rat", t = "car"
Output: false
```

**Note:**
You may assume the string contains only lowercase alphabets.

**Follow up:**
What if the inputs contain unicode characters? How would you adapt your solution to such case?

**Solution**

(1) Accepted

If the inputs only contains letters, a `uint8`slice is enough.

```go
func isAnagram(s string, t string) bool {
    if len(s) != len(t) {
        return false
    }
    count := make(map[uint8]int)
    for i := range s {
        count[s[i]]++
        count[t[i]]--
    }
    for _, c := range count {
        if c != 0 {
            return false
        }
    }
    return true
}
```

- Time complexity: $$O(n)$$  where n is the length of string.
- Space complexity: $$O(n)$$

(2) Accepted

Compare sorted runes.

```go
func isAnagram(s string, t string) bool {
	if len(s) != len(t) {
		return false
	}
	sortedRunes := func(str string) []int {
		runes := make([]int, 0, len(str))
		for _, r := range str {
			runes = append(runes, int(r))
		}
		sort.Ints(runes)
		return runes
	}
	runes1, runes2 := sortedRunes(s), sortedRunes(t)
	for i := range runes1 {
		if runes1[i] != runes2[i] {
			return false
		}
	}
	return true
}
```

- Time complexity: $$O(nlogn)$$  where n is the length of string.
- Space complexity: $$O(1)$$

**Recap**

If we can solve a problem in one pass, don't do it in tow or more passes.

## [387. First Unique Character in a String](<https://leetcode.com/problems/first-unique-character-in-a-string/>)

Given a string, find the first non-repeating character in it and return it's index. If it doesn't exist, return -1.

**Examples:**

```
s = "leetcode"
return 0.

s = "loveleetcode",
return 2.
```

**Solution**

(1) Accepted

```go
func firstUniqChar(s string) int {
    if len(s) == 0 {
        return -1
    }
    unique := make(map[rune]bool) // If a character is unique in the string
    for _, r := range s {
        if _, ok := unique[r]; !ok {
            unique[r] = true
        } else {
            unique[r] = false
        }
    }
    for i, r := range s {
        if unique[r] {
            return i
        }
    }
    return -1
}
```

- Time complexity: $$O(n)$$
- Space complexity: $$O(m)$$ where m is the number of distinct characters

Improvement:

Since we know there are only 26 characters at most, we can use array which is faster and smaller than map.

```go
func firstUniqChar(s string) int {
    if len(s) == 0 {
        return -1
    }
    present := make([]int, 26)
    for _, r := range s {
        if present[r-'a'] == 0 {
            present[r-'a'] = 1
        } else {
            present[r-'a'] = -1
        }
    }
    for i, r := range s {
        if present[r-'a'] == 1 {
            return i
        }
    }
    return -1
}
```

- Time complexity: $$O(n)$$
- Space complexity: $$O(1)$$

(2) Accepted

Just count the frequency of every character in string.

```go
func firstUniqChar(s string) int {
    if len(s) == 0 {
        return -1
    }
    present := make([]int, 26)
    for _, r := range s {
        present[r-'a']++
    }
    for i, r := range s {
        if present[r-'a'] == 1 {
            return i
        }
    }
    return -1
}
```

- Time complexity: $$O(n)$$
- Space complexity: $$O(1)$$

We can use `strings.Count`:

```go
func firstUniqChar(s string) int {
	if len(s) == 0 {
		return -1
	}
	for i := range s {
		if strings.Count(s, s[i:i+1]) == 1 {
			return i
		}
	}
	return -1
}
```

**Recap**

If possible, use array instead of map which is slower and bigger.

## [344. Reverse String](<https://leetcode.com/problems/reverse-string/>)

Write a function that reverses a string. The input string is given as an array of characters `char[]`.

Do not allocate extra space for another array, you must do this by **modifying the input array in-place** with O(1) extra memory.

You may assume all the characters consist of [printable ascii characters](https://en.wikipedia.org/wiki/ASCII#Printable_characters).

 

**Example 1:**

```
Input: ["h","e","l","l","o"]
Output: ["o","l","l","e","h"]
```

**Example 2:**

```
Input: ["H","a","n","n","a","h"]
Output: ["h","a","n","n","a","H"]
```

**Solution**

```go
func reverseString(s []byte)  {
    if len(s) == 0 {
        return
    } 
    for i, j := 0, len(s)-1; i < j; i, j = i+1, j-1 {
        s[i], s[j] = s[j], s[i]
    }
}
```

- Time complexity: $$O(n)$$
- Space complexity: $$O(1)$$

# Tree

## [230. Kth Smallest Element in a BST](<https://leetcode.com/problems/kth-smallest-element-in-a-bst/>)

Given a binary search tree, write a function `kthSmallest` to find the **k**th smallest element in it.

**Note:** 
You may assume k is always valid, 1 ≤ k ≤ BST's total elements.

**Example 1:**

```
Input: root = [3,1,4,null,2], k = 1
   3
  / \
 1   4
  \
   2
Output: 1
```

**Example 2:**

```
Input: root = [5,3,6,2,4,null,null,1], k = 3
       5
      / \
     3   6
    / \
   2   4
  /
 1
Output: 3
```

**Follow up:**
What if the BST is modified (insert/delete operations) often and you need to find the kth smallest frequently? How would you optimize the kth Smallest routine?

**Solution**

(1) Accepted

In-order traversal.

````go
func kthSmallest(root *TreeNode, k int) int {
    count, res := k, 0
    inorder(root, &count, &res)
    return res
}

func inorder(root *TreeNode, count *int, res *int) {
    if root.Left != nil {
        inorder(root.Left, count, res)
    }
    // Stop right at the kth node
    if *count = *count - 1; *count == 0 {
        *res = root.Val
        return
    }
    if root.Right != nil {
        inorder(root.Right, count ,res)
    }
}
````

Also, we can do it iteratively.

```go
func kthSmallest(root *TreeNode, k int) int {
    if root = nil || k <= 0 {
        return -1
    }
    stack := make([]*TreeNode, 0)
    for p := root; p != nil; p = p.Left {
        stack = append(stack, p)
    }
    for k > 0 {
        p := stack[len(stack)-1]
        stack = stack[:len(stack)-1]
        if k--; k == 0 {
            return p.Val
        }
        // Right-child tree
        for p = p.Right; p != nil; p = p.Left {
            stack = append(stack, p)
        }
    }
    return -1
}
```

- Time complexity: $$O(n)$$ worst
- Space complexity: $$O(logn)$$

(2) Accepted

Since BST is an implicit sorted sequence, we can binary search it.

```go
func kthSmallest(root *TreeNode, k int) int {
    if count := countNodes(root.Left); k <= count {
        return kthSmallest(root.Left, k)
    } else if k > count+1 {
        // 1 is counted as current node
        return kthSmallest(root.Right, k-count-1)
    } else {
        return root.Val
    }
}

func countNodes(node *TreeNode) int {
    if node == nil {
        return 0
    }
    return 1 + countNodes(node.Left) + countNodes(node.Right)
}
```

- Time complexity: $$O(logn)$$?
- Space complexity: $$O(logn)$$?

(3) 

> What if the BST is modified (insert/delete operations) often and you need to find the kth smallest frequently?

We can store nodes' values in an array.

```go
func kthSmallest(root *TreeNode, k int) int {
    if root == nil || k <= 0 {
        return -1
    }
    vals := make([]int, 0)
    inorder(root, &vals)
    return vals[k-1]
}

func inorder(root *TreeNode, vals *[]int) {
    if root == nil {
        return
    }
    inorder(root.Left, vals)
    *vals = append(*vals, root.Val)
    inorder(root.Right, vals)
}
```

- Time complexity: $$O(n)$$
- Space complexity: $$O(n)$$

**Recap**

If we in-order traverse a BST, we will get a sorted sequence.

## [236. Lowest Common Ancestor of a Binary Tree](<https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree/>)

Given a binary tree, find the lowest common ancestor (LCA) of two given nodes in the tree.

According to the [definition of LCA on Wikipedia](https://en.wikipedia.org/wiki/Lowest_common_ancestor): “The lowest common ancestor is defined between two nodes p and q as the lowest node in T that has both p and q as descendants (where we allow **a node to be a descendant of itself**).”

Given the following binary tree:  root = [3,5,1,6,2,0,8,null,null,7,4]

![1](https://assets.leetcode.com/uploads/2018/12/14/binarytree.png)

**Example 1:**

```
Input: root = [3,5,1,6,2,0,8,null,null,7,4], p = 5, q = 1
Output: 3
Explanation: The LCA of nodes 5 and 1 is 3.
```

**Example 2:**

```
Input: root = [3,5,1,6,2,0,8,null,null,7,4], p = 5, q = 4
Output: 5
Explanation: The LCA of nodes 5 and 4 is 5, since a node can be a descendant of itself according to the LCA definition.
```

 

**Note:**

- All of the nodes' values will be unique.
- p and q are different and both values will exist in the binary tree.

**Solution**

(1) Accepted

We can convert this problem to "Find the Intersection of Two Linked Lists".

```go
/**
 * Definition for TreeNode.
 * type TreeNode struct {
 *     Val int
 *     Left *ListNode
 *     Right *ListNode
 * }
 */
 func lowestCommonAncestor(root, p, q *TreeNode) *TreeNode {
     if root == nil || p == nil || q == nil {
         return nil
     }
     // Get the paths from to p and q respectively
     pathToP, pathToQ := make([]*TreeNode, 0), make([]*TreeNode, 0)
     dfs(root, p, &pathToP)
     dfs(root, q, &pathToQ)
     // Find the fisrt common node of two paths from bottom to top
     if gap := len(pathToP)-len(pathToQ); gap > 0 {
         for ; gap > 0; gap-- {
             pathToP = pathToP[:len(pathToP)-1]
         }
     } else if gap < 0 {
         for ; gap < 0; gap++ {
             pathToQ = pathToQ[:len(pathToQ)-1]
         }
     }
     for len(pathToP) > 0 && len(pathToQ) > 0 {
         if pathToP[len(pathToP)-1] == pathToQ[len(pathToQ)-1] {
             return pathToP[len(pathToP)-1]
         }
         pathToP, pathToQ = pathToP[:len(pathToP)-1], pathToQ[:len(pathToQ)-1]
     }
     return root
}

func dfs(cur *TreeNode, target *TreeNode, path *[]*TreeNode) bool {
    if cur == nil {
        return false
    }
    *path = append(*path, cur)
    if cur == target {
        return true
    }
    if dfs(cur.Left, target, path) {
        return true
    } else if dfs(cur.Right, target, path) {
        return true
    } else {
        *path = (*path)[:len(*path)-1]
        return false
    }
}
```

- Time complexity: $$O(n)$$
- Space complexity: $$O(logn)$$

(2) Accepted

If the current tree/subtree contains both p and q, then the function result is their LCA. If only one of them is in that subtree, then the result is that one of them. If neither are in that subtree, the result is null.

```go
func lowestCommonAncestor(root, p, q *TreeNode) *TreeNode {
    if root == nil || root == p || root == q {
        return root
    }
    left, right := lowestCommonAncestor(root.Left, p, q), lowestCommonAncestor(root.Right, p, q)
    if left == nil {
        // Both p and q are in the right subtree
        return right
    } else if right == nil {
        // Both p and q are in the left substree
        return left
    }
    // p and q are in defferent subtrees
    return root
}
```

- Time complexity: $$O(n)$$
- Space complexity: $$O(n)$$

(3) Accepted

To find the lowest common ancestor, we need to find where is `p` and `q` and a way to track their ancestors. A `parent` pointer for each node found is good for the job. After we found both `p` and `q`, we create a set of `p`'s `ancestors`. Then we travel through `q`'s `ancestors`, the first one appears in `p`'s is our answer.

```go
func lowestCommonAncestor(root, p, q *TreeNode) *TreeNode {
    if root == nil || p == nil || q == nil {
        return root
    }
    parent := map[*TreeNode]*TreeNode{root: nil}
    stack := []*TreeNode{root}
    for {
        _, ok1 := parent[p]
        _, ok2 := parent[q]
        if ok1 && ok2 {
            break
        }
        node := stack[0]
        stack = stack[1:]
        if node.Left != nil {
            parent[node.Left], stack = node, append(stack, node.Left)
        }
        if node.Right != nil {
            parent[node.Right], stack = node, append(stack, node.Right)
        }
    }
    ancestors := make(map[*TreeNode]bool)
    for p != nil {
        ancestors[p] = true
        p = parent[p]
    }
    for ; !ancestors[q]; q = parent[q] {
    }
    return q
}
```

- Time complexity: $$O(n)$$
- Space complexity: $$O(n)$$

## [297. Serialize and Deserialize Binary Tree](<https://leetcode.com/problems/serialize-and-deserialize-binary-tree/>)

Serialization is the process of converting a data structure or object into a sequence of bits so that it can be stored in a file or memory buffer, or transmitted across a network connection link to be reconstructed later in the same or another computer environment.

Design an algorithm to serialize and deserialize a binary tree. There is no restriction on how your serialization/deserialization algorithm should work. You just need to ensure that a binary tree can be serialized to a string and this string can be deserialized to the original tree structure.

**Example:** 

```
You may serialize the following tree:

    1
   / \
  2   3
     / \
    4   5

as "[1,2,3,null,null,4,5]"
```

**Clarification:** The above format is the same as [how LeetCode serializes a binary tree](https://leetcode.com/faq/#binary-tree). You do not necessarily need to follow this format, so please be creative and come up with different approaches yourself.

**Note:** Do not use class member/global/static variables to store states. Your serialize and deserialize algorithms should be stateless.

**Solution**

Just preorder traverse the tree and mark null nodes. For deserializing, we use a Queue to store the pre-order traversal and since we have "X" as null node, we know exactly where to end building subtress.

```go
func serialize(root *TreeNode) string {
    var sb strings.Builder
    buildString(root, sb)
    return sb.String()
}

func buildString(root *TreeNode, sb strings.Builder) {
    if root == nil {
        sb.WriteString("#,")
    } else {
        sb.WriteString(strconv.Itoa(root.Val)).WriteString(",")
        buildString(node.Left, sb)
        buildString(node.Right, sb)
    }
}

func deserialize(data string) *TreeNode {
    return buildTree(strings.Split(data, ","))
}

func buildTree(data []string) *TreeNode {
    s := data[0]
    data = data[1:]
    if s == "#" {
        return nil
    } 
    val, _ := strconv.Atoi(s)
    node := &TreeNode{Val: val}
    node.Left = buildTree(data)
    node.Right = buildTree(data)
    return node
}
```

- Time complexity: $$O(n)$$ for both `serialize`and `deserialize`
- Space complexity: $$O(n)$$

# Segment Tree

## [218. The Skyline Problem](<https://leetcode.com/problems/the-skyline-problem/>)

A city's skyline is the outer contour of the silhouette formed by all the buildings in that city when viewed from a distance. Now suppose you are **given the locations and height of all the buildings** as shown on a cityscape photo (Figure A), write a program to **output the skyline** formed by these buildings collectively (Figure B).

![Buildings](https://assets.leetcode.com/uploads/2018/10/22/skyline1.png) 



![Skyline Contour](https://assets.leetcode.com/uploads/2018/10/22/skyline2.png)



The geometric information of each building is represented by a triplet of integers `[Li, Ri, Hi]`, where `Li` and `Ri` are the x coordinates of the left and right edge of the ith building, respectively, and `Hi` is its height. It is guaranteed that `0 ≤ Li, Ri ≤ INT_MAX`, `0 < Hi ≤ INT_MAX`, and `Ri - Li > 0`. You may assume all buildings are perfect rectangles grounded on an absolutely flat surface at height 0.

For instance, the dimensions of all buildings in Figure A are recorded as: `[ [2 9 10], [3 7 15], [5 12 12], [15 20 10], [19 24 8] ] `.

The output is a list of "**key points**" (red dots in Figure B) in the format of `[ [x1,y1], [x2, y2], [x3, y3], ... ]`that uniquely defines a skyline. **A key point is the left endpoint of a horizontal line segment**. Note that the last key point, where the rightmost building ends, is merely used to mark the termination of the skyline, and always has zero height. Also, the ground in between any two adjacent buildings should be considered part of the skyline contour.

For instance, the skyline in Figure B should be represented as:`[ [2 10], [3 15], [7 12], [12 0], [15 10], [20 8], [24, 0] ]`.

**Notes:**

- The number of buildings in any input list is guaranteed to be in the range `[0, 10000]`.
- The input list is already sorted in ascending order by the left x position `Li`.
- The output list must be sorted by the x position.
- There must be no consecutive horizontal lines of equal height in the output skyline. For instance, `[...[2 3], [4 5], [7 5], [11 5], [12 7]...]` is not acceptable; the three lines of height 5 should be merged into one in the final output as such: `[...[2 3], [4 5], [12 7], ...]`

**Solution**

![](https://raw.githubusercontent.com/hot13399/leetcode-graphic-answer/master/218.%20The%20Skyline%20Problem.jpg)

```java
class Solution {
    public List<int[]> getSkyline(int[][] buildings) {
      int n = buildings.length;  
      int[][] points = new int[n + n][2];
      int i = 0;
      for(int[] b: buildings){
        points[i] = new int[]{b[0], b[2]}; // start point: +height
        points[i + 1] = new int[]{b[1], -b[2]}; // end point: -height
        i += 2;
      }
      
      //sort points by x coordinate
      Arrays.sort(points, (a, b)->{
        if(a[0] == b[0]){
          return b[1] - a[1]; 
        } else {
          return a[0] - b[0]; 
        }
      });
      
      // max height on top
      PriorityQueue<Integer> maxH = new PriorityQueue<Integer>((a,b)->{
        return b - a; 
      });
      maxH.offer(0);
      List<int[]> res = new ArrayList();
        
      int preH = -1;
      for(int[] p: points){
        if(p[1] > 0){
          maxH.offer(p[1]); 
        } else {
          maxH.remove(-p[1]);
        }
        
        int curH = maxH.peek();
        if(preH == curH) continue;
        preH = curH;
        res.add(new int[]{p[0], curH});
      }
      return res;
    }
}
```

- Time complexity: $$O(nlogn)$$
- Space complexity: $$O(n)$$

