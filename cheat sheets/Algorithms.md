## Sort

### Merge Sort

#### Definition

- A comparison based sorting algorithm
  - Divides entire dataset into groups of at most two.
  - Compares each number one at a time, moving the smallest number to left of the pair.
  - Once all pairs sorted it then compares left most elements of the two leftmost pairs creating a sorted group of four with the smallest numbers on the left and the largest ones on the right.
  - This process is repeated until there is only one set.

#### Key Points

- Know that it divides all the data into as small possible sets then compares them.
- Stable.
- Use extra space.

#### Big O Efficiency

- Best Case Sort: $$O(n)$$
- Average Case Sort: $$O(n log n)$$
- Worst Case Sort: $$O(nlog n)$$

#### Code Template

```go
func MergeSort(arr []int) []int {
    if n := len(arr); n < 2 {
        return arr
    } else {
        mid := n / 2
        return merge(MergeSort(arr[:mid]), MergeSort(arr[mid:]))
    }
}

func merge(arr1, arr2 []int) []int {
    n := len(arr1) + len(arr2)
    tmp := make([]int, n, n)
    for i, j, k := 0, 0, 0; k < n; k++ {
        if i >= len(arr1) && j < len(arr2) {
            tmp[k] = arr2[j]
            j++
        } else if j >= len(arr2) && i < len(arr1) {
            tmp[k] = arr1[i]
            i++
        } else if arr1[i] < arr2[j] {
            tmp[k] = arr1[i]
            i++
        } else {
            tmp[k] = arr2[j]
            j++
        }
    } 
    return tmp
} 
```

### Quicksort

#### Definition

- A comparison based sorting algorithm
  - Divides entire dataset in half by selecting the average element and putting all smaller elements to the left of the average.
  - It repeats this process on the left side until it is comparing only two elements at which point the left side is sorted.
  - When the left side is finished sorting it performs the same operation on the right side.
- Computer architecture favors the quicksort process.

#### Key Points

- While it has the same Big O as (or worse in some cases) many other sorting algorithms it is often faster in practice than many other sorting algorithms, such as merge sort.
- Know that it halves the data set by the average continuously until all the information is sorted.
- In place.
- Not stable.

#### Big O Efficiency

- Best Case Sort: $$O(log_{10}n)$$
- Average Case Sort: $$O(nlogn)$$
- Worst Case Sort: $$O(n^2)$$

#### Code Template

```go
func QuickSort(arr []int) {
    if len(arr) < 2 {
        return arr
    }
    left, right := 0, len(arr)-1
    pivot := rand.Intn(len(arr))
    // Move pivot to the end
    arr[pivot], arr[right] = arr[right], arr[pivot]
    for i  := range arr {
        if arr[i] < arr[right] {
            arr[left], arr[i] = arr[i], arr[left]
            left++
        }
    }
    // Move pivot to the correct postion
    arr[left], arr[right] = arr[right], arr[left]
    
    QuickSort(arr[:left])
    QuickSort(arr[left+1:])
    return a
}
```

### Heapsort

#### Definition

- A comparison based algorithm.
  -  It divides its input into a sorted and an unsorted region, and it iteratively shrinks the unsorted region by extracting the largest element and moving that to the sorted region.

#### Key Points

- The heapsort algorithm can be divided into two parts.
  - In the first step, a heap is built out of the data.
  - In the second step, a sorted array is created by repeatedly removing the largest element from the heap (the root of the heap), and inserting it into the array.
- In place.
- Not stable.

#### Big O Efficiency

- Best Case: $$O(n)$$
- Average Case: $$O(nlogn)$$
- Worst Case: $$O(nlogn)$$

#### Code Template

```go
func Heapsort(arr []int) []int {
    for i := len(arr)/2; i < len(arr); i++ {
        heapify(arr, i)
    }
    for i := len(arr)-1; i > 0; i-- {
        arr[0], arr[i] = arr[i], arr[0]
        heapify(arr[:i], 0)
    } 
    return arr
}

// heapify will build a max-heap
func heapify(arr []int, root int) []int {
    for parent, child := root, root*2+1; child < len(arr); child = parent*2+1 {
        if child+1 < len(arr) && arr[child] < arr[child+1] {
            child++
        }
        if arr[parent] > arr[child] {
            break
        }
        arr[parent], arr[child] = arr[child], arr[parent]
        parent = child
    }
}
```

### Summary

| ALGORITHM   | IN PLACE           | STABLE             | BEST         | AVERAGE      | WORST               |
| ----------- | ------------------ | ------------------ | ------------ | ------------ | ------------------- |
| Merge Sort  | :x:                | :heavy_check_mark: | $$O(n)$$     | $$O(nlogn)$$ | $$O(nlogn)$$        |
| Quicksort   | :heavy_check_mark: | :x:                | $$O(nlogn)$$ | $$O(nlogn)$$ | $$O(n^2)$$          |
| Heapsort    | :heavy_check_mark: | :x:                | $$O(n)$$     | $$O(nlogn)$$ | $$O(nlogn)$$        |
| Bubble Sort | :heavy_check_mark: | :heavy_check_mark: | $$O(n)$$     | $$O(n^2)$$   | $$O(n^2)$$          |
| Shellsort   | :heavy_check_mark: | :x:                | $$nlog_3n$$  | $$N/A$$      | $$O(n\frac{3}{2})$$ |

