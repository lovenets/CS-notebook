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

