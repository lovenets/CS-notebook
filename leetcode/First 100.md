## 5. Longest Palindromic Substring

Given a string **s**, find the longest palindromic substring in **s**. You may assume that the maximum length of **s** is 1000.

**Example 1:**

```
Input: "babad"
Output: "bab"
Note: "aba" is also a valid answer.
```

**Example 2:**

```
Input: "cbbd"
Output: "bb"
```

**Solution**

A palindrome mirrors around its center so we can just expand an existing palindrome to find a longer one.

```go
func longestPalindrome(s string) string {
	if len(s) < 2 {
		return s
	}
	var longest string
	expand := func(start, end int) {
		for ; start >= 0 && end < len(s) && s[start] == s[end]; start, end = start-1, end+1 {
		}
		if length := end - start - 1; length > len(longest) {
			longest = s[start+1 : start+1+length]
		}
	}
	for i := 0; i < len(s); i++ {
		// substring with odd length
		expand(i, i)
		// substring with even length
		expand(i, i+1)
	}
	return longest
}
```

- Time complexity : $$O(n^2)$$. Since expanding a palindrome around its center could take $$O(n)$$ time, the overall complexity is $$O(n^2)$$.

- Space complexity : $$O(1)$$.

## ZigZag Conversion

The string `"PAYPALISHIRING"` is written in a zigzag pattern on a given number of rows like this: (you may want to display this pattern in a fixed font for better legibility)

```
P    A    H    N
↓   ↗↓   ↗↓   ↗↓  
A  P L  S I  I  G
↓↗   ↓↗   ↓↗ 
Y    I    R
```

And then read line by line: `"PAHNAPLSIIGYIR"`

Write the code that will take a string and make this conversion given a number of rows:

```
string convert(string s, int numRows);
```

**Example 1:**

```
Input: s = "PAYPALISHIRING", numRows = 3
Output: "PAHNAPLSIIGYIR"
```

**Example 2:**

```
Input: s = "PAYPALISHIRING", numRows = 4
Output: "PINALSIGYAHRPI"
Explanation:

P     I    N
A   L S  I G
Y A   H R
P     I
```

**Solution**

```go
func convert(s string, numRows int) string {
   if s == "" || numRows < 0 {
      return ""
   }
   // Be careful with this corner case
   if numRows == 1 {
      return s
   }
   builders := make([]strings.Builder, numRows) // builder of each row
   curRow, down := -1, true
   for _, r := range s {
      if down {
         // top-down
         if curRow++; curRow == numRows-1 {
            down = false
         }
      } else {
         // bottom-up
         if curRow--; curRow == 0 {
            down = true
         }
      }
      builders[curRow].WriteRune(r)
   }
   var res strings.Builder
   for i := range builders {
      res.WriteString(builders[i].String())
   }
   return res.String()
}
```

- Time complexity : $$O(n)$$.
- Space complexity : $$O(n)$$.

## 7. Reverse Integer

Given a 32-bit signed integer, reverse digits of an integer.

**Example 1:**

```
Input: 123
Output: 321
```

**Example 2:**

```
Input: -123
Output: -321
```

**Example 3:**

```
Input: 120
Output: 21
```

**Note:**
Assume we are dealing with an environment which could only store integers within the 32-bit signed integer range: [$$−2^{31}$$,  $$2^{31}$$ − 1]. For the purpose of this problem, assume that your function returns 0 when the reversed integer overflows.

**Solution**

```go
func reverse(x int) int {
    // Be careful that int in Go is at least 32-bit.
	reversed := 0
	for x != 0 {
		lastDigit := x % 10
		if tmp := reversed*10 + lastDigit; tmp > math.MaxInt32 || tmp < math.MinInt32 {
			return 0
		} else {
			reversed = tmp
		}
		x /= 10
	}
	return reversed
}
```

- Time complexity: $$O(n)$$
- Space complexity: $$O(1)$$

## 8. String to Integer (atoi)

Implement `atoi` which converts a string to an integer.

The function first discards as many whitespace characters as necessary until the first non-whitespace character is found. Then, starting from this character, takes an optional initial plus or minus sign followed by as many numerical digits as possible, and interprets them as a numerical value.

The string can contain additional characters after those that form the integral number, which are ignored and have no effect on the behavior of this function.

If the first sequence of non-whitespace characters in str is not a valid integral number, or if no such sequence exists because either str is empty or it contains only whitespace characters, no conversion is performed.

If no valid conversion could be performed, a zero value is returned.

**Note:**

- Only the space character `' '` is considered as whitespace character.
- Assume we are dealing with an environment which could only store integers within the 32-bit signed integer range: [$$−2^{31}$$,  $$2^{31} − 1$$]. If the numerical value is out of the range of representable values, $$2^{31} − 1$$ or $$−2^{31}$$ is returned.

**Example 1:**

```
Input: "42"
Output: 42
```

**Example 2:**

```
Input: "   -42"
Output: -42
Explanation: The first non-whitespace character is '-', which is the minus sign.
             Then take as many numerical digits as possible, which gets 42.
```

**Example 3:**

```
Input: "4193 with words"
Output: 4193
Explanation: Conversion stops at digit '3' as the next character is not a numerical digit.
```

**Example 4:**

```
Input: "words and 987"
Output: 0
Explanation: The first non-whitespace character is 'w', which is not a numerical 
             digit or a +/- sign. Therefore no valid conversion could be performed.
```

**Example 5:**

```
Input: "-91283472332"
Output: -2147483648
Explanation: The number "-91283472332" is out of the range of a 32-bit signed integer.
             Thefore INT_MIN (−2^31) is returned.
```

**Solution**

```go
func myAtoi(str string) int {
	// trim whitespace characters
	str = strings.TrimSpace(str)
	if str == "" {
		// Input string only contains whitespace characters or is empty
		return 0
	}
	if !unicode.IsDigit(rune(str[0])) && str[0] != '+' && str[0] != '-' {
		// The first character is not a digit character, plus symbol or minus symbol
		return 0
	}
	start := 0
	// If the first character is plus or minus symbol
	symbol := 1
	if str[start] == '-' {
		symbol, start = -1, start+1
	} else if str[start] == '+' {
		symbol, start = 1, start+1
	}
	// Retrieve as many digit characters as possible
	// Trim leading 0s at first
	digits := make([]rune, 0)
	for _, r := range strings.TrimLeft(str[start:], "0") {
		if unicode.IsDigit(r) {
			digits = append(digits, r)
		} else {
			break
		}
	}
	// Construct the number
	res, magnitude := 0, 1
	for i := len(digits) - 1; i >= 0; i-- {
		d := int(digits[i]-'0') * symbol
		// Be careful: the order of magnitude must be not greater than 10^9 
		if tmp := res + magnitude*d; float64(magnitude) <= math.Pow10(9) && tmp >= math.MinInt32 && tmp <= math.MaxInt32 {
			res, magnitude = tmp, magnitude*10
		} else {
			// Overflow
			if symbol == 1 {
				return math.MaxInt32
			} else {
				return math.MinInt32
			}
		}
	}
	return res
}
```

- Time complexity: $$O(n)$$
- Space complexity: $$O(n)$$

## 9. [Palindrome Number](<https://leetcode.com/problems/palindrome-number/>)

Determine whether an integer is a palindrome. An integer is a palindrome when it reads the same backward as forward.

**Example 1:**

```
Input: 121
Output: true
```

**Example 2:**

```
Input: -121
Output: false
Explanation: From left to right, it reads -121. From right to left, it becomes 121-. Therefore it is not a palindrome.
```

**Example 3:**

```
Input: 10
Output: false
Explanation: Reads 01 from right to left. Therefore it is not a palindrome.
```

**Follow up:**

Could you solve it without converting the integer to a string?

**Solution**

```go
func isPalindrome(x int) bool {
	// Solve this problem without converting number to string
	if x < 0 || (x%10 == 0 && x != 0) {
		return false
	}
	origin, reverse := x, 0
	for x != 0 {
		reverse, x = reverse*10+x%10, x/10
	}
	return origin == reverse
}
```

- Time complexity: $$O(n)$$
- Space complexity: $$O(1)$$

## 10. [Regular Expression Matching](<https://leetcode.com/problems/regular-expression-matching/>)

Given an input string (`s`) and a pattern (`p`), implement regular expression matching with support for `'.'` and `'*'`.

```
'.' Matches any single character.
'*' Matches zero or more of the preceding element.
```

The matching should cover the **entire** input string (not partial).

**Note:**

- `s` could be empty and contains only lowercase letters `a-z`.
- `p` could be empty and contains only lowercase letters `a-z`, and characters like `.` or `*`.

**Example 1:**

```
Input:
s = "aa"
p = "a"
Output: false
Explanation: "a" does not match the entire string "aa".
```

**Example 2:**

```
Input:
s = "aa"
p = "a*"
Output: true
Explanation: '*' means zero or more of the precedeng element, 'a'. Therefore, by repeating 'a' once, it becomes "aa".
```

**Example 3:**

```
Input:
s = "ab"
p = ".*"
Output: true
Explanation: ".*" means "zero or more (*) of any character (.)".
```

**Example 4:**

```
Input:
s = "aab"
p = "c*a*b"
Output: true
Explanation: c can be repeated 0 times, a can be repeated 1 time. Therefore it matches "aab".
```

**Example 5:**

```
Input:
s = "mississippi"
p = "mis*is*p*."
Output: false
```

**Solution**

[explanation](<https://www.youtube.com/watch?v=l3hda49XcDE&index=17&list=PLrmLmBdmIlpsHaNTPP_jHHDx_os9ItYXr>)

As the problem has an **optimal substructure**, it is natural to cache intermediate results. We ask the question `dp[i][j]`: does `s[:i]` and `p[:j]`match?

- If `s[i]==p[j] || p[j]=='.'`, `dp[i][j]=dp[i-1][j-1]`
- If `p[j]=='*'`, `dp[i][j]=dp[i][j-2]`; and if `s[i]==p[j-1] || p[j-1]=='.'`, then `dp[i][j]=dp[i-1][j ]`
- Else, `dp[i][j]=false`

```go
func isMatch(s string, p string) bool {
   dp := make([][]bool, len(s)+1)
   for i := range dp {
      dp[i] = make([]bool, len(p)+1)
   }
   // Deals with patterns like a* or a*b* or a*b*c*
   dp[0][0] = true
   for i := 1; i < len(dp[0]); i++ {
      if p[i-1] == '*' {
         dp[0][i] = dp[0][i-2]
      }
   }
   for i := 1; i < len(dp); i++ {
      for j := 1; j < len(dp[0]); j++ {
         if p[j-1] == '.' || p[j-1] == s[i-1] {
            dp[i][j] = dp[i-1][j-1]
         } else if p[j-1] == '*' {
            dp[i][j] = dp[i][j-2]
            if p[j-2] == '.' || p[j-2] == s[i-1] {
               dp[i][j] = dp[i][j] || dp[i-1][j]
            }
         } else {
            dp[i][j] = false
         }
      }
   }
   return dp[len(s)][len(p)]
}
```

- Time complexity: $$O(mn)$$ where m is the length of `s`and n is the length of `p`
- Space complexity: $$O(mn)$$