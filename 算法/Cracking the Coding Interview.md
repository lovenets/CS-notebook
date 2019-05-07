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





