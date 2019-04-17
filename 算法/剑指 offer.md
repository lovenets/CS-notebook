数组中重复的数字

### 1. 找出数组中重复的数字

在一个长度为 n 的数组里的所有数字都在 0~n-1 的范围内，数组中有些数字是重复的，但不知道有几个数字重复了，也不知道每个数字重复了几次。请找出数组中任意一个重复的数字。例如，如果输入长度为 7 的数组 `{2, 3, 1, 0, 2, 5, 3}`，那么对应的输出是重复的数字 2 或者 3。

#### 分析

- 可以先对数组进行排序，然后顺序扫描数组，判断相邻的数字是否相同。快速排序的时间复杂度是$$O(nlogn)$$，所以这个解法的时间复杂度是$$O(nlogn)$$；空间复杂度是$$O(logn)$$

- 可以利用哈希表，顺序扫描的过程中判断数字是否已经出现过。因为已经明确所有数字都在 0~n-1 范围内，所以可以直接用一个长度为 n 的数组作为哈希表，键就是下标，所以查找时间是$$O(1)$$，整个算法的时间复杂度是$$O(n)$$，空间复杂度是$$O(n)$$

可以根据排序的思想，用一个空间复杂度为$$O(1)$$的方法解决问题。如果数组有序，那么第一个位置上的数字是 0，第二个位置上的数字是 1，即`a[i] == i`。所以，顺序扫描数组，对于数字`a[i]`，判断`a[i] == i`，如果成立，说明这个数字位于正确的位置上，接着扫描下一个数字；如果不成立，判断`a[i] == a[a[i]]`，如果成立，就说明有重复，否则交换`a[i]`和`a[a[i]]`，直到`a[i]`位于正确的位置上。

```go
// 存在则返回一个重复的数字，其他情况返回 -1
func findDuplication1(numbers []int) int {
	// 数组为空
	if len(numbers) == 0 {
		return -1
	}
	// 数组中的数字不在 0~n-1 的范围内
	for _, n := range numbers {
		if n < 0 || n > len(numbers)-1 {
			return -1
		}
	}

	for i := range numbers {
		// 当前数字不在正确的位置上
		if numbers[i] != i {
			if numbers[i] == numbers[numbers[i]] {
				// 找到重复的数字
				return numbers[i]
			} else {
				// 将当前数字放到正确的位置上
				for numbers[i] != i {
					numbers[i], numbers[numbers[i]] = numbers[numbers[i]], numbers[i]
				}
			}
		}
	}
	return -1
}
```

每个数字最多交换两次就能位于正确的位置上，所以时间复杂度是$$O(n)$$，空间复杂度是$$O(1)$$。

### 2. 不修改数组中找出重复的数字

在一个长度为 n+1 的数组中，所有数字都在 1~n 的范围内，所以数组中至少有一个数字是重复的，请找出任意一个重复的数字，要求不能修改原数组。

#### 分析

如果不重复，那么 1~n 的范围内正好是每个数字一个。把 1~n 的数字从中间的数字 m 分为两部分，前面一半为 1~m，后面一半为 m+1~n。如果前面一半的数字个数超过 m，意味着这一个区间里面一定有重复的数；否则，就是后面一半这个区间里面有重复的数字。按照这种类似二分查找的方法，就可以找到重复的数字。

```go
func findDuplicatioin2(numbers []int) int {
	if len(numbers) == 0 {
		return -1
	}
	for _, n := range numbers {
		if n < 1 || n > len(numbers) {
			return -1
		}
	}
	start, end := 0, len(numbers)-1
	for start <= end {
		mid := (start + end) / 2
		count := countRange(numbers, start, mid)
		// 区间长度为1，也就是只需判断这个数在整个数组中出现了多少次
		// 就能直到它是否重复
		if end == start {
			if count > 1 {
				return start
			} else {
				break
			}
		}
		// 确定下一次查找的区间
		if count > mid-start+1 {
			end = mid
		} else {
			start = mid + 1
		}
	}
	return -1
}

func countRange(numbers []int, start, end int) int {
	if len(numbers) == 0 {
		return 0
	}
	count := 0
	for i := range numbers {
		if numbers[i] >= start && numbers[i] <= end {
			count++
		}
	}
	return count
}
```

`countRange`被调用$$logn$$次，它的时间复杂度是$$O(n)$$，因此整个算法时间复杂度是$$O(nlogn)$$，空间复杂度是$$O(1)$$。

## 二维数组中的查找

在一个二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序，请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。

### 分析

以下面的矩阵为例：

```
1 2 8  9
2 4 9  12
4 7 10 13
6 8 11 15
```

查找 7 的的过程为：
（1）首先看矩阵右上角的数字 9，9 大于 7，说明 9 所在的列不可能包含 7，所以把这一列去掉；

（2）在剩下的矩阵中同样是先看右上角的数字，也就是 8，8 也大于 7，这一列舍去；

（3）在剩下的矩阵中取右上角数字 2，2 小于 7，说明 7 不可能在这一行，所以把这一行去掉。

以此类推，当剩下的矩阵为：

```
4 7
6 8
```

这时右上角的数字就是 7。

也就说，每次都取当前矩阵右上角数字，如果不是所要找的数，那么就根据大小关系去掉一行或是一列；重复这个过程，直到找到目标或是整个矩阵都被去掉。

```go
func findInMatrix(matrix [][]int, target int) bool {
   if len(matrix) == 0 || len(matrix[0]) == 0 {
      return false
   }
   i, j := 0, len(matrix[0])-1
   for i < len(matrix) && j >= 0 {
      if matrix[i][j] == target {
         return true
      } else if matrix[i][j] < target {
         // 去掉一行
         i++
      } else {
         // 去掉一列    
         j--
      }
   }
   return false
}
```

最好的情况下只用找比较 1 次，最坏的情况下需要比较的次数为副对角线上的数字个数，所以时间复杂度为$$O(n)$$，空间复杂度是$$O(1)$$。

## 替换空格

请实现一个函数，把字符串中的每个空格替换成“%20”.

### 分析

可以直接使用库函数`strings.Replace`来实现，这里参考`strings.Replace`的源码给出一种实现。

```go
func Replace(s string) string {
	// Compute number of replacements.
	m := countBlank(s)
	if  m == 0 {
		return s // avoid allocation
	}
    
	// Calculate the length of string to be generated
    // and make a buffer.
	t := make([]byte, len(s)+m*(len("%20")-len(" ")))
	w := 0
	start := 0
	for i := 0; i < m; i++ {
        j := start
        // let j point to the first blank in the left string
        j += indexOfBlank(s[start:])
        // s[start:j] is the substring before a blank
        w += copy(t[w:], s[start:j])
        // let new string replace a blank
        w += copy(t[w:], "%20")
        // let start point to the place just after the blank replaced
        start = j + len(" ")
	}
    // copy the left substring
	w += copy(t[w:], s[start:])
	return string(t[0:w])
}

func countBlank(s string) int {
    if s == "" {
        return 0
    }
    count := 0
    for _, r := range s {
        if r == ' ' {
            count++
        }
    }
    return count
}

func indexOfBlank(s string) int {
    for i, r := range s {
        if r == ' ' {
            return i
        }
    }
    return -1
}
```

空间复杂度和源字符串以及替换的内容有关，时间复杂度是$$O(n)$$。





