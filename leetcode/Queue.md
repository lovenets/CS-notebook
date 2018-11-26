#### 1.[Task Scheduler](https://leetcode.com/problems/task-scheduler)

Given a char array representing tasks CPU need to do. It contains capital letters A to Z where different letters represent different tasks.Tasks could be done without original order. Each task could be done in one interval. For each interval, CPU could finish one task or just be idle.

However, there is a non-negative cooling interval **n** that means between two **same tasks**, there must be at least n intervals that CPU are doing different tasks or just be idle.

You need to return the **least** number of intervals the CPU will take to finish all the given tasks.

**Example:**

```
Input: tasks = ["A","A","A","B","B","B"], n = 2
Output: 8
Explanation: A -> B -> idle -> A -> B -> idle -> A -> B.
```

**Note:**

1. The number of tasks is in the range [1, 10000].
2. The integer n is in the range [0, 100].

**Solution**

(1) 

> This solution is too smart to understand it...

First consider the most frequent characters, we can determine their relative positions first and use them as a frame to insert the remaining less frequent characters. Here is a proof by construction:

Let F be the set of most frequent chars with frequency k.
We can create k chunks, each chunk is identical and is a string consists of chars in F in a specific fixed order.
Let the heads of these chunks to be H_i; then H_2 should be at least n chars away from H_1, and so on so forth; then we insert the less frequent chars into the gaps between these chunks sequentially one by one ordered by frequency in a decreasing order and try to fill the k-1 gaps as full or evenly as possible each time you insert a character. **In summary, append the less frequent characters to the end of each chunk of the first k-1 chunks sequentially and round and round, then join the chunks and keep their heads' relative distance from each other to be at least n**.

Examples:

`AAAABBBEEFFGG 3`

here X represents a space gap:

```
Frame: "AXXXAXXXAXXXA"
insert 'B': "ABXXABXXABXXA" <--- 'B' has higher frequency than the other characters, insert it first.
insert 'E': "ABEXABEXABXXA"
insert 'F': "ABEFABEXABFXA" <--- each time try to fill the k-1 gaps as full or evenly as possible.
insert 'G': "ABEFABEGABFGA"
```

`AACCCBEEE 2`

```
3 identical chunks "CE", "CE CE CE" <-- this is a frame
insert 'A' among the gaps of chunks since it has higher frequency than 'B' ---> "CEACEACE"
insert 'B' ---> "CEABCEACE" <----- result is tasks.length;
```

`AACCCBBEEE 3`

```
3 identical chunks "CE", "CE CE CE" <--- this is a frame.
Begin to insert 'A'->"CEA CEA CE"
Begin to insert 'B'->"CEABCEABCE" <---- result is tasks.length;
```

`ACCCEEE 2`

```
3 identical chunks "CE", "CE CE CE" <-- this is a frame
Begin to insert 'A' --> "CEACE CE" <-- result is (c[25] - 1) * (n + 1) + 25 -i = 2 * 3 + 2 = 8
```

```java
// (c[25] - 1) * (n + 1) + 25 - i  is frame size
// when inserting chars, the frame might be "burst", then tasks.length takes precedence
// when 25 - i > n, the frame is already full at construction, the following is still valid.
public class Solution {
    public int leastInterval(char[] tasks, int n) {

        int[] c = new int[26];
        for(char t : tasks){
            c[t - 'A']++;
        }
        Arrays.sort(c);
        int i = 25;
        while(i >= 0 && c[i] == c[25]) i--;

        return Math.max(tasks.length, (c[25] - 1) * (n + 1) + 25 - i);
    }
}
```

Time Complexity: $$O(n)$$, n is the number of tasks.

(2)

The idea is:

0. To work on the same task again, CPU has to wait for time `n`, therefore we can think of as if there is a `cycle`, of `time n+1`, regardless whether you schedule some other task in the cycle or not.

1. To avoid leave the CPU with limited choice of tasks and having to sit there cooling down frequently at the end, it is critical the keep the diversity of the task pool for as long as possible.
2. In order to do that, we should try to schedule the CPU to `always try round robin between the most popular tasks at any time`.

```java
class Solution {
    public int leastInterval(char[] tasks, int n) {
        Map<Character, Integer> count = new HashMap<>();
        // calculate the frequencies of tasks
        for (char t : tasks) {
            count.put(t, count.getOrDefault(t, 0) + 1);
        }

        // descending by frequencies,
        // because we should schedule tasks
        // based on the most frequent task
        PriorityQueue<Integer> pq = new PriorityQueue<>((a, b) -> b - a);
        pq.addAll(count.values());

        int total = 0;
        int cycle = n + 1;
        while (!pq.isEmpty()) {
            int workTime = 0;
            List<Integer> frequencies = new ArrayList<>();
            // a cycle lasts (n + 1) interval
            // because there is at least n interval
            // between two same task
            for (int i = 0; i < cycle; i++) {
                if (!pq.isEmpty()) {
                    frequencies.add(pq.poll());
                    workTime++;
                }
            }
            // if c > 0, it means a specific kind of
            // task hasn't been accomplished yet
            for (int f : frequencies) {
                f--;
                if (f > 0) {
                    pq.offer(f);
                }
            }
            // if the queue is empty, that means we can finish all tasks
            // during a single cycle
            total += !pq.isEmpty() ? cycle : workTime;
        }

        return total;
    }
}
```

