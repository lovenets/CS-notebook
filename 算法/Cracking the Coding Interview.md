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

$$O(n)$$ time, $$O(1)$$ space.

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
    Name string
    Kind int // 1: dog, 2: cat
}

type AnimalWrapper struct {
    id     int
    animal *Animal
}

type AnimalShelter struct {
    count int
    dogs  []*AnimalWrapper
    cats  []*AnimalWrapper
}

func NewAniamlShelter() *AnimalShelter {
    return &AnimalShelter{0, make([]*AnimalWrapper, 0), make([]*AnimalWrapper, 0)}
}

func (as *AnimalShelter) Enqueue(a *Animal) {
    as.count++
    switch a.Kind {
    case 1:
        as.dogs = append(as.dogs, &AnimalWrapper{
            id:     as.count,
            animal: a,
        })
    case 2:
        as.cats = append(as.cats, &AnimalWrapper{
            id:     as.count,
            animal: a,
        })
    }
}

func (as *AnimalShelter) DequeueAny() (*Animal, error) {
    if len(as.cats) == 0 && len(as.dogs) == 0 {
        return nil, errors.New("empty shelter")
    }
    var peek *Animal
    if numOfCats, numOfDogs := len(as.cats), len(as.dogs); numOfCats > 0 && numOfDogs > 0 {
        if as.cats[0].id < as.dogs[0].id {
            peek = as.cats[0].animal
            as.cats = as.cats[1:]
        } else {
            peek = as.dogs[0].animal
            as.dogs = as.dogs[1:]
        }
    } else if numOfCats > 0 {
        peek = as.cats[0].animal
        as.cats = as.cats[1:]
    } else {
        peek = as.dogs[0].animal
        as.dogs = as.dogs[1:]
    }
    return peek, nil
}

func (as *AnimalShelter) DequeueDog() (*Animal, error) {
    if len(as.dogs) == 0 {
        return nil, errors.New("no dog")
    }
    dog := as.dogs[0].animal
    as.dogs = as.dogs[1:]
    return dog, nil
}

func (as *AnimalShelter) DequeueCat() (*Animal, error) {
    if len(as.cats) == 0 {
        return nil, errors.New("no dog")
    }
    cat := as.cats[0].animal
    as.cats = as.cats[1:]
    return cat, nil
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

## 6. Successor

Write an algorithm to find the "next" node (i.e., in-order successor) of a given node in a binary search tree. You may assume that each node has a link to its parent. 

If this given node has right subtree, then it's next in-order node is the leftmost node of its right subtree. 

If it doesn't have a right subtree, then we are done traversing its subtree. Let's say the given node is n. We'll need to pick up where we left off with n's parent, which we'll call q. 

- If n was to the left of q, then the next node should be q. (left->current->right)
- If n was to the right of q, then we have fully traversed q's subtree as well. We need to traverse upwards from q until we find a node x which we have not fully traversed. We know we hit this case when we move from a left node to its parent. The left node is fully traversed, but its parent is not.

There is one corner case: if we are already on the far right of the tree, then there is no in-order successor. 

```go
func Successor(n *TreeNode) *TreeNode {
    if n == nil {
        return nil
    }
    if n.Right != nil {
        // Find the leftmost node of n's right subtree
        cur := n.Right
        for ; cur != nil && cur.Left != nil; cur = cur.Left {
        }
        return cur
    } else {
        q := n
        x := q.Parent
        // Go up until we're on left instead of right.
        // Here we also handle the corner case.
        for x != nil && x.Left != q {
            q = x
            x = x.Parent
        }
        return x
    }
}
```

The worst case takes $$O(logn)$$ time. $$O(1)$$ space.

## 7. Build Order

You are given a list of products and a list of dependencies (which is a list of pairs of projects, where the second project is dependent on the first project). All of a project's dependencies must be built before the project is. Find a build order that will allow the projects to be built. If there is no valid build order, return an error. 

```
EXAMPLE
Input: 
    projects: a, b, c, d, e, f
    dependencies: (a, b), (f, b), (b, d). (f, a), (d, c)
Output: e, a, b, d, c
```

```go
func BuildOrder(projects []rune, dependencies [][]rune) ([]rune, error) {
    if len(projects) == 0 {
        return nil, errors.New("invalid input")
    }
    // Construct a directed graph represented by adjacency list.
    adj := make(map[rune][]rune, len(projects))
    indegree := make(map[rune]int, len(projects))
    for _, edge := range dependencies {
        if _, ok := adj[edge[0]]; !ok {
            adj[edge[0]] = make([]rune, 0)
        }
        adj[edge[0]] = append(adj[edge[0]], edge[1])
        indegree[edge[1]]++
    }
    zeroIndegree := make([]rune, 0 ,len(projects)) // Vertices which have no dependency.
    for _, n := range projects {
        if indegree[n] == 0 {
            zeroIndegree = append(zeroIndegree, n)
        }
    }
    // Do topological sorting.
    order := make([]rune, 0, len(projects)) // Resulting order.
    for len(zeroIndegree) > 0 {
        n := zeroIndegree[0]
        order = append(order, n)
        zeroIndegree = zeroIndegree[1:]
        for _, t := range adj[n] {
            if indegree[t]--; indegree[t] == 0 {
                zeroIndegree = append(zeroIndegree, t)
            }
        }
    }
    if len(order) != len(projects) {
        return nil, errors.New("can not order")
    } else {
        return order, nil
    }
}
```

This solution takes $$O(P + D)$$ time, where P is the number of projects and D is the number of dependency pairs.

## 8. First Common Ancestor

Design an algorithm and write code to find the first common ancestor of two nodes in a binary tree. Avoid storing additional nodes in data structure. NOTE: This is not necessarily a BST.

Assume that each node only has links to its children. 

if p and q are both on the left of the node, branch left to look for the common ancestor. If they are both on the right, branch right to look for the common ancestor. When p and q are no longer on the same side, we must have found the first common ancestor.

```go
func FirstCommonAncestor(root, p, q *TreeNode) *TreeNode {
   if !covers(root, p) || !covers(root, q) {
      return nil
   }
   return ancestorHelper(root, p, q)
}

func ancestorHelper(root, p, q *TreeNode) *TreeNode {
   if root == nil || root == p || root == q {
      return root
   }
   pIsOnLeft, qIsOnRight := covers(root.Left, p), covers(root.Left, q)
   if pIsOnLeft != qIsOnRight {
      return root
   }
   if pIsOnLeft {
      return ancestorHelper(root.Left, p, q)
   } else {
      return ancestorHelper(root.Right, p, q)
   }
}

func covers(root, p *TreeNode) bool {
   if root == nil {
      return false
   }
   if root == p {
      return true
   }
   return covers(root.Left, p) || covers(root.Right, p)
}
```

This algorithm runs in $$O(n)$$ time on a balanced tree.

## 9. BST Sequences

A binary search tree was created by traversing through an array from left to right and inserting each element. Given a binary search tree with distinct elements, print all possible arrays that could have led to this tree.

```
EXAMPLE
Input: 
    2
   /  \
  1    3
Output: {2, 1, 3}, {2, 3, 1}
```

In the example above, the very first element in our array must be a 2 in order to create the above tree. 

Let's think about this problem recursively. If we had all arrays that could have created the subtree rooted at 1 (call this `arrays1`) and all arrays that could have created that subtree rooted at 3 (call this `arrays3`), we could just weave each array from `arrays1`with each array from `array3`and then prepend array with a 2.

Weaving means we are merging two arrays in all possible ways, while keeping elements within each array in the same relative order.

```
array1: [1, 2]
array2: [3, 4]
weaved: [1, 2, 3, 4], [1, 3, 2, 4], [1, 3, 4, 2], [3, 1, 2, 4], [3, 1, 4, 2], [3, 4, 1, 2]
```

Note that, as long as there aren't any duplicates in the original array sets, we won't have to worry that weaving will create duplicates.

How does weaving work? Let's say, we want to weave [1, 2] and [3, 4] and the subproblems are:

```
- prepend 1 to all weaves of [2] and [3, 4]
- prepend 3 to all weaves of [1, 2] and [4]
```

To implement this, we'll store each as linked lists. This will make it easy to add and remove elements. When we recurse, we'll push the prefixed elements down the recursion. When first or second are empty, we add the remainder to prefix and store the result.

```
weave(first, second, prefix)
    weave([1, 2], [3, 4], [])
        weave([2], [3, 4], [1])
            weave([], [3, 4], [1, 2])
                [1, 2, 3, 4]
            weave([2], [4], [1, 3])
                weave([], [4], [1, 3, 2])
                    [1, 3, 2 ,4]
                weave([2], [], [1, 3 ,4])
                    [1, 3, 4, 2]
        weave([1, 2], [4], [3])
            weave([2], [4], [3, 1])
                weave([], [4], [3, 1, 2])
                    [3, 1, 2, 4]
                weave([2], [], [3, 1, 4])
                    [3, 1, 4, 2]
            weave([1, 2], [], [3, 4])
                [3, 4, 1, 2]    
```

```go
func BSTSequences(root *TreeNode) [][]int {
   res := make([][]int, 0)
   if root == nil {
      // Append [] so that the nested for loop will still execute
      res = append(res, []int{})
      return res
   }
   // prefix will always be the root of subtrees.
   prefix := []int{root.Val}
   leftSeq, rightSeq := BSTSequences(root.Left), BSTSequences(root.Right)
   for i := range leftSeq {
      for j := range rightSeq {
         weaved := make([][]int, 0)
         weave(leftSeq[i], rightSeq[j], &weaved, prefix)
         res = append(res, weaved...)
      }
   }
   return res
}

func weave(first []int, second []int, res *[][]int, prefix []int) {
   if len(first) == 0 || len(second) == 0 {
      // Assure that tmp is a copy.
      tmp := make([]int, len(prefix))
      copy(tmp, prefix)
      tmp = append(tmp, first...)
      tmp = append(tmp, second...)
      *res = append(*res, tmp)
      return
   }

   headOfFirst := first[0]
   first = first[1:]
   prefix = append(prefix, headOfFirst)
   weave(first, second, res, prefix)
   // Exit when fist is empty.
   // Reset prefix for the second recursion below.
   prefix = prefix[:len(prefix)-1]
   first = append([]int{headOfFirst}, first...)

   headOfSecond := second[0]
   second = second[1:]
   prefix = append(prefix, headOfSecond)
   weave(first, second, res, prefix)
   prefix = prefix[:len(prefix)-1]
   second = append([]int{headOfSecond}, second...)
}
```

## 10. Check Subtrees

T1 and T2 are two very large binary trees, with T1 much bigger than T2. Create an algorithm to determine if T2 is a subtree of T1.

A tree T2 is a subtree of T1 if there exists a node n in T1 such that the subtree of n is identical to T2. That is, if you cut off the tree at node n, the two trees would be identical.

(1) Transform trees into strings.

In this smaller, simpler problem, we could consider comparing string representations of traversals of each tree. If T2 is a subtree of T1, then T2's traversal should be a substring of T1.

An in-order traversal will definitely not work. After all, consider a scenario in which we were using binary search trees. A binary search tree's in-order traversal always prints out the values in sorted order. Therefore, two binary search trees with the same values will always have the same in-order traversals, even if their structure is different.

An pre-order traversal, with null nodes represented properly, can work. As long as we represent the NULL nodes, the pre-order traversal of a tree is unique. That is, if two trees have the same pre order traversal, then we know they are identical trees in values and structure. 

```go
func CheckSubtrees(t1, t2 *TreeNode) bool {
   var tStr1 string
   traverseTree(t1, &tStr1)
   var tStr2 string
   traverseTree(t2, &tStr2)
   return strings.Contains(tStr1, tStr2)
}

func traverseTree(root *TreeNode, s *string) {
   if root == nil {
      *s += "#"
      return
   }
   *s += strconv.Itoa(root.Val)
   traverseTree(root.Left, s)
   traverseTree(root.Right, s)
}
```

$$O(n+m)$$ time and $$O(n+m)$$ space where n and m are the number of nodes in T1 and T2 respectively.

(2) Search and compare.

Search through the larger tree T1. Each time a node in Tl matches the root of T2, call `matchTree`. The `matchTree` method will compare the two subtrees to see if they are identical.

```go
func CheckSubtrees(t1, t2 *TreeNode) bool {
	if t2 == nil {
		// An empty tree is always a subtree.
		return true
	} 
	return subtree(t1, t2)
}

func subtree(t1 *TreeNode, t2 *TreeNode) bool {
	if t1 == nil {
		return false
	} else if t1.Val == t2.Val && matchTree(t1, t2) {
		return true
	}
	return subtree(t1.Left, t2) || subtree(t1.Right, t2)
}

func matchTree(t1, t2 *TreeNode) bool {
	if t1 == nil && t2 == nil {
		return true
	} else if t1 == nil || t2 == nil {
		return false
	} else if t1.Val != t2.Val {
		return false
	} else {
		return matchTree(t1.Left, t2.Left) && matchTree(t1.Right, t2.Right)
	}
}
```

## 11. Random Node

You are implementing a binary search tree class from scratch, which, in addition to insert, find and delete, has a method `getRandomNode()`which returns a random node from the tree. All nodes should be equally likely to be chosen. Design and implement an algorithm for `getRandomNode`, and explain how you would implement the rest of the methods.

We can start with the root. With what probability should we return the root? Since we have N nodes, we must return the root node with $$1/n$$ probability.

With what probability should we traverse left versus right? It's not 50/50. Even in a balanced tree, the number of nodes on each side might not be equal. If we have more nodes on the left than the right, then we need to go left more often.

The odds of picking something from the left must have probability $$LEFTSIZE * 1/n$$ .This should therefore be the odds of going left. Likewise, the odds of going right should be $$RIGHTSIZE * 1/n$$.

This means that each node must know the size of the nodes on the left and the size of the nodes on the right. Fortunately, our interviewer has told us that we're building this tree class from scratch. It's easy to keep track of this size information on inserts and deletes. We can just store a `size` variable in each node. Increment`size`on inserts and decrement it on deletes.

```go
type TreeNode struct {
   Val   int
   Left  *TreeNode
   Right *TreeNode
   size  int
}

func NewTreeNode(val int) *TreeNode {
   return &TreeNode{
      Val:   val,
      Left:  nil,
      Right: nil,
      size:  1,
   }
}

func insert(root *TreeNode, val int) {
   if val <= root.Val {
      if root.Left == nil {
         root.Left = NewTreeNode(val)
      } else {
         insert(root.Left, val)
      }
   } else {
      if root.Right == nil {
         root.Right = NewTreeNode(val)
      } else {
         insert(root.Right, val)
      }
   }
   root.size++
}

func GetRandomNode(root *TreeNode) *TreeNode {
   var leftSize int
   if root.Left == nil {
      leftSize = 0
   } else {
      leftSize = root.Left.size
   }
   if idx := rand.Intn(root.size); idx < leftSize {
      return GetRandomNode(root.Left)
   } else if idx == leftSize {
      return root
   } else {
      return GetRandomNode(root.Right)
   }
}
```

$$O(D)$$ time, where D is the max depth. 

## 12. Paths with Sums

You are given a binary tree in which each node contains an integer value (which might be positive or negative). Design an algorithm to count the number of paths that sum to a given value. The path does not need to start or end at the root or a leaf, but it must go downwards (traveling only from parent nodes to child nodes).

(1) Brute Force

We traverse to each node. At each node, we recursively try all paths downwards, tracking the sum as we go. As soon as we hit our target sum, we increment the total.

```go
func PathsWithSums(root *TreeNode, target int) int {
   if root == nil {
      return 0
   }
   return pathsWithSumsFromNode(root, target, 0) +
      PathsWithSums(root.Left, target) +
      PathsWithSums(root.Right, target)
}

func pathsWithSumsFromNode(node *TreeNode, target int, current int) int {
   if node == nil {
      return 0
   }
   current += node.Val
   total := 0
   if current == target {
      total++
   }
   return total +
      pathsWithSumsFromNode(node.Left, target, current) +
      pathsWithSumsFromNode(node.Right, target, current)
}
```

In a balanced binary tree, d will be no more than approximately $$logN$$. Therefore, we know that with N nodes in the tree, `pathsWithSumsFromNode`be called $$O(NlogN)$$ times. The runtime is $$O(NlogN)$$.

In an unbalanced tree, the runtime could be much worse. Consider a tree that is just a straight line down. At the root, we traverse to $$N - 1$$ nodes. At the next level (with just a single node), we traverse to $$N - 2$$ nodes. At the third level, we traverse to $$N - 3$$ nodes, and so on. This  leads us to the sum of numbers between 1 and N, which is $$O(N^2)$$.

(2) 

In analyzing the last solution, we may realize that we repeat some work.

If we treat a path as an array, the problem will be: How many contiguous subsequences in this array sum to a target sum. Since we're just looking for the number of paths, we can use a hash table. As we iterate through the array, build a hash table that maps from a `current` to the number of times we've seen that sum.Then, for each y, look up `current-target` in the hash table. The value in the hash table will tell you the number of paths with sum `target`that end at y.

```go
func PathsWithSums(root *TreeNode, target int) int {
   return pathsWithSumsFromNode(root, target, 0, map[int]int{})
}

func pathsWithSumsFromNode(node *TreeNode, target int, current int, pathCount map[int]int) int {
   if node == nil {
      return 0
   }
   // Count path with sum ending at the current node.
   current += node.Val
   sum := current - target
   total := pathCount[sum]
   if current == target {
      // One additional path starts at current node.
      total++
   }
   modifyMap(pathCount, current, 1)
   total += pathsWithSumsFromNode(node.Left, target, current, pathCount) +
      pathsWithSumsFromNode(node.Right, target, current, pathCount)
   // Reverse the changes to the map so that 
   // other nodes won't use it.
   modifyMap(pathCount, current, -1)
   return total
}

func modifyMap(pathCount map[int]int, key int, delta int) {
   if newCount := pathCount[key] + delta; newCount == 0 {
      // Remove key-value pair when zero
      // in order to reduce space usage
      delete(pathCount, key)
   } else {
      pathCount[key] = newCount
   }
}
```

The runtime for this algorithm is $$O(N)$$, where N is the number of nodes in the tree because we travel to each node just once, doing $$O(1)$$ work each time. In a balanced tree, the space complexity is $$O(NlogN)$$ due to the hash table. The space complexity can grow to $$O(N)$$ in an unbalanced.

# Bit Manipulation

**Two's Complement and Negative Numbers**

The two's complement of an N-bit number (where N is the number of bits used for the number *excluding* the sign bit) is the complement of the number with $$2^N$$.

Let's look at the 4-bit integer -3 as an example. If it's a 4-bit number, we have one bit for the sign and three bits for the value. We want the complement with respect to $$2^3$$, which is 8. The complement of 3 (the absolute value of -3) with respect to 8 is 5. 5 in binary is 101. Therefore, -3 in binary as a 4-bit number is 1101, with the first bit being the sign bit. In other words, the binary representation of -K (negative K) as a N-bit number is concat(1, $$2^{N-1}-K$$).

Another way to look at this is that we invert the bits in the positive representation and then add 1, 3 is 011 in binary. Flip the bits to get 100, add 1 to get 101, then prepend the sign bit (1) to get 1101. 

**Arithmetic vs. Logical Right Shift**

The arithmetic right shift essentially divides by two. The logical right shift does what we would visually see as shifting the bits. This is best seen on a negative number. 

**Common Bit Tasks: Getting And Setting**

1.Get Bit

This method shifts 1 over by `i` bits, creating a value that looks like 00010000. By performing an AND with it, we clear all bits other than the bit at bit `i`. Finally, we compare that to 0. If that new value is not zero, then bit `i` must have a 1. Otherwise, bit `i` is a 0. 

2.Set Bit

Shift 1 over by `i` bits, creating a value like 00910000, By performing an OR with `num`, only the value at bit `i` will change. All other bits of the mask are zero and will not affect `num`.  

3.Clear Bit

First, we create a number like 11101111 by creating the reverse of it (00010000) and negating it. Then, we perform an AND with `num`. This will clear the it h bit and leave the remainder unchanged.

4.Update Bit

To set the ith bit to a value v, we first clear the bit at position `i` by using a mask that looks like 11101111. Then, we shift the intended  value, v, left by `i` bits. This will create a number with bit `i` equal to v and all other bits equal to 0. Finally, we OR these two numbers, updating the it h bit if v is 1 and leaving it as 0 otherwise. 

## 1. Insertion

You are given two 32-bit numbers, N and M, and two bit positions, i and j. Write a method to insert M into N such that M starts at bit j and ends at bit i. You can assume that the bits through i have enough space to fit all of M. That is, if M = 10011, you can assume that there are at least 5 bits between j and i. You would not, for example, have j = 3 and i = 2, because M could not fully fit between 3 and bit 2.

```
EXAMPLE
Input: N = 10000000000, M = 10011, i = 2, j = 6
Output: N = 10001001100
```

This problem can be approached in three key steps:
1. Clear the bits j through i in N
2. Shift M so that it lines up with bits j through i
3. Merge M and N.

```go
func Insert(n int32, m int32, i int, j int) int32 {
	// Create a mask to clear bits i through j in N. 
	// For example, i = 2, j = 4, the mask should be 11100011
	allOnes := ^0

	// 1s before position j, left = 11100000
	left := allOnes << uint(j+1)
	// 1s after position i, right = 00000011
	// We can get 011 by subtracting 1 from 100
	right := 1<<uint(i) - 1
	// Then the mask is 11100011
	mask := left | right

	// Clear bits i through j in N
	nCleared := n & int32(mask)
	return nCleared | (m << uint(i))
}
```

$$O(i+j)$$ time, $$O(1)$$ space.

## 2. Binary to String

Given a real number between 0 and 1 (e.g., 0.72) that is passed in as a double, print the binary representation. If the number cannot be represented accurately in binary with at most 32 characters, print "ERROR".

(1) 

For example, convert 0.6875 to binary:

![decimal to binary fixed point](img/decimal to binary fixed point.jpg)

```go
func BinaryToString(num float64) string {
   if num >= 1 || num < 0 {
      return "ERROR"
   }
   var b strings.Builder
   for num > 0 {
      if b.Len() > 32 {
         return "ERROR"
      }
      if tmp := num * 2; tmp >= 1 {
         b.WriteString("1")
         num = tmp - 1.0
      } else {
         b.WriteString("0")
         num = tmp
      }
   }
   return b.String()
}
```

$$O(n)$$ time, $$O(1)$$ space.

(2) 

Alternatively, rather than multiplying the number by two and comparing it to 1, we can compare the number to .5, then .25, and so on.

```go
func BinaryToString(num float64) string {
	if num >= 1 || num < 0 {
		return "ERROR"
	}
	var b strings.Builder
	frac := 0.5
	for num > 0 {
		if b.Len() > 32 {
			return "ERROR"
		}
		if num >= frac {
			b.WriteString("1")
			num -= frac
		} else {
			b.WriteString("0")
		}
		frac /= 2.0
	}
	return b.String()
}
```

## 3. Fit Bit to Win

You have an integer and you can flip exactly one bit from a 0 to a 1. Write code to find the length of the longest sequence of 1s you could create. 

```
EXAMPLE
Input: 1775 (or: 11011101111)
Output: 8
```


```go
func FitBitToWin(num int) int {
   if ^num == 0 {
      // All bits are 1.
      return int(reflect.TypeOf(num).Size() * 8)
   }
   cur, pre, max := 0, 0, 0
   for num != 0 {
      if num&1 == 1 {
         cur++
      } else {
         // If current bit is 0 then check the next (left) bit. 
         if next := num & 2; next == 1 {
            pre = cur
         } else {
            // If next bit is 0, then the sequence we've found can not 
            // merge with the next sequence we will find.
            pre = 0
         }
         cur = 0
      }
      if tmp := cur + pre; tmp > max {
         max = tmp
      }
      num >>= 1
   }
   // We can always have a sequence of
   // at least one 1, this is flipped bit
   return max + 1
}
```

$$O(n)$$ time, $$O(1)$$ time.