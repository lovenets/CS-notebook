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

[Fisher-Yates Algorithm]([https://en.wikipedia.org/wiki/Fisher%E2%80%93Yates_shuffle#The_modern_algorithm](https://en.wikipedia.org/wiki/Fisher–Yates_shuffle#The_modern_algorithm))

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

1. `sort.Ints`will return the index where target **should be** in array.
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
- `right[i]=nums[len(nums)-1]*...*nums[i+1]`

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





