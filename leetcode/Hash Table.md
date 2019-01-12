#### 1.[Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters)

Given a string, find the length of the **longest substring** without repeating characters.

**Example 1:**

```
Input: "abcabcbb"
Output: 3 
Explanation: The answer is "abc", with the length of 3. 
```

**Example 2:**

```
Input: "bbbbb"
Output: 1
Explanation: The answer is "b", with the length of 1.
```

**Example 3:**

```
Input: "pwwkew"
Output: 3
Explanation: The answer is "wke", with the length of 3. 
             Note that the answer must be a substring, "pwke" is a subsequence and not a substring.
```

**Solution**

1.Brute Force

Check all the substring one by one to see if it has no duplicate character.

```go
func lengthOfLongestSubstring(s string) int {
	length := len(s)
	max := 0
	for i := 0; i < length; i++ {
		for j := i + 1; j <= length; j++ {
			if allUnique(s, i, j) {
				if max < j-i {
					max = j - i
				}
			}
		}
	}
	return max
}

func allUnique(s string, start int, end int) bool {
	chars := make(map[uint8]bool)
	for i := start; i < end; i++ {
		if chars[s[i]] {
			return false
		} else {
			chars[s[i]] = true
		}
	}
	return true
}
```

Time complexity: $$O(n^3)$$

To verify if characters within index range $$[i, j)$$ are all unique, we need to scan all of them. Thus, it costs $$O(j - i)$$ time.

For a given `i`, the sum of time costed by each $$j \in [i+1, n]$$ is $$\sum_{i+1}^{n}O(j - i)$$

Thus, the sum of all the time consumption is:

$$O\left(\sum_{i = 0}^{n - 1}\left(\sum_{j = i + 1}^{n}(j - i)\right)\right) = O\left(\sum_{i = 0}^{n - 1}\frac{(1 + n - i)(n - i)}{2}\right) = O(n^3)$$

2.Sliding Window

A sliding window is an abstract concept commonly used in array/string problems. A window is a range of elements in the array/string which usually defined by the start and end indices, i.e. $$[i, j)$$ (left-closed, right-open). A sliding window is a window "slides" its two boundaries to the certain direction. For example, if we slide $$[i, j)$$ to the right by 1 element, then it becomes $$[i+1, j+1)$$ (left-closed, right-open).

In the naive approaches, we repeatedly check a substring to see if it has duplicate character. But it is unnecessary. If a substring $$s_{ij}$$ from index $$i$$ to $$j−1$$ is already checked to have no duplicate characters. We only need to check if $$s[j]$$ is already in the substring $$s_{ij}$$.

We use hash table to store the characters in current window $$[i, j)$$($$j=i$$ initially). The key is the character and the value is the current index of this character. Then we slide the index $$j$$ to the right. If it is not in the hash table, we slide $$j$$ further. Doing so until s[j] is already in the hash table. At this point, we found the maximum size of substrings without duplicate characters start with index $$i$$. If we do this for all $$i$$, we get our answer.

if $$s[j]$$ have a duplicate in the range $$[i, j)$$ with index $$j'$$, we don't need to increase $$i$$ little by little. We can skip all the elements in the range $$[i, j']$$ and let $$i$$ to be $$j' + 1$$ directly.

```go
func lengthOfLongestSubstring(s string) int {
	if len(s) == 0 {
		return 0
	}

	chars := make(map[rune]int)
	max := 0
	i := 0
	for i, v := range s {
		if j, ok := chars[v]; ok {
			// assure that left pointer moves forward
			if i < j+1 {
				i = j + 1
			}
		}
		chars[v] = i
		// update the length of substring
		if max < i-j+1 {
			max = i - j + 1
		}
	}
	return max
}
```

Time complexity: $$O(n)$$

#### 2.[Word Pattern](https://leetcode.com/problems/word-pattern)

Given a `pattern` and a string `str`, find if `str` follows the same pattern.

Here **follow** means a full match, such that there is a bijection between a letter in `pattern`and a **non-empty** word in `str`.

**Example 1:**

```
Input: pattern = "abba", str = "dog cat cat dog"
Output: true
```

**Example 2:**

```
Input:pattern = "abba", str = "dog cat cat fish"
Output: false
```

**Example 3:**

```
Input: pattern = "aaaa", str = "dog cat cat dog"
Output: false
```

**Example 4:**

```
Input: pattern = "abba", str = "dog dog dog dog"
Output: false
```

**Notes:**
You may assume `pattern` contains only lowercase letters, and `str` contains lowercase letters separated by a single space.

**Solution**

```go
func wordPattern(pattern string, str string) bool {
   words := strings.Split(str, " ")
   if len(words) != len(pattern) {
      return false
   }
   // go through the pattern letters and words in parallel
   // compare the indices where they appeared last time
   patternIndex := make(map[uint8]int)
   wordIndex := make(map[string]int)
   for i := 0; i < len(words); i++ {
      i1,ok1 := patternIndex[pattern[i]]
      i2,ok2 := wordIndex[words[i]]
      if ok1 == ok2 {
         if i1 != i2 {
            return false
         }
         patternIndex[pattern[i]] = i
         wordIndex[words[i]] = i
      } else {
         return false
      }
   }
   return true
}
```

Time complexity: $$O(n)$$, n is the length of `str`.

#### 3.[Daily Temperatures](https://leetcode.com/problems/daily-temperatures/)

Given a list of daily temperatures `T`, return a list such that, for each day in the input, tells you how many days you would have to wait until a warmer temperature. If there is no future day for which this is possible, put `0` instead.

For example, given the list of temperatures `T = [73, 74, 75, 71, 69, 72, 76, 73]`, your output should be `[1, 1, 4, 2, 1, 1, 0, 0]`.

**Note:** The length of `temperatures` will be in the range `[1, 30000]`. Each temperature will be an integer in the range `[30, 100]`.

(1) hash table

The range of temperature is quite small, so it is possible to have a hash map of temperatures to earliest days when that temperature occurred.

We iterate through the list of temperatures from the back, and for each day, loop through higher temperatures and find the minimum day for existing higher temperatures.

Example, for the input, when we are at 72

```
[73, 74, 75, 71, 69, 72, 76, 73]
                      ^
# We have the following hash map:
{
  73: 7,
  76: 6,
}
```

```java
class Solution {
    public int[] dailyTemperatures(int[] T) {
        Map<Integer, Integer> temps = new HashMap<>();
        int[] res = new int[T.length];
        // iterate the array from back to front
        for (int i = T.length - 1; i > -1; i--) {
            int t = T[i];
            // use a list to store the distances between current and higher temperatures
            List<Integer> days = new ArrayList<>();
            for (int higher = t + 1; higher <= 100; higher++) {
                if (temps.containsKey(higher)) {
                    days.add(temps.get(higher) - i);
                }
            }
            if (!days.isEmpty()) {
                res[i] = days.stream().min(Comparator.naturalOrder()).get();
            }
            // update the closet position where current temperature appears
            temps.put(t, i);
        }
        return res;
    }
}
```

Time complexity: $$O(n)$$, n is the number of temperatures.

(2) stack

```java
public int[] dailyTemperatures(int[] temperatures) {
    Stack<Integer> stack = new Stack<>();
    int[] ret = new int[temperatures.length];
    for(int i = 0; i < temperatures.length; i++) {
        while(!stack.isEmpty() && temperatures[i] > temperatures[stack.peek()]) {
            int idx = stack.pop();
            ret[idx] = i - idx;
        }
        stack.push(i);
    }
    return ret;
}
```

Time complexity: $$O(n)$$, n is the number of temperatures.

Since popping and pushing is time-consuming, we can use an array to stimulate stack.

```java
public int[] dailyTemperatures(int[] temperatures) {
    int[] stack = new int[temperatures.length];
    int top = -1;
    int[] ret = new int[temperatures.length];
    for(int i = 0; i < temperatures.length; i++) {
        while(top > -1 && temperatures[i] > temperatures[stack[top]]) {
            int idx = stack[top--];
            ret[idx] = i - idx;
        }
        stack[++top] = i;
    }
    return ret;
}
```

#### 4. [Array of Doubled Pairs](https://leetcode.com/problems/array-of-doubled-pairs/)

Given an array of integers `A` with even length, return `true` if and only if it is possible to reorder it such that `A[2 * i + 1] = 2 * A[2 * i]` for every `0 <= i < len(A) / 2`.

**Example 1:**

```
Input: [3,1,3,6]
Output: false
```

**Example 2:**

```
Input: [2,1,2,6]
Output: false
```

**Example 3:**

```
Input: [4,-2,2,-4]
Output: true
Explanation: We can take two groups, [-2,-4] and [2,4] to form [-2,-4,2,4] or [2,4,-2,-4].
```

**Example 4:**

```
Input: [1,2,4,16,8,4]
Output: false
```

**Note:**

1. `0 <= A.length <= 30000`
2. `A.length` is even
3. `-100000 <= A[i] <= 100000`

**Solution**

**Let's see a simple case**
Assume all interger are positive, for example `[2,4,4,8]`.
We have one `x = 2`, we need to match it with one `2x = 4`.
Then one `4` is gone, we have the other `x = 4`.
We need to match it with one `2x = 8`.
Finally no number left.

**Why we start from 2?**
Because it's the smallest and we no there is no `x/2` left.
So we know we need to find `2x`

**What if the case negative?**
One way is that start from the biggest (with smallest absolute value),
and we apply the same logic.

Another way is that start from the smallest (with biggest absolute value ), and we try to find `x/2` each turn.

```go
func canReorderDoubled(A []int) bool {
	counter := make(map[int]int)
	for _, val := range A {
		counter[val]++
	}
	// we need to traverse the map
	// from the smallest key to the biggest key
	keys := make([]int, 0)
	for k := range counter {
		keys = append(keys, k)
	}
	sort.Ints(keys)

	for _, k := range keys {
        // we have completed matching a number
		if counter[k] == 0 {
			continue
		}
		var want int
		if k < 0 {
			// if k is negative then k is A[2*i+1]
			want = k / 2
		} else {
            // if k is positive then k is A[2*i]
			want = k * 2
		}
		if (k < 0 && k%2 != 0) || counter[k] > counter[want] {
			return false
		}
		counter[want] = counter[want] - counter[k]
	}
	return true
}
```

Time complexity: $$O(Max{K,N})$$, K is the number of distinct numbers and N is the number of total numbers.

 #### 5.[Binary Subarrays With Sum](https://leetcode.com/problems/binary-subarrays-with-sum)

In an array `A` of `0`s and `1`s, how many **non-empty** subarrays have sum `S`?

**Example 1:**

```
Input: A = [1,0,1,0,1], S = 2
Output: 4
Explanation: 
The 4 subarrays are bolded below:
[1,0,1,0,1]
[1,0,1,0,1]
[1,0,1,0,1]
[1,0,1,0,1]
```

**Note:**

1. `A.length <= 30000`
2. `0 <= S <= A.length`
3. `A[i]` is either `0` or `1`.

**Solution**

Here are two important basic ideas to go through the algorithm:

(1) Let `P[i] = A[0] + A[1] + ... + A[i-1]`. Then `P[j+1] - P[i] = A[i] + A[i+1] + ... + A[j]`, the sum of the subarray `[i, j]`. **This is a very important idea to solve this kind of problem. **

(2) Subarrays are **continuous**. 

Let's say now we arrive `A[j]`in array `A`and `P[j]>=S`, so if we can find the number of subarrays summing to `P[j]-S`then we will find the number of subarrays, **which end at index `j`**, summing to S. We can subtract `P[j+1]`by`P[i]`to get `S`. Since subarrays are continuous, extract a smaller subarray from a larger subarray then we will surely get another subarray as long as the two subarrays end at the same position. 

```go
func numSubarraysWithSum(A []int, S int) int {
    // count[i] means unitl now we know there are count[i]
    // subarrays summing to i
	psum, res, count := 0, 0, make(map[int]int)
	count[0] = 1
	for _, val := range A {
		psum += val
		if psum >= S {
			res += count[psum-S]
		}
		count[psum]++
	}
	return res
}
```

Using slice instead of map may be faster.

```go
func numSubarraysWithSum(A []int, S int) int {
    psum, res, count := 0, 0, make([]int, len(A)+1)
	count[0] = 1
	for _, val := range A {
		psum += val
		if psum >= S {
			res += count[psum-S]
		}
		count[psum]++
	}
	return res
}
```

Time complexity: $$O(n)$$, n is the length of `A`.

#### 6. [Brick Wall](https://leetcode.com/problems/brick-wall/)

There is a brick wall in front of you. The wall is rectangular and has several rows of bricks. The bricks have the same height but different width. You want to draw a vertical line from the **top** to the **bottom** and cross the **least** bricks.

The brick wall is represented by a list of rows. Each row is a list of integers representing the width of each brick in this row from left to right.

If your line go through the edge of a brick, then the brick is not considered as crossed. You need to find out how to draw the line to cross the least bricks and return the number of crossed bricks.

**You cannot draw a line just along one of the two vertical edges of the wall, in which case the line will obviously cross no bricks.**

**Example:**

```
Input: [[1,2,2,1],
        [3,1,2],
        [1,3,2],
        [2,4],
        [3,1,2],
        [1,3,1,1]]

Output: 2
```

Explanation: ![explanation](https://assets.leetcode.com/uploads/2018/10/12/brick_wall.png)

**Note:**

1. The width sum of bricks in different rows are the same and won't exceed INT_MAX.
2. The number of bricks in each row is in range [1,10,000]. The height of wall is in range [1,10,000]. Total number of bricks of the wall won't exceed 20,000.

**Solution**

This is very tricky. We want to cut from the edge of the most common location among all the levels, hence using a map to record the locations and their corresponding occurrence. Moreover, we can not just draw a line across the wall edge so we skip the last brick in every level. 

```go
func leastBricks(wall [][]int) int {
	if len(wall) == 0 {
		return 0
	}

	count := 0
	edges := make(map[int]int)
	for _, level := range wall {
		width := 0
        // skip the last brick in every level
		for i := 0; i < len(level)-1; i++ {
			width += level[i]
			edges[width]++
			if count < edges[width] {
				count = edges[width]
			}
		}
	}
	return len(wall) - count
}

```

Time complexity: $$O(n^2)$$

#### 7. [Contiguous Array](https://leetcode.com/problems/contiguous-array)

Given a binary array, find the maximum length of a contiguous subarray with equal number of 0 and 1.

**Example 1:**

```
Input: [0,1]
Output: 2
Explanation: [0, 1] is the longest contiguous subarray with equal number of 0 and 1.
```

**Example 2:**

```
Input: [0,1,0]
Output: 2
Explanation: [0, 1] (or [1, 0]) is a longest contiguous subarray with equal number of 0 and 1.
```

**Note:** The length of the given binary array will not exceed 50,000.

**Solution**

Use this problem directly using hash map and prefix sum. Change 0 in the original array to 1 then calculate the prefix sum. If `sum[0, i] == sum[0, j]`, we can know that there are equal 0 and 1 in the subarray `[i + 1, j]`. We can use a hash map to accelerate the process by mapping sum to its corresponding index.

```go
func findMaxLength(nums []int) int {
    sum2index := make(map[int]int)
    sum2index[0] = -1
    sum, res := 0, 0 
    for i, v := range nums {
        if v == 0 {
            sum--
        } else {
            sum++
        }
        if j, ok := sum2index[sum]; ok {
            if i - j > res {
                res = i - j
            }
        } else {
            sum2index[sum] = i
        }
    }
    return res
}
```

Time complexity: $$O(n)$$, n is the length of array.

#### 8. [Design Twitter](https://leetcode.com/problems/design-twitter/)

Design a simplified version of Twitter where users can post tweets, follow/unfollow another user and is able to see the 10 most recent tweets in the user's news feed. Your design should support the following methods:

1. **postTweet(userId, tweetId)**: Compose a new tweet.
2. **getNewsFeed(userId)**: Retrieve the 10 most recent tweet ids in the user's news feed. Each item in the news feed must be posted by users who the user followed or by the user herself. Tweets must be ordered from most recent to least recent.
3. **follow(followerId, followeeId)**: Follower follows a followee.
4. **unfollow(followerId, followeeId)**: Follower unfollows a followee.

**Example:**

```
Twitter twitter = new Twitter();

// User 1 posts a new tweet (id = 5).
twitter.postTweet(1, 5);

// User 1's news feed should return a list with 1 tweet id -> [5].
twitter.getNewsFeed(1);

// User 1 follows user 2.
twitter.follow(1, 2);

// User 2 posts a new tweet (id = 6).
twitter.postTweet(2, 6);

// User 1's news feed should return a list with 2 tweet ids -> [6, 5].
// Tweet id 6 should precede tweet id 5 because it is posted after tweet id 5.
twitter.getNewsFeed(1);

// User 1 unfollows user 2.
twitter.unfollow(1, 2);

// User 1's news feed should return a list with 1 tweet id -> [5],
// since user 1 is no longer following user 2.
twitter.getNewsFeed(1);
```

**Solution**

```scala
class Twitter() {

  /** Initialize your data structure here. */
  private var timer = 0

  private var tweets = Map[Int, List[(Int, Int)]]()

  private var followees = Map[Int, Set[Int]]()

  /** Compose a new tweet. */
  def postTweet(userId: Int, tweetId: Int) {
    if (tweets.contains(userId)) {
      tweets = tweets.updated(userId, (timer, tweetId) :: tweets(userId))
    } else {
      tweets = tweets.updated(userId, List((timer, tweetId)))
    }
    timer += 1
  }

  /** Retrieve the 10 most recent tweet ids in the user's news feed. Each item in the news feed must be posted by users who the user followed or by the user herself. Tweets must be ordered from most recent to least recent. */
  def getNewsFeed(userId: Int): List[Int] = {
    var tmp: List[(Int, Int)] = Nil
    if (tweets.contains(userId)) {
      tmp = tweets(userId) ::: tmp
    }
    if (followees.contains(userId)) {
      tmp = followees(userId).filter(tweets.contains(_)).flatMap(tweets(_)).toList ::: tmp
    }
    if (tmp.nonEmpty) {
      tmp.sortWith(_._1 > _._1).take(10).map(_._2)
    } else {
      Nil
    }
  }

  /** Follower follows a followee. If the operation is invalid, it should be a no-op. */
  def follow(followerId: Int, followeeId: Int) {
    if (followeeId == followerId) {
      return
    }
    if (followees.contains(followerId)) {
      followees = followees.updated(followerId, followees(followerId) + followeeId)
    } else {
      followees = followees.updated(followerId, Set(followeeId))
    }
  }

  /** Follower unfollows a followee. If the operation is invalid, it should be a no-op. */
  def unfollow(followerId: Int, followeeId: Int) {
    if (followees.contains(followerId)) {
      followees = followees.updated(followerId, followees(followerId) - followeeId)
    }
  }
}

/**
 * Your Twitter object will be instantiated and called as such:
 * var obj = new Twitter()
 * obj.postTweet(userId,tweetId)
 * var param_2 = obj.getNewsFeed(userId)
 * obj.follow(followerId,followeeId)
 * obj.unfollow(followerId,followeeId)
 */
```

