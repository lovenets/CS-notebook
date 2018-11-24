#### 1. [Valid Parentheses](https://leetcode.com/problems/valid-parentheses/)

Given a string containing just the characters `'('`, `')'`, `'{'`, `'}'`, `'['` and `']'`, determine if the input string is valid.

An input string is valid if:

1. Open brackets must be closed by the same type of brackets.
2. Open brackets must be closed in the correct order.

Note that an empty string is also considered valid.

**Example 1:**

```
Input: "()"
Output: true
```

**Example 2:**

```
Input: "()[]{}"
Output: true
```

**Example 3:**

```
Input: "(]"
Output: false
```

**Example 4:**

```
Input: "([)]"
Output: false
```

**Example 5:**

```
Input: "{[]}"
Output: true
```

**My Solution**

```go
func isValid(s string) bool {
	if len(s) == 1 {
		return false
	}

	if len(s) == 0 {
		return true
	}

	left := make([]string,0)
	parenthese := map[string]string {
		"(" : ")",
		"{" : "}",
		"[" : "]",
	}

	for index,ch := range s {
		str := string(ch)

		if str == "(" || str == "{" || str == "[" {
			// when we find a left parenthesis, push it into a stack
			left = append(left,str)
		} else {
			// If a right parenthesis is the first character
			// in the string, then return false directly.
			if index == 0 {
				return false
			}
            
			// When we find a right parenthesis:
			// If the stack is empty, return false.
			if len(left) == 0 {
				return false
			}
			// If the stack is not empty,
			// pop the stack and verify if the left parenthesis
			// is corresponding to the right parenthesis.
			if parenthese[left[len(left) - 1]] != str {
				return false
			}
			// delete the verified element
			left = append(left[:len(left) - 1],left[len(left):]...)
		}
	}
    
    return len(left) == 0
}
```

Time Complexity: $O(n)$, n is the length of input string

**Improvement**

Maybe there is a more direct and elegant solution(C++):

```c++
    bool isValid(const string &s) {
        stack<int> st;
        
        for (int i=0; i<s.length(); i++) {
            switch(s[i]) {
                case '(':
                    st.push(1);
                    break;
                case '[':
                    st.push(2);
                    break;
                case '{':
                    st.push(3);
                    break;
                case ')':
                    if (st.empty() || st.top() != 1) return false;
                    st.pop();
                    break;
                case ']':
                    if (st.empty() || st.top() != 2) return false;
                    st.pop();
                    break;
                case '}':
                    if (st.empty() || st.top() != 3) return false;
                    st.pop();
                    break;
            }
        }
        return st.empty();
    }
```

#### 2. [Verify Preorder Serialization of a Binary Tree](https://leetcode.com/problems/verify-preorder-serialization-of-a-binary-tree/)

One way to serialize a binary tree is to use pre-order traversal. When we encounter a non-null node, we record the node's value. If it is a null node, we record using a sentinel value such as `#`.

```
     _9_
    /   \
   3     2
  / \   / \
 4   1  #  6
/ \ / \   / \
# # # #   # #
```

For example, the above binary tree can be serialized to the string `"9,3,4,#,#,1,#,#,2,#,6,#,#"`, where `#`represents a null node.

Given a string of comma separated values, verify whether it is a correct preorder traversal serialization of a binary tree. Find an algorithm **without reconstructing the tree**.

Each comma separated value in the string must be either an integer or a character `'#'`representing `null` pointer.

You may assume that the input format is always valid, for example it could never contain two consecutive commas such as `"1,,3"`.

**Example 1:**

```
Input: "9,3,4,#,#,1,#,#,2,#,6,#,#"
Output: true
```

**Example 2:**

```
Input: "1,#"
Output: false
```

**Example 3:**

```
Input: "9,#,#,1"
Output: false
```

**Solution**

(1) Using stack

The key here is, when you see two consecutive "#" characters on stack, pop both of them and replace the top element on the stack with "#". For example,

```
preorder = 1,2,3,#,#,#,#

Pass 1: stack = [1]

Pass 2: stack = [1,2]

Pass 3: stack = [1,2,3]

Pass 4: stack = [1,2,3,#]

Pass 5: stack = [1,2,3,#,#] -> two #s on top so pop them and replace top with #. -> stack = [1,2,#]

Pass 6: stack = [1,2,#,#] -> two #s on top so pop them and replace top with #. -> stack = [1,#]

Pass 7: stack = [1,#,#] -> two #s on top so pop them and replace top with #. -> stack = [#]

```

If there is only one # on stack at the end of the string then return true else return false.

The point is when you find two consecutive "#" on the top of stack, that means you have constructed a child tree whose children are two "#" and root is the element just before two  "#" on the stack. Since a binary tree consists of child trees recursively, if there is only a "#" on the stack at last, the "#" represents just the whole tree.

```go
// using stack
func isValidSerialization(preorder string) bool {
	nodes := strings.Split(preorder, ",")
	stack := make([]string, 0)

	for _, node := range nodes {
		for len(stack) > 0 && node == "#" && stack[len(stack)-1] == "#" {
			// pop stack
			stack = append(stack[:len(stack)-1], stack[len(stack):]...)
			// For efficiency, we can check if stack is empty firstly.
			if len(stack) == 0 {
				return false
			}
			// pop stack
			stack = append(stack[:len(stack)-1], stack[len(stack):]...)
		}
		// node is #
		stack = append(stack, node)
	}

	return len(stack) == 1 && stack[0] == "#"
}
```

Time Complexity: $O(n^2)$, n is the sum of nodes

(2) Not Using Stack

Obviously, in a binary tree, a node except for the root consumes an edge which points to its father and a not null node generates two edges which point to its children. Therefore, the sum of edges should not be negative. 

```go
// Borrowed from 7 Lines Easy Java Solution
func isValidSerialization(preorder string) bool {
	nodes := strings.Split(preorder, ",")

	// sum of edges
	edges := 1
	for _, node := range nodes {
		// every node should consume an edge
		edges--

		// to prevent the case: #,a,...
		if edges < 0 {
			return false
		}

		// a not null node should generate two edges
		if node != "#" {
			edges += 2
		}
	}

	// the sum of a binary tree should not be negative
	return edges == 0
}
```

Time Complexity: $O(n)$, n is the sum of nodes

#### 3. [Sum of Subarrays Minimums](https://leetcode.com/problems/sum-of-subarray-minimums/)

Given an array of integers `A`, find the sum of `min(B)`, where `B`ranges over every (contiguous) subarray of `A`.

Since the answer may be large, **return the answer modulo 10^9 + 7.**

**Example 1:**

```
Input: [3,1,2,4]
Output: 17
Explanation: Subarrays are [3], [1], [2], [4], [3,1], [1,2], [2,4], [3,1,2], [1,2,4], [3,1,2,4]. 
Minimums are 3, 1, 2, 4, 1, 1, 2, 1, 1, 1.  Sum is 17.
```

**Solution**

Here is the [explanation link](https://leetcode.com/problems/sum-of-subarray-minimums/discuss/178876/stack-solution-with-very-detailed-explanation-step-by-step).

Before diving into the solution, we first introduce a very important stack type, which is called **monotone stack** .

**What is monotonous increase stack?**

Roughly speaking, the elements in the an monotonous increase stack keeps an increasing order.

**The typical paradigm for monotonous increase stack**:

```
for(int i = 0; i < A.size(); i++){
  while(!in_stk.empty() && in_stk.top() > A[i]){
    in_stk.pop();
  }
  in_stk.push(A[i]);
}
```

**What can monotonous increase stack do?**

(1) find the **previous less** element of each element in a vector **with O(n) time**:

- What is the previous less element of an element?
  For example:
  [3, 7, 8, 4]
  The previous less element of 7 is 3.
  The previous less element of 8 is 7.
  **The previous less element of 4 is 3**.
  There is no previous less element for 3.

For simplicity of notation, we use abbreviation **PLE** to denote **P**revious **L**ess **E**lement.

- C++ code (by slightlymodifying the paradigm):
  Instead of directly pushing the element itself, here for simplicity, we push the **index**.
  We do some record when the index is pushed into the stack.

```c++
// previous_less[i] = j means A[j] is the previous less element of A[i].
// previous_less[i] = -1 means there is no previous less element of A[i].
vector<int> previous_less(A.size(), -1);
for(int i = 0; i < A.size(); i++){
  while(!in_stk.empty() && A[in_stk.top()] > A[i]){
    in_stk.pop();
  }
  previous_less[i] = in_stk.empty()? -1: in_stk.top();
  in_stk.push(i);
}
```

(2) find the **next less** element of each element in a vector with **O(n) time**:

- What is the next less element of an element?
  For example:
  [3, 7, 8, 4]
  The next less element of 8 is 4.
  **The next less element of 7 is 4**.
  There is no next less element for 3 and 4.

For simplicity of notation, we use abbreviation **NLE** to denote **N**ext **L**ess **E**lement.

- C++ code (by slighly modifying the paradigm):
  We do some record when the index is poped out from the stack.

```c++
// next_less[i] = j means A[j] is the next less element of A[i].
// next_less[i] = -1 means there is no next less element of A[i].
vector<int> previous_less(A.size(), -1);
for(int i = 0; i < A.size(); i++){
  while(!in_stk.empty() && A[in_stk.top()] > A[i]){
    auto x = in_stk.top(); in_stk.pop();
    next_less[x] = i;
  }
  in_stk.push(i);
}
```

**How can the monotonous increase stack be applied to this problem?**

For example:
Consider the element `3` in the following vector:

```
                            [2, 9, 7, 8, 3, 4, 6, 1]
			     |                    |
	             the previous less       the next less 
	                element of 3          element of 3
```

After finding both **NLE** and **PLE** of `3`, we can determine the
distance between `3` and `2`(previous less) , and the distance between `3` and `1`(next less).
In this example, the distance is `4` and `3` respectively.

**How many subarrays with 3 being its minimum value?**
The answer is `4*3`.

```
9 7 8 3 
9 7 8 3 4 
9 7 8 3 4 6 
7 8 3 
7 8 3 4 
7 8 3 4 6 
8 3 
8 3 4 
8 3 4 6 
3 
3 4 
3 4 6
```

**How much the element 3 contributes to the final answer?**
It is `3*(4*3)`.
**What is the final answer?**
Denote by `left[i]` the distance between element `A[i]` and its **PLE**.
Denote by `right[i]` the distance between element `A[i]` and its **NLE**.

The final answer is,`sum(A[i]*left[i]*right[i] )`.

```go
func sumSubarrayMins(A []int) int {
	// initialize previous less element and next less element
	// of each element in the array
	// Actually, we use a 2D array to simulate a 1D array
	// whose elements are tuples (A[i],i)
	PLE := make([][]int, 0)
	NLE := make([][]int, 0)

	// left is for the distance to PLE
	// right is for the distance to NLE
	left := make([]int, len(A))
	right := make([]int, len(A))

	// generate PLE and left
	for i := 0; i < len(A); i++ {
		for len(PLE) != 0 && PLE[len(PLE)-1][0] >= A[i] {
			PLE = append(PLE[:len(PLE)-1], PLE[len(PLE):]...)
		}
		if len(PLE) == 0 {
			// no PLE for A[i] so let left[i] be (i + 1)
			left[i] = i + 1
		} else {
			left[i] = i - PLE[len(PLE)-1][1]
		}
		PLE = append(PLE, []int{A[i], i})
	}

	// generate MLE and right
	for i := len(A) - 1; i >= 0; i-- {
		for len(NLE) != 0 && NLE[len(NLE)-1][0] > A[i] {
			NLE = append(NLE[:len(NLE)-1], NLE[len(NLE):]...)
		}
		if len(NLE) == 0 {
			// no NLE for A[i] so let right[i] be (len(A) - i)
			right[i] = len(A) - i
		} else {
			right[i] = NLE[len(NLE)-1][1] - i
		}
		NLE = append(NLE, []int{A[i], i})
	}

	ans := 0
	for i := 0; i < len(A); i++ {
		ans += A[i]*left[i]*right[i]
	}
	return ans % 1000000007
}
```

Please notice that when PLE or NLE is empty, we set `left[i] = i + 1` and `right[i] = len(A) - i`repectively. When NLE is empty, which means no previous less element for `A[i]`, we set `left[i] = i+1` by default.
For example `[7 8 4 3]`, there is no PLE for element `4`, so `left[2] = 2+1 =3`.
How many subarrays with 4(`A[2]`) being its minimum value? It's `left[2]*right[2]=3*1`.
So the default value `i+1` for `left[i]` and the default value `len(A)-i` for `right[i]` are for counting the subarrays **conveniently**.  

#### 4. [Simplify Path](https://leetcode.com/problems/simplify-path/)

Given an absolute path for a file (Unix-style), simplify it. 

For example,
**path** = `"/home/"`, => `"/home"`
**path** = `"/a/./b/../../c/"`, => `"/c"`
**path** = `"/a/../../b/../c//.//"`, => `"/c"`
**path** = `"/a//b////c/d//././/.."`, => `"/a/b/c"`

In a UNIX-style file system, a period ('.') refers to the current directory, so it can be ignored in a simplified path. Additionally, a double period ("..") moves up a directory, so it cancels out whatever the last directory was. For more information, look here: <https://en.wikipedia.org/wiki/Path_(computing)#Unix_style>

**Corner Cases:**

- Did you consider the case where **path** = `"/../"`?
  In this case, you should return `"/"`.
- Another corner case is the path might contain multiple slashes `'/'` together, such as `"/home//foo/"`.
  In this case, you should ignore redundant slashes and return `"/home/foo"`.

**My Solution**

```go
func simplifyPath(path string) string {
	if len(path) == 0 {
		return ""
	}

	// use a stack to store string except for . and ..
	stack := make([]string, 0)

	for _, s := range strings.Split(path, "/") {
		// if we find a normal string ie. a directory, push it into stack
		if ok, _ := regexp.MatchString("[a-zA-Z]+", s); ok {
			stack = append(stack, s)
		}
		// if we find a "..", pop the stack
		// because ".." cancels last directory
		if s == ".." {
			if len(stack) != 0 {
				stack = append(stack[:len(stack)-1], stack[len(stack):]...)
			}
		}
        // if we find a "...", push it into stack
        if s == "..." {
            stack = append(stack,s)
        }
	}

	var simplified string
	var builder strings.Builder
	if len(stack) == 0 {
		simplified = "/"
	} else {
		for _, s := range stack {
			builder.WriteString("/" + s)
		}
		simplified = builder.String()
	}
	return simplified
}
```

Time Complexity: $O(n)$, n is the number of valid directories in the path

Since using regular expression is a time consuming operation, drop it.

```go
func simplifyPath(path string) string {
		if len(path) == 0 {
		return ""
	}

	// use a stack to store string except for . and ..
	stack := make([]string, 0)

	for _, s := range strings.Split(path, "/") {
		// if we find a "..." or valid directory, push it
		if s != "" && s != "." && s != ".." {
			stack = append(stack, s)
		}
		// if we find a "..", pop the stack
		// because ".." cancels last directory
		if s == ".." && len(stack) != 0 {
			stack = append(stack[:len(stack)-1], stack[len(stack):]...)
		}
	}

	var simplified string
	var builder strings.Builder
	if len(stack) == 0 {
		simplified = "/"
	} else {
		for _, s := range stack {
			builder.WriteString("/" + s)
		}
		simplified = builder.String()
	}
	return simplified
}
```

#### 5. [Score of Parentheses](https://leetcode.com/problems/score-of-parentheses/)

Given a balanced parentheses string `S`, compute the score of the string based on the following rule:

- `()` has score 1
- `AB` has score `A + B`, where A and B are balanced parentheses strings.
- `(A)` has score `2 * A`, where A is a balanced parentheses string.

**Example 1:**

```
Input: "()"
Output: 1
```

**Example 2:**

```
Input: "(())"
Output: 2
```

**Example 3:**

```
Input: "()()"
Output: 2
```

**Example 4:**

```
Input: "(()(()))"
Output: 6
```

**Solution**

```go
func scoreOfParentheses(S string) int {
	// since S.length <= 50, initialize the length og stack
	stack := make([]int, 50/2+1)

	for _, val := range S {
		s := string(val)
		if s == "(" {
			stack = append(stack, -1)
		} else {
			cur := 0
			// when we find a ")", pop the stack until the top is not -1
			for stack[len(stack)-1] != -1 {
				cur += stack[len(stack)-1]
				stack = append(stack[:len(stack)-1], stack[len(stack):]...)
			}
			stack = append(stack[:len(stack)-1], stack[len(stack):]...)

			if cur == 0 {
				// we find a "()"
				stack = append(stack, 1)
			} else {
				// we find nested "()" 
				stack = append(stack, cur*2)
			}
		}
	}
	sum := 0
	for len(stack) != 0 {
		sum += stack[len(stack)-1]
		stack = append(stack[:len(stack)-1], stack[len(stack):]...)
	}
	return sum
}
```

Time Complexity: $O(n)$, n is the length of `S`.

#### 6.[Remove K Digits](https://leetcode.com/problems/remove-k-digits/)

Given a non-negative integer *num* represented as a string, remove *k*digits from the number so that the new number is the smallest possible.

**Note:**

- The length of *num* is less than 10002 and will be ≥ *k*.
- The given *num* does not contain any leading zero.

**Example 1:**

```
Input: num = "1432219", k = 3
Output: "1219"
Explanation: Remove the three digits 4, 3, and 2 to form the new number 1219 which is the smallest.
```

**Example 2:**

```
Input: num = "10200", k = 1
Output: "200"
Explanation: Remove the leading 1 and the number is 200. Note that the output must not contain leading zeroes.
```

**Example 3:**

```
Input: num = "10", k = 2
Output: "0"
Explanation: Remove all the digits from the number and it is left with nothing which is 0.
```

**Solution**

Since we can't change the orders of digits, we just need to carry on finding a less digit until we get enough digits.

```go
func removeKdigits(num string, k int) string {
   if len(num) == k {
      return "0"
   }

   stack := make([]string, 0)
   i := 0
   for i < len(num) {
      digit, _ := strconv.Atoi(string(num[i]))
      // when we find a less digit, pop the stack
      for k > 0 && len(stack) != 0 {
         top, _ := strconv.Atoi(string(stack[len(stack)-1]))
         if digit < top {
            stack = append(stack[:len(stack)-1], stack[len(stack):]...)
            k--
         } else {
            break
         }
      }
      stack = append(stack, string(num[i]))
      i++
   }

   // what if input number is like "111"
   for k > 0 {
      stack = append(stack[:len(stack)-1], stack[len(stack):]...)
      k--
   }

   // delete all leading "0"
   for len(stack) > 1 && string(stack[0]) == "0" {
      stack = append(stack[:0], stack[1:]...)
   }
   var b strings.Builder
   for _, s := range stack {
      b.WriteString(s)
   }
   return b.String()
}
```

Time Complexity: $O(n)$, n is the length of `num`.

#### 7. [Online Stock Span](https://leetcode.com/problems/online-stock-span)

Write a class `StockSpanner` which collects daily price quotes for some stock, and returns the *span* of that stock's price for the current day.

The span of the stock's price today is defined as the maximum number of **consecutive** days (**starting from today** and going backwards) for which the price of the stock was less than or equal to today's price.

For example, if the price of a stock over the next 7 days were `[100, 80, 60, 70, 60, 75, 85]`, then the stock spans would be `[1, 1, 1, 2, 1, 4, 6]`.

**Example 1:**

```
Input: ["StockSpanner","next","next","next","next","next","next","next"], [[],[100],[80],[60],[70],[60],[75],[85]]
Output: [null,1,1,1,2,1,4,6]
Explanation: 
First, S = StockSpanner() is initialized.  Then:
S.next(100) is called and returns 1,
S.next(80) is called and returns 1,
S.next(60) is called and returns 1,
S.next(70) is called and returns 2,
S.next(60) is called and returns 1,
S.next(75) is called and returns 4,
S.next(85) is called and returns 6.

Note that (for example) S.next(75) returned 4, because the last 4 prices
(including today's price of 75) were less than or equal to today's price.
```

**Note:**

1. Calls to `StockSpanner.next(int price)` will have `1 <= price <= 10^5`.
2. There will be at most `10000` calls to `StockSpanner.next` per test case.
3. There will be at most `150000` calls to `StockSpanner.next`across all test cases.
4. The total time limit for this problem has been reduced by 75% for C++, and 50% for all other languages.

**My Solution**

```go
type StockSpanner struct {
	Prices []int
}

func Constructor() StockSpanner {
	return StockSpanner{Prices: make([]int, 0)}
}

// linear scan
func (this *StockSpanner) Next(price int) int {
	this.Prices = append(this.Prices, price)
	days := 0
	if len(this.Prices) > 1 {
		count := 0
		for i := len(this.Prices) - 1; i >= 0; i-- {
			if this.Prices[i] <= price {
				count++
			} else {
				days = count
				break
			}
		}
		// corner case: [null,1,2,3,4,5]
		if days == 0 {
			days = count
		}
	} else {
		// the first input
		days = 1
	}
	return days
}
```

Time Complexity: $O(n)$, n is the number of prices which have been input so far.

**Improvement**

If we can store the previous results, we don't need to iterate through the prices array every time.

Push every pair of `<price, result>` to a stack. Pop lower price from the stack and accumulate the count.

```go
type StockSpanner struct {
	// we use a 2D array to simulate a tuple (price,days)
	Prices [][]int
}

func Constructor() StockSpanner {
	return StockSpanner{Prices: make([][]int, 0)}
}

func (this *StockSpanner) Next(price int) int {
	days := 1
	for len(this.Prices) != 0 && this.Prices[len(this.Prices)-1][0] <= price {
		days += this.Prices[len(this.Prices)-1][1]
		// we pop the stack to avoid accumulating repeatedly
		this.Prices = append(this.Prices[:len(this.Prices)-1], this.Prices[len(this.Prices):]...)
	}
	this.Prices = append(this.Prices, []int{price, days})
	return days
}
```

Time Complexity: $O(1)$, just think about the case [1,2,3,4,5].

#### 8.[Next Greater Element II](https://leetcode.com/problems/next-greater-element-ii)

Given a circular array (the next element of the last element is the first element of the array), print the Next Greater Number for every element. The Next Greater Number of a number x is the first greater number to its traversing-order next in the array, which means you could search **circularly** to find its next greater number. If it doesn't exist, output -1 for this number.

**Example 1:**

```
Input: [1,2,1]
Output: [2,-1,2]
Explanation: The first 1's next greater number is 2; 
The number 2 can't find next greater number; 
The second 1's next greater number needs to search circularly, which is also 2.
```

**Note:** The length of given array won't exceed 10000.

**My Solution**

Brute Force, again.

```go
// brute force
func nextGreaterElements(nums []int) []int {
	result := make([]int, len(nums))
	for i, v := range nums {
		result[i] = -1
		var j int
		if i == len(nums) - 1 {
			j = 0
		} else {
			j = i + 1
		}
		for j != i {
			if nums[j] > v {
				result[i] = nums[j]
				break
			}
			if j + 1 == len(nums) {
				j = 0
			} else {
				j++
			}
		}
	}
	return result
}
```

To lessen the number of `if` statements, use modulus (%) operation.

```go
// brute force
func nextGreaterElements(nums []int) []int {
	result := make([]int, len(nums))
	for i, v := range nums {
		result[i] = -1
		for j := 1; j < len(nums); j++ {
			if nums[(i+j)%len(nums)] > v {
				result[i] = nums[(i+j)%len(nums)]
				break
			}
		}
	}
	return result
}
```

Keep this trick in mind.

Time complexity (the worst case): $O(n^2)$, n is the length of `nums` .

**Improvement**

Considering the example `[5, 4, 3, 2, 1, 6]`, the greater number `6` is the next greater element for all previous numbers in the sequence. We use a stack to keep a **decreasing** sub-sequence, whenever we see a number `x` greater than the top of stack we pop all elements less than `x` and for all the popped ones, their next greater element is `x`.

```go
func nextGreaterElements(nums []int) []int {
	length := len(nums)
	// set the default value -1
	res := make([]int, length)
	for k, _ := range res {
		res[k] = -1
	}
	// the stack stores the indices of descending subarray
	stack := make([]int, 0)
	// we can transverse the array circularly, so i < length*2
	for i := 0; i < length*2; i++ {
		num := nums[i%length]
		// pop all indices of elements less than current number
		for len(stack) > 0 && nums[stack[len(stack)-1]] < num {
			res[stack[len(stack)-1]] = num
			stack = append(stack[:len(stack)-1], stack[len(stack):]...)
		}
		// avoid pushing the same index repeatedly
		if i < length {
			stack = append(stack, i)
		}
	}
	return res
}
```

Time Complexity: $O(n)$, n is the length of `nums`.

#### 9.[Flatten Nested List Iterator](https://leetcode.com/problems/flatten-nested-list-iterator)

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

**My Solution**

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

    public NestedIterator(List<NestedInteger> nestedList) {
        singleIntegers = new LinkedList<>();
        flatten(nestedList);
    }

    @Override
    public boolean hasNext() {
        return singleIntegers.isEmpty();
    }

    @Override
    public Integer next() {
        return singleIntegers.isEmpty() ? null : singleIntegers.remove(0);
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

Since it's Java, use `Iterator`to simplify the codes.

```java
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
```

**Other**

In the constructor, we push all the `nestedList` into the stack from back to front, so when we pop the stack, it returns the very first element. Second, in the `hasNext()` function, we peek the first element in stack currently, and if it is an Integer, we will return true and pop the element. If it is a list, we will further flatten it. This is iterative version of flatting the nested list. Again, we need to iterate from the back to front of the list.

```java
public class NestedIterator implements Iterator<Integer> {
    Stack<NestedInteger> stk = null;
    public NestedIterator(List<NestedInteger> nestedList) {
        stk = new Stack();
        flattenHelper(nestedList);
    }

    @Override
    public Integer next() {
        return stk.pop().getInteger();
    }

    @Override
    public boolean hasNext() {
        while(!stk.isEmpty()){
            NestedInteger ele = stk.peek();
            if(ele.isInteger())
                return true;
            else {
                stk.pop();
                flattenHelper(ele.getList());
            }
                
        }
        return false;
    }
    
    // NOTE: iterate through the list from back to front
    private void flattenHelper(List<NestedInteger> nestedList){
        for(int i=nestedList.size()-1 ; i>=0; i--){
            stk.push(nestedList.get(i));
        }
    }
}
```

Time Complexity : `next` - $O(1)$, `hasNext` - $O(m)$ : `m` is average size of nested list, Constructor : $O(n)$ - size of input list

#### 10.[Evaluate Reverse Polish Notation](https://leetcode.com/problems/evaluate-reverse-polish-notation/)

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

**My Solution**

Quite easy, hm?

```go
func evalRPN(tokens []string) int {
	stack := make([]int, 0)
	for _, v := range tokens {
		if !strings.Contains("+-*/", v) {
			// if we find a number, push it into the stack
			num, _ := strconv.Atoi(v)
			stack = append(stack, num)
		} else {
			// if we find an operator, pop the stack twice
			// to get two operands
			right := stack[len(stack)-1]
			stack = append(stack[:len(stack)-1], stack[len(stack):]...)
			left := stack[len(stack)-1]
			stack = append(stack[:len(stack)-1], stack[len(stack):]...)
			// evaluate the expression and
			// push the result into the stack
			stack = append(stack, calc(left, right, v))
		}
	}
	return stack[0]
}

func calc(left int, right int, op string) int {
	switch op {
	case "+":
		return left + right
	case "-":
		return left - right
	case "*":
		return left * right
	case "/":
		return left / right
	default:
		return 0
	}
}
```

Time Complexity: $O(n)$, n is the length of input array.

**Improvement**

Actually, `strings.Contains`is a time-consuming operation. We can use `switch`statement instead.

```go
func evalRPN(tokens []string) int {
	stack := make([]int, 0)
	var l int
	var r int
	for _, v := range tokens {
		switch v {
		case "+":
			l, r, stack = getOperands(stack)
			stack = append(stack, l+r)
		case "-":
			l, r, stack = getOperands(stack)
			stack = append(stack, l-r)
		case "*":
			l, r, stack = getOperands(stack)
			stack = append(stack, l*r)
		case "/":
			l, r, stack = getOperands(stack)
			stack = append(stack, l/r)
		default:
			num, _ := strconv.Atoi(v)
			stack = append(stack, num)
		}
	}
	return stack[0]
}

func getOperands(stack []int) (int, int, []int) {
	r := stack[len(stack)-1]
	stack = append(stack[:len(stack)-1], stack[len(stack):]...)
	l := stack[len(stack)-1]
	stack = append(stack[:len(stack)-1], stack[len(stack):]...)
	return l, r, stack
}
```
