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

