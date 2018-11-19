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

