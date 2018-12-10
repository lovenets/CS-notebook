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

