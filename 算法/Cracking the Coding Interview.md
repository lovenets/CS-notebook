# Core Data Structures, Algorithms and Concepts 

| Data Structure     | Algorithms    | Concepts                |
| ------------------ | ------------- | ----------------------- |
| Linked List        | BFS           | Bit Manipulation        |
| Tree, Trie & Graph | DFS           | Memory (Stack vs. Heap) |
| Stack & Queue      | Binary Search | Recursion               |
| Heap               | Merge Sort    | Dynamic Programming     |
| Vector/ArrayList   | Quick Sort    | Big O Time & Space      |
| Hash Table         |               |                         |

For each of these topics, make sure you understand how to use and implement them and, where applicable, the space and time complexity.

| Power Of 2 | Exact Value (X)       | Approx. Value | X bytes into MB, GB, etc |
| ---------- | --------------------- | ------------- | ------------------------ |
| 7          | 128                   |               |                          |
| 8          | 256                   |               |                          |
| 10         | 1024                  | 1, 000        | 1 kB                     |
| 16         | 65, 536               |               | 64 KB                    |
| 20         | 1, 048, 576           | 1 million     | 1 MB                     |
| 30         | 1, 073, 741, 824      | 1 billion     | 1 GB                     |
| 32         | 4, 294, 967, 296      |               | 4 GB                     |
| 40         | 1, 099, 511, 627, 776 | 1 trilliion   | 1 TB                     |

The table above is useful for many questions involving scalability or any sort of memory limitation. Memorizing this table isn't strictly required, but it can be useful. You should at least be comfortable deriving it 

# Big O

1.The following code copies an array. What is its runtime?

```java
int copyArray(int[] array) {
    int[] copy = new int[0];
    for (int value : array) {
        copy = appendToNew(copy, value);
    }
    return copy;
}

int[] appendToNew(int[] array, int value) {
    int[] bigger = new int[array.length + 1];
    for (int i = 0; i < array.length; i++) {
        bigger[i] = array[i];
    }
    bigger[bigger.length - 1] = value;
    return bigger;
}
```

$$O(n^2)$$, where n is the number of elements in the array. The first call to `appendToNew`takes 1 copy. The second call takes 2 copies. The third takes 3 copies. And so on. The total time will be the sum of 1 through n, which is $$O(n^2)$$.

2.The following code prints all strings of length k where the characters are in sorted order. What is its runtime?

```java
int numChars = 26;

void printSortedStrings(int remaining) {
    printSortedStrings(remaining, "");
}

void printSortedStrings(int remaining, String prefix) {
    if (remaining == 0) {
        if (isInOrder(prefix)) {
            System.out.Println(prefix);
        }
    } else {
        for (int i = 0; i < numChars; i++) {
            char c = ithLeter(i);
            printSortedStrings(remaining - 1, prefix + c);
        }
    }
}

boolean isInOrder(String s) {
    for (int i = 1; i < s.length; i++) {
        int prev = ithLeter(s.charAt(i - 1));
        int curr = ithLeter(s.charAt(i));
        if (prev > curr) {
            return false;
        }
    }
    return true;
}

char ithLeter(int i) {
    return (char) ((int) 'a' + 1);
}
```

$$O(kc^k)$$, where k is the length of the string and c is the number of characters in the alphabet. It takes $$O(c^k)$$ time to generate each string. Then, we need to check that each of these is sorted, which takes $$O(k)$$ time.

(Also, we can solve this problem from a different perspective. There are $$c^k$$ possible strings in total and it takes $$O(k)$$ time to check a string is sorted.)

## Best Conceivable Runtime (BCR)

The best conceivable runtime is the best runtime you could conceive of a solution to a problem having, i.e., it's the solution taking the least time *in theory*. You can easily prove that there is no way you could beat the BCR.

It tells us that we  are done in terms of optimizing the runtime and we should therefore turn our efforts to the space complexity. If we ever reach the BCR and have $$O(1)$$ additional space, then we know we can't optimize the time or space.

# Arrays and Strings

Array questions and string questions are often interchangeable. 

## 1. Is Unique

Implement an algorithm to determine if a string has all unique characters. What if you cannot use additional data structures?

(1) hash table  

```go
func IsUnique(s string) bool {
	if s == "" {
		return false
	}
	chars := make(map[rune]bool)
	for _, r := range s {
		if chars[r] {
			return false
		}
		chars[r] = true
	}
	return true
}
```

$$O(n)$$ time, $$O(n)$$ space.

(2) 

```go
// Assumes only letters a through z
func IsUnique(s string) bool {
	if l := len(s); l == 0 || l > 26 {
		return false
	}
    // As long as the string has all unique characters
    // the one bit must be different
	checker := 0
	for i := range s {
		if val := 1 << (s[i] - 'a'); (checker & val) > 0 {
			return false
		} else {
			checker |= val
		}
	}
	return true
}
```

$$O(n)$$ time, $$O(1)$$ space.

## 2. Check Permutation 

Given two strings, write a method to decide if one is a permutation of the other.

(1) count characters' frequency

```go
func CheckPermutation(a, b string) bool {
	if len(a) != len(b) || a == b {
		return false
	}
	// mapping a character to its frequency
	charsOfA := make(map[byte]int)
	for i := range a {
		charsOfA[a[i]]++
	}
	for i := range b {
		if _, ok := charsOfA[b[i]]; !ok {
			return false
		} else if charsOfA[b[i]]--; charsOfA[b[i]] < 0 {
			return false
		}
	}

	return true
}
```

$$O(n)$$ time, $$O(n)$$ space.

(2) sort

```go
func CheckPermutation(a, b string) bool {
	if len(a) != len(b) || a == b {
		return false
	}
	return sortString(a) == sortString(b)
}

func sortString(s string) string {
	ints := make([]int, len(s))
	for _, r := range s {
		ints = append(ints, int(r))
	}
	sort.Ints(ints)
	var b strings.Builder
	for i := range ints {
		b.WriteRune(rune(ints[i]))
	}
	return b.String()
}
```

$$O(nlogn)$$ time.

## 3. URLify

Write a method to replace all spaces in a string with '%20'. You may assume that the string has sufficient space at the end to hold the additional characters, and that you are given the "true" length of the string. (Note: If implementing in Java or other similar languages, please use a character array so that you can perform this operation in place.)

```go
func Urlify(s []rune, length int) []rune {
	// Count spaces.
	spaceCount := 0
	for i := 0; i < length; i++ {
		if s[i] == ' ' {
			spaceCount++
		}
	}
	// i is initialized to point to the last character of old string
	// j is initialized to point to the last character of new string
	for i, j := length-1, length+2*spaceCount-1; i >= 0; i-- {
		if s[i] == ' ' {
			// Replace space.
			s[j], s[j-1], s[j-2] = '0', '2', '%'
			j = j - 3
		} else {
			// Move other characters back.
			s[j], j = s[i], j-1
		}
	}
	return s
}
```

$$O(n)$$ time, $$O(1)$$ space.

## 4. Palindrome Permutation 

Given a string, write a function to check if it is a permutation of a palindrome. A palindrome is a word or phrase that is the same forwards and backwards. A permutation is a rearrangement of letters. The palindrome does not need to be limited to just dictionary word. 

```
EXAMPLE
Input: on no evil live star Rats
Output: True (palindrome: "Rats live on no evil star")
```

```go
func PalindromePermutation(s string) bool {
	if s == "" {
		return false
	}
	// Build character frequency table.
	freq := make(map[rune]int)
	for _, r := range s {
		if unicode.IsLetter(r) {
			freq[unicode.ToLower(r)]++
		}
	}
	// If a string is palindrome,
	// the number of letters occurring odd times
	// must be odd.
	count := 0
	for r := range freq {
		if freq[r]&1 == 1 {
            count++
		}
	}
	return count == 0 || count&1 == 1
}
```

$$O(n)$$ time, $$O(n)$$ space.

 ## 5. One Away

There are three types of edits that can be performed on strings: insert a character, remove a character, or replace a character. Given two strings, write a function to check if they are one edit (or zero edit) away.

```
EXAMPLE
pales, pale -> true
pale, bale  -> true
pale, bake  -> false
```

```go
func OneAway(a, b string) bool {
   if a == b {
      return true
   }
   // Try to insert a character.
   // Inserting a character into the short one equals
   // removing a character from the long one.
   if math.Abs(float64(len(a)-len(b))) == 1.0 {
      var s, l string
      if len(a) > len(b) {
         s, l = b, a
      } else {
         s, l = a, b
      }
      i := 0
      for ; i < len(s); i++ {
         if s[i] != l[i] {
            break
         }
      }
      if s[0:i]+string(l[i])+s[i:] == l {
         return true
      }
   }
   // Try to replace a character. 
   if len(a) == len(b) {
      count := 0
      for i := range a {
         if a[i] != b[i] {
            // There are more than one different characters. 
            if count++; count > 1 {
               return false
            }
         }
      }
      return true
   }
   return false
}
```

$$O(n)$$ time, $$O(1)$$ space.

## 6. String Compression 

Implement a method to perform basic string compression using the counts of repeated characters. For example, the string "aabcccccaaa" would become "a2b1c5a3", if the "compressed" string would not become smaller than the original string, your method should return the original string. You can assume the string has only uppercase and lowercase letters (a-z).

```go
func StringCompression(origin string) string {
	if origin == "" {
		return ""
	}
	var compressed string
	count := 0
	for i, r := range origin {
		count++
		// If next character is different, 
		// append it to result.
		if i+1 >= len(origin) || origin[i] != origin[i+1] {
			compressed += string(r) + strconv.Itoa(count)
			count = 0
		}
	}
	if len(compressed) >= len(origin) {
		compressed = origin
	}
	return compressed
}
```

$$O(n)$$ time, $$O(1)$$ space.

## 7. Rotate Matrix 

Given an image represented by an N*N matrix, where each pixel in the image is 4 bytes, write a method to rotate the image by 90 degrees. Can you do this in place?

```go
func RotateMatri(matrix [][]int) [][]int {
   if len(matrix) == 0 || len(matrix) != len(matrix[0]) {
      return nil
   }
   rows := len(matrix)
   for r := 0; r < rows/2; r++ {
      first, last := r, rows-1-r
      for i := first; i < last; i++ {
         offset := i - last
         top := matrix[first][i]
         // left -> top
         matrix[first][i] = matrix[last-offset][first]
         // bottom -> left
         matrix[last-offset][first] = matrix[last][last-offset]
         // right -> bottom
         matrix[last][last-offset] = matrix[i][last]
         // top -> right
         matrix[i][last] = top
      }
   }
   return matrix
}
```

$$O(N^2)$$ time, $$O(1)$$ space.

## 8. Zero Matrix 

Write an algorithm such that if an element in an M*N matrix is 0, its entire row and column are set to 0.

```go
func ZeroMatrix(matrix [][]int) [][]int {
   if len(matrix) == 0 {
      return nil
   }
   // Mark the zero cells' rows and columns.
   zeroRows, zeroCols := make([]int, 0), make([]int, 0)
   for i := range matrix {
      for j := range matrix[i] {
         if matrix[i][j] == 0 {
            zeroRows, zeroCols = append(zeroRows, i), append(zeroCols, j)
         }
      }
   }
   // Clear specific rows and columns.
   rows, cols := len(matrix), len(matrix[0])
   for i := range zeroRows {
      for j := 0; j < cols; j++ {
         matrix[zeroRows[i]][j] = 0
      }
   }
   for i := range zeroCols {
      for j := 0; j < rows; j++ {
         matrix[j][zeroCols[i]] = 0
      }
   }
   return matrix
}
```

$$O(nm)$$ time, $$O(n+m)$$ space where n is the number of rows and m is the number of columns.

## 9. String Rotation

Assume you have a method `isSubstring`which checks if one word is a substring of another. Given two strings, s1 and s2, write code to check if  s2 is a rotation of s1 using only one call to `isSubstring`(e.g., "waterbottle" is a rotation of "erbottlewat").

```go
func StringRotation(s1, s2 string) bool {
    // Assure s1 and s2 have the same length 
    // and are not empty.
   if len(s1) == len(s2) && len(s1) != 0 {
      // Too tricky
      return isSubstring(s1+s1, s2)
   }
   return false
}

func isSubstring(str, sub string) bool {
	return strings.Contains(str, sub)
}
```

# Linked List

**The "Runner" Technique**

The "runner" (or second pointer) technique is used in many linked list problems. The runner technique means that you iterate through the list with two pointers simultaneously, with one ahead of the other.  The "fast" node might be ahead by a fixed amount, or it might be hopping multiple nodes for each one node that the "slow" node iterates through. 

**Recursive Problems**

If you are having trouble solving a linked list problem, you should explore if a recursive approach will work. 

Recursive algorithms take at least $$O(n)$$ space, where n is the depth of the recursive call. All recursive algorithms can be implemented iteratively although they may be much more complex. 

## 1. Remove Dups

Write code to remove duplicates from an unsorted linked list. 

FOLLOW UP: How would you solve this problem if a temporary buffer is not allowed?

(1) 

```go
func RemoveDups(head *ListNode) *ListNode {
   if head == nil {
      return nil
   }
   dump := &ListNode{Next: head}
   existed := make(map[int]bool)
   for pre, cur := dump, head; cur != nil; {
      if !existed[cur.Value] {
         existed[cur.Value] = true
         pre, cur = pre.Next, cur.Next
      } else {
         // Remove the duplicate node. 
         cur = cur.Next
         pre.Next = cur
      }
   }
   return dump.Next
}
```

$$O(n)$$ time, $$O(n)$$ space.

(2) 

```go
func RemoveDups(head *ListNode) {
	for cur := head; cur != nil; cur = cur.Next {
		// Remove all future nodes that have the same value as current node.
		runner := cur
		for runner.Next != nil {
			if runner.Next.Value == cur.Value {
				runner.Next = runner.Next.Next
			} else {
				runner = runner.Next
			}
		}
	}
}
```

$$O(n^2)$$ time, $$O(1)$$ space.

## 2. Return Kth to Last

Implement an algorithm to find the kth to last element of a singly linked list. 

(1)

```go
func ReturnKthToLast(head *ListNode, k int) *ListNode {
	if head == nil || k <= 0 {
		return nil
	}
	fast, i := head, 0
	for ; fast != nil && i < k; fast, i = fast.Next, i+1 {
	}
	if i < k {
		// The number of nodes is less than k.
		return nil
	}
	slow := head
	for ; fast != nil && slow != nil; slow, fast = slow.Next, fast.Next {
	}
	return slow
}
```

$$O(n)$$ time, $$O(1)$$ space.

(2) 

```go
func ReturnKthToLast(head *ListNode, k int) *ListNode {
   if k <= 0 {
      return nil
   }
   index := 0
   return kToLast(head, k, &index)
}

func kToLast(node *ListNode, k int, i *int) *ListNode {
   if node == nil {
      return nil
   }
   tmp := kToLast(node.Next, k, i)
   if *i++; *i == k {
      return node
   }
   return tmp
}
```

$$O(n)$$ time, $$O(n)$$ space.

## 3. Delete Middle Node

Implement an algorithm to delete a node in the middle (i.e. any node but the first and last, not necessarily the exact middle) of a singly linked list, given only access to that node.

```
EXAMPLE
Input: the node c from the linked list a -> b -> c -> d -> e -> f
Result: nothing is return, but the new linked list looks like a -> b -> d -> e -> f
```

```go
func DeleteMiddleNode(middle *ListNode) {
	if middle == nil || middle.Next == nil {
		return
	}
	pre, cur, next := middle, middle, middle.Next
	for ; next != nil; pre, cur, next = cur, cur.Next, next.Next {
		// Overwrite current node's value with next node's.
		cur.Value = next.Value
	}
	// Remove the last node.
	pre.Next = nil
}
```

$$O(n)$$ time, $$O(1)$$ space.

## 4. Partition 

Write code to partition a linked list around a value x, such that all nodes less than x come before all nodes greater than or equal to x. If x is contained within the list, the value of x only need to be after the elements less than x. The partition element x can appear everywhere in the "right partition"; it doesn't need to appear between the left and right partitions.

```
EXAMPLE
Input: 3->5->8->5->10->2->1 [partition=5]
Output: 3->1->2->10->5->5->5->8
```

```go
func Partition(head *ListNode, x int) *ListNode {
   if head == nil {
      return nil
   }
   var headOfLess, less, headOfGreater, greater *ListNode
   for cur := head; cur != nil; cur = cur.Next {
      if cur.Value < x {
         if headOfLess == nil {
            headOfLess = cur
            less = headOfLess
         } else {
            less.Next = cur
            less = less.Next
         }
      } else {
         if headOfGreater == nil {
            headOfGreater = cur
            greater = headOfGreater
         } else {
            greater.Next = cur
            greater = greater.Next
         }
      }
   }
   if headOfLess != nil && headOfGreater != nil || headOfLess != nil {
      less.Next = headOfGreater
      return headOfLess
   } else {
      return headOfGreater
   }
}
```

$$O(n)$$ time, $$O(1)$$ space.

## 5. Sum Lists

You have two numbers represented by a linked list, where each node contains a single digit. The digits are stored in reverse order, such that the Vs digit is at the head of the list. Write a function that adds the two numbers and returns the sum as as linked list. 

```
EXAMPLE
Input: (7->1->6)+(5->9->2) That is 617 + 295
Output: 2->1->9 That is 912.
```

FOLLOW UP

Suppose the digits are stored in forward order. Repeat the above problem. 

(1) Digits stored in reverse order

```go
func SumListsInReverseOrder(a, b *ListNode) *ListNode {
   if a == nil && b == nil {
      return nil
   } else if a == nil {
      return b
   } else if b == nil {
      return a
   }
   // Assure that tow lists are not empty.
   var head, last *ListNode
   carry := 0
   p, q := a, b
   for ; p != nil && q != nil; p, q = p.Next, q.Next {
      tmp := p.Value + q.Value + carry
      if tmp > 10 {
         carry, tmp = 1, tmp-10
      } else {
         carry = 0
      }
      if head == nil {
         head = &ListNode{
            Value: tmp,
            Next:  nil,
         }
         last = head
      } else {
         node := &ListNode{
            Value: tmp,
            Next:  nil,
         }
         last.Next = node
         last = last.Next
      }
   }
   // If one list is longer than the other.
   if p != nil || q != nil {
      var cur *ListNode
      if p != nil {
         cur = p
      } else {
         cur = q
      }
      for ; cur != nil; cur = cur.Next {
         tmp := cur.Value + carry
         if tmp > 10 {
            carry, tmp = 1, tmp-10
         } else {
            carry = 0
         }
         node := &ListNode{
            Value: tmp,
            Next:  nil,
         }
         last.Next = node
         last = last.Next
      }
   }
   return head
}
```

$$O(n)$$ time, $$O(1)$$ space.

(2) Digits stored in forward order

Just reverse lists and do as above.

```go
func SumListsInForwardOrder(a, b *ListNode) *ListNode {
	reverse := func(head *ListNode) *ListNode {
		if head == nil {
			return nil
		}
		cur := head
		var pre, next *ListNode
		for cur != nil {
			next = cur.Next
			cur.Next = pre
			pre = cur
			cur = next
		}
		return pre
	}
	ra, rb := reverse(a), reverse(b)
	return reverse(SumListsInReverseOrder(ra, rb))
}
```

$$O(n)​$$ time, $$O(1)​$$ space.

## 6. Palindrome 

Implement a function to check if a linked list is a palindrome.

```go
func IsPalindrome(head *ListNode) bool {
   if head == nil {
      return false
   }
   var b strings.Builder
   for cur := head; cur != nil; cur = cur.Next {
      b.WriteByte(byte(cur.Value))
   }
   forward := b.String()
   rhead := reverseList(head)
   b.Reset()
   for cur := rhead; cur != nil; cur = cur.Next {
      b.WriteByte(byte(cur.Value))
   }
   reverse := b.String()
   return forward == reverse
}
```

$$O(n)$$ time, $$O(1)$$ space.

## 7. Intersection

Given two singly linked lists, determine if the two lists intersect. Return the intersecting nodes. Note that the intersection is defined based on reference, not value. That is, if the kth node of the first list is the exact same node (by reference) as the jth node of the second linked list, then they are intersecting.

```go
func Intersection(a, b *ListNode) *ListNode {
   if a == nil || b == nil {
      return nil
   }
   existed := make(map[*ListNode]bool)
   for cur := a; cur != nil; cur = cur.Next {
      existed[cur] = true
   }
   for cur := b; cur != nil; cur = cur.Next {
      if existed[cur] {
         return cur
      }
   }
   return nil
}
```

$$O(n)$$ time, $$O(n)$$ space.

## 8. Loop Detection

Given a circular linked list, implement an algorithm that returns the node at the beginning of the loop.

```go
func LoopDetection(head *ListNode) *ListNode {
        if p := detect(head); p != nil {
            return start(p, head)
        } else {
            return nil
        }
}

func detect(head *ListNode) *ListNode {
    slow, fast := head, head
    for slow != nil && fast != nil && fast.Next != nil {
        slow = slow.Next
        fast = fast.Next.Next
        if slow == fast {
            return slow
        }
    }
    return nil
}

func start(p, head *ListNode) *ListNode {
    q := head
    for p != q {
        p = p.Next
        q = q.Next
    }
    return q
}
```

$$O(n)$$ time, $$O(1)$$ space.

# Stacks and Queues

## 1. Three In One

Describe how you could use a single array to implement three stacks.

Just think of a array as it is combined with three arrays one by one.

```go
type MultiStacks struct {
   values   []int // underlying array
   sizes    []int // actual capacity of each stack
   capacity int   // max capacity of each stack
}

func NewMultiStacks(numOfStacks int, capacity int) *MultiStacks {
   return &MultiStacks{
      values:   make([]int, capacity*numOfStacks),
      sizes:    make([]int, numOfStacks),
      capacity: capacity,
   }
}

func (ms *MultiStacks) IsEmpty(idxOfStack int) (bool, error) {
   if idxOfStack >= len(ms.sizes) {
      return false, errors.New("index out of range")
   }
   return ms.sizes[idxOfStack] == 0, nil
}

func (ms *MultiStacks) Push(idxOfStack int, value int) error {
   if idxOfStack >= len(ms.sizes) {
      return errors.New("index of range")
   }
   if ms.sizes[idxOfStack] == ms.capacity {
      return errors.New("stack is full")
   }
   ms.values[idxOfStack*ms.capacity+ms.sizes[idxOfStack]] = value
   ms.sizes[idxOfStack]++
   return nil
}

func (ms *MultiStacks) Peek(idxOfStack int) (int, error) {
   if idxOfStack >= len(ms.sizes) {
      return 0, errors.New("index out of range")
   }
   if ms.sizes[idxOfStack] == 0 {
      return 0, errors.New("stack is empty")
   }
   return ms.values[idxOfStack*ms.capacity+ms.sizes[idxOfStack]-1], nil
}

func (ms *MultiStacks) Pop(idxOfStack int) error {
   if idxOfStack >= len(ms.sizes) {
      return errors.New("index out of range")
   }
   if ms.sizes[idxOfStack] == 0 {
      return errors.New("stack is empty")
   }
   ms.sizes[idxOfStack]--
   return nil
}
```

## 2. Stack Min

How would you design a stack which, in addition to pop and push, has a function min which returns the minimum element? Push, pop and min should all operate in $$O(1)$$ time.

Just make each element contain a field called `min`which represents the *current* minimum value on the stack, which means the minimum value in the "substack" ended with the element.

```go
type NodeWithMin struct {
	val int
	min int
}

type StackWithMin struct {
	nodes []NodeWithMin
}

func NewStackWithMin() *StackWithMin {
	return &StackWithMin{make([]NodeWithMin, 0)}
}

func (s *StackWithMin) Push(val int) {
	curMin := math.MaxInt64
	if len(s.nodes) > 0 {
		curMin = s.nodes[len(s.nodes)-1].min
	}
	if curMin > val {
		curMin = val
	}
	s.nodes = append(s.nodes, NodeWithMin{val, curMin})
}

func (s *StackWithMin) Pop() (int, error) {
	if len(s.nodes) == 0 {
		return 0, errors.New("empty stack")
	}
	top := s.nodes[len(s.nodes)-1]
	s.nodes = s.nodes[:len(s.nodes)-1]
	return top.val, nil
}

func (s *StackWithMin) Min() (int, error) {
	if len(s.nodes) == 0 {
		return 0, errors.New("empty stack")
	}
	return s.nodes[len(s.nodes)-1].min, nil
}
```

## 3. Stack Of Plates

Imagine a literal stack of plates. If the stack gets too high, it might topple. Therefore, in real life, we would likely start a new stack when the previous stack exceeds some threshold. Implement a data structure `SetOfStacks` that mimics this. `SetOfStacks` should be composed of several stacks and should create a new stack once the previous one exceeds capacity, `SetOfStacks.push()`and `SetOfStacks.pop()`should behave identically to a single stack (that is, `pop()`should return the same values as it would if there were just a single stack.)

FOLLOW UP

Implement a function `popAt(int index)`which preforms a pop operation on a specific sub-stack.

```go
type SetOfStacks struct {
   capacity int
   stacks   [][]int
}

func NewSetOfStacks(capacity int) (*SetOfStacks, error) {
   if capacity <= 0 {
      return nil, errors.New("invalid capacity")
   }
   stacks := make([][]int, 0)
   stacks = append(stacks, make([]int, capacity))
   return &SetOfStacks{
      capacity: capacity,
      stacks:   stacks,
   }, nil
}

func (ss *SetOfStacks) Push(val int) {
   if len(ss.stacks[len(ss.stacks)-1]) == ss.capacity {
      newStack := make([]int, ss.capacity)
      newStack = append(newStack, val)
      ss.stacks = append(ss.stacks, newStack)
   } else {
      ss.stacks[len(ss.stacks)-1] = append(ss.stacks[len(ss.stacks)-1], val)
   }
}

func (ss *SetOfStacks) Pop() (int, error) {
   if len(ss.stacks[len(ss.stacks)-1]) == 0 {
      return 0, errors.New("empty set of stacks")
   }
   lenOfLastStack := len(ss.stacks[len(ss.stacks)-1])
   top := ss.stacks[len(ss.stacks)-1][lenOfLastStack-1]
   ss.stacks[len(ss.stacks)-1] = ss.stacks[len(ss.stacks)-1][:lenOfLastStack-1]
   return top, nil
}

func (ss *SetOfStacks) PopAt(index int) (int, error) {
   if index < 0 || index >= len(ss.stacks) {
      return 0, errors.New("index of range")
   }
   if len(ss.stacks[index]) == 0 {
      return 0, errors.New("empty stack")
   }
   lenOfStack := len(ss.stacks[index])
   top := ss.stacks[index][lenOfStack-1]
   ss.stacks[index] = ss.stacks[index][:lenOfStack-1]
   return top, nil
}
```

## 4. Queue via Stacks

Implement a MyQueue class which implements a queue using two stacks.

```go
type MyQueue struct {
	stack1 *Stack
	stack2 *Stack
}

func NewMyQueue() *MyQueue {
	return &MyQueue{NewStack(), NewStack()}
}

func (mq *MyQueue) IsEmpty() bool {
	return mq.stack1.IsEmpty()
}

func (mq *MyQueue) Add(val int) {
	mq.stack1.Push(val)
}

func (mq *MyQueue) Remove() (int, error) {
	for mq.stack1.Size() > 1 {
		val, _ := mq.stack1.Pop()
		mq.stack2.Push(val)
	}
	peek, _ := mq.stack1.Pop()
	for !mq.stack2.IsEmpty() {
		val, _ := mq.stack2.Pop()
		mq.stack1.Push(val)
	}
	return peek, nil
}
```

## 5. Sort Stack

Write a program to sort a stack such that the smallest item are on the top. You can use an additional stack, but you may not copy the elements into any other data structure (such as an array). The stack supports the following operations: `push`, `pop`, `peek` and `isEmpty`.

1. Create a temporary stack say **helper stack**.
2. While input stack is NOT empty do this:
   - Pop an element from input stack call it **temp**
   - While temporary stack is NOT empty and top of temporary stack is greater than temp, pop from temporary stack and push it to the input stack. By doing this, we can assure elements in helper stack are in order and max is on the top.
   - Push **temp** in helper stack
3. The sorted numbers are in helper stack

```go
func (s *Stack) Sort() {
   helper := NewStack()
   for !s.IsEmpty() {
      // Push each element in s in sorted order to helper stack.
      tmp, _ := s.Pop()
      for !helper.IsEmpty() && helper.Peek() > tmp {
         val, _ := helper.Pop()
         s.Push(val)
      }
      helper.Push(tmp)
   }
   // Now elements in helper are in order and max is on the top.
   for !helper.IsEmpty() {
      val, _ := helper.Pop()
      s.Push(val)
   }
}
```

$$O(n^2)$$ time in the worst case?

## 6. Animal Shelter 

An animal shelter, which holds only dogs and cats, operates on a strictly    "first in, first out" basis. People must adopt either the "oldest" (based on arrival time) of all animals at the shelter, or they can select whether they would prefer a dog or a cat (and will receive the oldest animal of that type). They cannot select which specific animal they would like. Create the data structure to maintain this system and implement operations such as `enqueue`, `dequeueAny`, `dequeueDog`, and `dequeueCat`. You may use the built-in linked list data structure.

```go
type Animal struct {
   Id   int
   Kind int // 1: dog, 2: cat
}

type AnimalShelter struct {
   animals []*Animal
}

func NewAniamlShelter() *AnimalShelter {
   return &AnimalShelter{make([]*Animal, 0)}
}

func (as *AnimalShelter) Enqueue(a *Animal) {
   as.animals = append(as.animals, a)
}

func (as *AnimalShelter) DequeueAny() (*Animal, error) {
   if len(as.animals) == 0 {
      return nil, errors.New("empty shelter")
   }
   peek := as.animals[0]
   as.animals = as.animals[1:]
   return peek, nil
}

func (as *AnimalShelter) DequeueDog() (*Animal, error) {
   return as.dequeueKind(1)
}

func (as *AnimalShelter) DequeueCat() (*Animal, error) {
   return as.dequeueKind(2)
}

func (as *AnimalShelter) dequeueKind(kind int) (*Animal, error) {
   if len(as.animals) == 0 {
      return nil, errors.New("empty shelter")
   }
   for i := range as.animals {
      if as.animals[i].Kind == kind {
         animal := as.animals[i]
         as.animals = append(as.animals[:i], as.animals[i+1:]...)
         return animal, nil
      }
   }
   return nil, errors.New("no such kind animal")
}
```

# Trees and Graphs

## 1. Route Between Nodes

Given a directed graph, design an algorithm to find out whether there is a route between two nodes.

Assuming a graph is represented by a adjacency matrix.

```go
func RouteBetweenNodes(g [][]int, u int, v int) bool {
	if maxIdx := len(g) - 1; maxIdx < 0 ||
		u < 0 || u >= maxIdx ||
		v < 0 || v >= maxIdx {
		return false
	}
	visited := make([]bool, len(g))
	// BFS
	q := make([]int, 0)
	q = append(q, u)
	visited[u] = true
	for len(q) > 0 {
		vertex := q[0]
		q = q[1:]
		if vertex == v {
			return true
		}
		for i := range g[vertex] {
			if g[vertex][i] == 1 && !visited[i] {
				q = append(q, i)
				visited[i] = true
			}
		}
	}
	return false
}
```
$$O(n^2)$$ time, $$O(n)$$ space.

```go
func RouteBetweenNodesDfs(g [][]int, u int, v int) bool {
   if maxIdx := len(g) - 1; maxIdx < 0 ||
      u < 0 || u >= maxIdx ||
      v < 0 || v >= maxIdx {
      return false
   }
   visited := make([]bool, len(g))
   return dfs(&g, u, v, &visited)
}

func dfs(g *[][]int, u int, v int, visited *[]bool) bool {
	if !(*visited)[u] {
		(*visited)[u] = true
		if u == v {
			return true
		}
		for i := range (*g)[u] {
			if (*g)[u][i] == 1 {
				if dfs(g, i, v, visited) {
					return true
				}
			}
		}
	}
	return false
}
```
$$O(n^2)$$ time, $$O(n^2)$$ space.

## 2. Minimal Tree

Given a sorted (increasing order) array with unique integer elements, write an algorithm to create a binary tree with minimal height.

Since the array is sorted, then we just build the BST recursively using the median as root.

```go
func MinimalTree(numbers []int) *TreeNode {
   if len(numbers) == 0 || !sort.IsSorted(sort.IntSlice(numbers)) {
      return nil
   }
   mid := len(numbers) / 2
   return &TreeNode{
      Val:   numbers[mid],
      Left:  MinimalTree(numbers[0:mid]),
      Right: MinimalTree(numbers[mid+1:]),
   }
}
```

$$O(n)$$ time, $$O(n)$$ space.

## 3. List of Depths

Given a binary tree, design an algorithm which creates a linked list of all the nodes at each depth (e.g., if you have a tree with depth D, you will have D linked lists).

```go
func ListOfDepths(root *TreeNode) []*ListNode {
	if root == nil {
		return nil
	}
	// Level-order traversal
	q := make([]*TreeNode, 0)
	q = append(q, root)
	numOfNodesAtCurLevel, numOfNodesAtNextLevel := 1, 0
	nodesAtCurLevel := make([]int, 0)
	lists := make([]*ListNode, 0)
	for len(q) > 0 {
		node := q[0]
		q = q[1:]
		if node.Left != nil {
			q = append(q, node.Left)
			numOfNodesAtNextLevel++
		}
		if node.Right != nil {
			q = append(q, node.Right)
			numOfNodesAtNextLevel++
		}
		nodesAtCurLevel = append(nodesAtCurLevel, node.Val)
		if numOfNodesAtCurLevel--; numOfNodesAtCurLevel == 0 {
			// Build a list
			head := &ListNode{Value: nodesAtCurLevel[0]}
			cur := head
			for _, v := range nodesAtCurLevel[1:] {
				cur.Next = &ListNode{Value: v}
				cur = cur.Next
			}
			lists = append(lists, head)
			numOfNodesAtCurLevel = numOfNodesAtNextLevel
			numOfNodesAtNextLevel = 0
			nodesAtCurLevel = nodesAtCurLevel[:0]
		}
	}
	return lists
}
```

$$O(n)$$ time, $$O(n)$$ space.

## 4. Check Balanced

Implement a function to check if a binary tree is balanced. For the purposes of this question, a balanced tree is defined to be a tree such that the length of the two subtrees of any node never differ by more than one.

```go
func CheckBalanced(root *BinaryTreeNode) bool {
	depth := 0
	return postorder(root, &depth)
}

func postorder(root *BinaryTreeNode, depth *int) bool {
	if root == nil {
		*depth = 0
		return true
	}
	dl, dr := 0, 0 // depths of two subtrees
	l, r := postorder(root.Left, &dl), postorder(root.Right, &dr)
	if l && r {
		if math.Abs(float64(dl)-float64(dr)) <= 1.0 {
             // If this subtree is balanced, then propagate the result and its height
			*depth = int(math.Max(float64(dl), float64(dr))) + 1
			return true
		}
	}
	return false
}
```

$$O(n)$$ time, $$O(logn)$$ space.

## 5. Validate BST

Implement a function to check if a binary tree is a binary search tree.

Do inorder-traversal iteratively.

```go
func ValidateBST(root *TreeNode) bool {
	if root == nil {
		return true
	}
	stack := make([]*TreeNode, 0)
	var pre *TreeNode
	for cur := root; cur != nil || len(stack) > 0; {
		for cur != nil {
			stack = append(stack, cur)
			cur = cur.Left
		}
		top := stack[len(stack)-1]
		stack = stack[:len(stack)-1]
		if pre != nil && pre.Val >= top.Val {
			return false
		}
		pre, cur = top, top.Right
	}
	return true
}
```

$$O(n)$$ time, $$O(n)$$ space.

