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



