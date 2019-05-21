# 最短路径

## Dijkstra 算法

特点：权重不为负的有向图；计算从源点到其余所有点的最短路径

下面是采用邻接表的代码模板：

```java
class Edge {
    private int from;

    private int to;

    private int weight;

	// ...
}

class Dijkstra {
    // 到某个顶点的最短路径长度
    private int[] dist;

    // 到某个顶点的最短路径上的最后一条边，用来求表示最短路径的边序列
    private Edge[] edgeTo;

    // 优先级队列，按照从源点到顶点的距离进行升序排序
    // 队列中的元素是键值对 <顶点，从源点到顶点的距离>
    private Queue<Map.Entry<Integer, Integer>> queue = new PriorityQueue<>(new Comparator<>() {
        @Override
        public int compare(Map.Entry<Integer, Integer> o1, Map.Entry<Integer, Integer> o2) {
            return o1.getValue() - o2.getValue();
        }
    });

    // graph 保存的是顶点和以该点为起点的有向边的对应关系
    // N 是顶点个数，注意如果顶点是从1开始编号的，那么 dist 和 edgeTo 的大小就是 N + 1
    public void dijkstra(Map<Integer, List<Edge>> graph, int N, int src) {
        dist = new int[N + 1];
        Arrays.fill(dist, Integer.MAX_VALUE);
        dist[src] = 0;
        edgeTo = new Edge[N + 1];

        // 开始 Dijkstra 算法
        queue.offer(Map.entry(src, 0));
        while (!queue.isEmpty()) {
            int f = queue.poll().getKey();
            if (!graph.containsKey(f)) {
                continue;
            }
            for (Edge e : graph.get(f)) {
                int t = e.getTo();
                if (dist[t] > dist[f] + e.getWeight()) {
                    int oldDist = dist[t];
                    dist[t] = dist[f] + e.getWeight();
                    edgeTo[t] = e;
                    // 更新队列
                    queue.remove(Map.entry(t, oldDist));
                    queue.offer(Map.entry(t, dist[t]));
                }
            }
        }
    }

    // 获取到某个点的最短路径长度
    public int distTo(int v) {
        return dist[v];
    }

    // 获取构成到某个点的最短路径的边序列
    public List<Edge> pathTo(int v) {
        if (dist[v] == Integer.MAX_VALUE) {
            return null;
        } else {
            List<Edge> path = new LinkedList<>();
            for (Edge e = edgeTo[v]; e != null; e = edgeTo[e.getFrom()]) {
                path.add(e);
            }
            return path;
        }
    }
}
```

## Floyd 算法

特点：可以解决权重为负的图，但是不能存在负环；计算每对顶点之间的最短路径

因为 Floyd 算法本身就利用了矩阵，所以直观的角度来看最好用邻接矩阵来表示图。

```java
class AdjMatrix {
    // 边的数量
    private int edgeNum;
    
    // 顶点的数量
    private int vertexNum;
    
    // 邻接矩阵，不可达定义为 Integer.MAX_VALUE
    private int[][] matrix;
    
    // ...
}

public class Floyd {
    // path[i][j] = k 表示从 i 到 j 的最短路径上有 k
    private int[][] path;
    
    // dist[i][j] 表示从 i 到 j 的最短路径长度
    private int[][] dist;
    
    public void floyd(int[][] adj) {
        // 初始化
        int N = adj.length;
        for (int i = 0; i < N; i++) {
            for (int j = 0; j < N; j++) {
                dist[i][j] = adj[i][j];
                path[i][j] = j;
            }
        }
        
        // Floyd 算法
        // 假设顶点是0~N-1，所以迭代 N 次
        for (int k = 0; k < N; k++) {
            for (int i = 0; i < N; i++) {
                for (int j = 0; j < N; j++) {
                    // 要防止 dist[i][k] 或 dist[k][j] 为 Integer.MAX_VALUE 而溢出
                    if (dist[i][k] < Integer.MAX_VALUE && dist[k][j] < Integer.MAX_VALUE && dist[i][j] > dist[i][k] + dist[k][j]) {
                        dist[i][j] = dist[i][k] + dist[k][j];
                        path[i][j] = k;
                    }
                }
            }
        }
    }
}
```

## Bellman-Ford 算法

特点：权重可为负，如果有负环虽不能求解，但是能检测到负环的存在；计算从源点到其余所有点的最短路径

这个算法实现起来比较简单，可以只存储边，而不需要邻接表或是邻接矩阵。

```java
class Egde {
    int from;
    
    int to;
    
    int weight;
    
    //...
}

public class BellmanFord {
    private int[] dist;
    
    // N 是顶点数
    public boolean sp(Edge[] edges, int N, int src) {
        // 初始化
        dist = new int[N];
        Arrays.fill(dist, Integer.MAX_VALUE);
        dist[src] = 0;
        
        // Bellman-Ford 算法
        // 对每条边放松 N - 1 次
        for (int i = 0; i < N - 1; i++) {
            for (Edge e : edges) {
                if (dist[e.to] > dist[e.from] + e.weight) {
                    dist[e.to] > dist[e.from] + e.weight;
                }
            }
        }
        // 检查是否有负环
        for (Edge e : edges) {
            if (dist[e.to] > dist[e.from] + e.weight) {
                return false;
            }
        }
        return true;
    }
}
```

## SPFA 算法

特点：权重可为负，如果有负环虽不能求解，但是能检测到负环的存在；计算从源点到其余所有点的最短路径

这里采用邻接表。

```java
class Egde {
    int from;
    
    int to;
    
    int weight;
    
    //...
}

public class SPFA {
    private int[] dist;
    
    // N 个顶点，1~N
    public boolean spfa(Map<Integer, List<Edge>> adjList, int N, int src) {
        // 初始化
        dist = new int[N];
        Arrays.fill(dist, Integer.MAX_VALUE);
        dist[src] = 0;
        
        // SPFA 算法
        Queue<Integer> q = new LinkedList<>();
        // count 用来记录顶点的入队次数
        int[] count = new int[N + 1];
        q.offer(src);
        count[src]++;
        while (!q.isEmpty()) {
            int f = q.poll();
            if (!adjList.containsKey(f)) {
                continue;
            }
            for (Edge e : adjList.get(f)) {
                int t = e.to;
                if (dist[t] > dist[f] + e.weight) {
                    dist[t] = dist[f] + e.weight;
                    if (!q.contains(t)) {
                        q.offer(t);
                        count[t]++;
                        // 某个顶点入队超过 N 次则存在负环
                        if (count[t] > N) {
                            return false;
                        }
                    }
                }
            }
        }
        return true;
    }
}
```

## 习题

### Emergency

1003 Emergency （25 point(s)）

As an emergency rescue team leader of a city, you are given a special map of your country. The map shows several scattered cities connected by some roads. Amount of rescue teams in each city and the length of each road between any pair of cities are marked on the map. When there is an emergency call to you from some other city, your job is to lead your men to the place as quickly as possible, and at the mean time, call up as many hands on the way as possible.

Input Specification:

Each input file contains one test case. For each test case, the first line contains 4 positive integers: *N* (≤500) - the number of cities (and the cities are numbered from 0 to *N*−1), *M* - the number of roads, *C*1 and *C*2 - the cities that you are currently in and that you must save, respectively. The next line contains *N* integers, where the *i*-th integer is the number of rescue teams in the *i*-th city. Then *M* lines follow, each describes a road with three integers *c*1, *c*2 and *L*, which are the pair of cities connected by a road and the length of that road, respectively. It is guaranteed that there exists at least one path from *C*1 to *C*2.

Output Specification:

For each test case, print in one line two numbers: the number of different shortest paths between *C*1 and *C*2, and the maximum amount of rescue teams you can possibly gather. All the numbers in a line must be separated by exactly one space, and there is no extra space allowed at the end of a line.

Sample Input:

```in
5 6 0 2
1 2 1 5 3
0 1 1
0 2 2
0 3 1
1 2 1
2 4 1
3 4 1
```

Sample Output:

```out
2 4
```

**解法**

这个题的特别之处就在于需要统计最短路径的条数；而且每个顶点也都有一个值，需要求出所有最短路径中顶点权重之和的最大值。

下面采用 SPFA 算法。

```java
import java.util.*;

public class Main {
    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        int n = in.nextInt();
        int m = in.nextInt();
        int src = in.nextInt();
        int dest = in.nextInt();
        int[] teams = new int[n];
        for (int i = 0; i < n; i++) {
            teams[i] = in.nextInt();
        }
        Map<Integer, List<Edge>> edges = new HashMap<>();
        for (int i = 0; i < m; i++) {
            int from = in.nextInt();
            int to = in.nextInt();
            int weight = in.nextInt();
            addEdge(edges, from, to, weight);
            addEdge(edges, to, from, weight);
        }
        // key：顶点，value：到达该顶点的各条最短路径上的前驱顶点
        Map<Integer, Set<Integer>> prev = new HashMap<>();
        
        int[] dist = new int[n];
        Arrays.fill(dist, Integer.MAX_VALUE);
        dist[src] = 0;
        // countPath[i]：到达顶点 i 的最短路径数量
        int[] countPath = new int[n];
        countPath[src] = 1;
        // countTeam[i]: 到达顶点 i 的最短路径中顶点权重之和的最大值
        int[] countTeam = new int[n];
        countTeam[src] = teams[src];
        spfa(src, edges, prev, dist, teams, countTeam, countPath);
        System.out.print(countPath[dest] + " ");
        System.out.println(countTeam[dest]);
    }

    private static void addEdge(Map<Integer, List<Edge>> edges, int from, int to, int weight) {
        if (!edges.containsKey(from)) {
            edges.put(from, new LinkedList<>());
        }
        edges.get(from).add(new Edge(to, weight));
    }

    private static void spfa(int src, Map<Integer, List<Edge>> edges, Map<Integer, Set<Integer>> prev, int[] dist, int[] teams, int[] countTeam, int[] countPath) {
        Queue<Integer> q = new LinkedList<>();
        q.offer(src);
        while (!q.isEmpty()) {
            int from = q.poll();
            if (!edges.containsKey(from)) {
                continue;
            }
            for (Edge e : edges.get(from)) {
                if (e.weight + dist[from] < dist[e.to]) {
                    // 找到一条更短路径
                    dist[e.to] = e.weight + dist[from];
                    countTeam[e.to] = countTeam[from] + teams[e.to];
                    countPath[e.to] = countPath[from];
                    // 清除之前记录的所有前驱顶点，因为之前的路径已经舍去了
                    prev.remove(e.to);
                    Set<Integer> set = new TreeSet<>();
                    set.add(from);
                    prev.put(e.to, set);
                    if (!q.contains(e.to)) {
                        q.offer(e.to);
                    }
                } else if (e.weight + dist[from] == dist[e.to]) {
                    // 发现一条一样长的最短路径
                    countTeam[e.to] = Math.max(countTeam[e.to], countTeam[from] + teams[e.to]);
                    // 添加前驱顶点
                    if (!prev.containsKey(e.to)) {
                        prev.put(e.to, new TreeSet<>());
                    }
                    prev.get(e.to).add(from);
                    // 因为在 SPFA 算法中一个顶点可能会被多次访问，所以这里需要重新计算
                    countPath[e.to] = 0;
                    for (int pre : prev.get(e.to)) {
                        countPath[e.to] += countPath[pre];
                    }
                }
            }
        }
    }
}

class Edge {
    int to;

    int weight;

    public Edge(int to, int weight) {
        this.to = to;
        this.weight = weight;
    }
}
```

部分正确。

从这个题可以看出，即使题目中添加了其他的要求，但只要主要的着眼点还是在求最短路径上，那么对于题目增加的要求就只需在松弛边的操作中增加新的判断，即判断是否找到了一条相同长度的路径。

### Public Bike Management

There is a public bike service in Hangzhou City which provides great convenience to the tourists from all over the world. One may rent a bike at any station and return it to any other stations in the city.

The Public Bike Management Center (PBMC) keeps monitoring the real-time capacity of all the stations. A station is said to be in **perfect** condition if it is exactly half-full. If a station is full or empty, PBMC will collect or send bikes to adjust the condition of that station to perfect. And more, all the stations on the way will be adjusted as well.

When a problem station is reported, PBMC will always choose the shortest path to reach that station. If there are more than one shortest path, the one that requires the least number of bikes sent from PBMC will be chosen.

![img](https://images.ptausercontent.com/213)

The above figure illustrates an example. The stations are represented by vertices and the roads correspond to the edges. The number on an edge is the time taken to reach one end station from another. The number written inside a vertex *S* is the current number of bikes stored at *S*. Given that the maximum capacity of each station is 10. To solve the problem at *S*3, we have 2 different shortest paths:

1. PBMC -> *S*1 -> *S*3. In this case, 4 bikes must be sent from PBMC, because we can collect 1 bike from *S*1 and then take 5 bikes to *S*3, so that both stations will be in perfect conditions.
2. PBMC -> *S*2 -> *S*3. This path requires the same time as path 1, but only 3 bikes sent from PBMC and hence is the one that will be chosen.

Input Specification:

Each input file contains one test case. For each case, the first line contains 4 numbers: Cmax (≤100), always an even number, is the maximum capacity of each station; *N* (≤500), the total number of stations; Sp, the index of the problem station (the stations are numbered from 1 to *N*, and PBMC is represented by the vertex 0); and *M*, the number of roads. The second line contains *N* non-negative numbers *Ci* (*i*=1,⋯,*N*) where each *Ci* is the current number of bikes at *Si* respectively. Then *M* lines follow, each contains 3 numbers: *Si*, *Sj*, and *T**i**j* which describe the time *T**i**j* taken to move betwen stations *S**i* and *S**j*. All the numbers in a line are separated by a space.

Output Specification:

For each test case, print your results in one line. First output the number of bikes that PBMC must send. Then after one space, output the path in the format: 0−>*S*1−>⋯−>*S**p*. Finally after another space, output the number of bikes that we must take back to PBMC after the condition of *S**p* is adjusted to perfect.

Note that if such a path is not unique, output the one that requires minimum number of bikes that we must take back to PBMC. The judge's data guarantee that such a path is unique.

Sample Input:

```in
10 3 3 5
6 7 0
0 1 1
0 2 1
0 3 3
1 3 1
2 3 1
```

Sample Output:

```out
3 0->2->3 0
```

**解法**

首先求出所有的最短路径，然后再用 DFS 确定应该选择哪一条。在开始前可以先将每个顶点的自行车数量减去$$Cmax / 2$$，这样就可以通过结果的正负来判断这个顶点是需要增加车辆还是可以从这个车站拿走一些。因此需要记录下每个顶点的需求量，还要记录下每离开一个顶点后从 PBMC 带来的车辆还剩多少。

注意本题中从起点出发后一路上就需要将沿途经过的所有顶点的状态调整为 perfect。



## 真题

### [20171204](http://118.190.20.162/view.page?gpid=T65)

问题描述

　　小明和小芳出去乡村玩，小明负责开车，小芳来导航。
　　小芳将可能的道路分为大道和小道。大道比较好走，每走1公里小明会增加1的疲劳度。小道不好走，如果连续走小道，小明的疲劳值会快速增加，连续走*s*公里小明会增加*s*2的疲劳度。
　　例如：有5个路口，1号路口到2号路口为小道，2号路口到3号路口为小道，3号路口到4号路口为大道，4号路口到5号路口为小道，相邻路口之间的距离都是2公里。如果小明从1号路口到5号路口，则总疲劳值为(2+2)2+2+22=16+2+4=22。
　　现在小芳拿到了地图，请帮助她规划一个开车的路线，使得按这个路线开车小明的疲劳度最小。

输入格式

　　输入的第一行包含两个整数*n*, *m*，分别表示路口的数量和道路的数量。路口由1至*n*编号，小明需要开车从1号路口到*n*号路口。
　　接下来*m*行描述道路，每行包含四个整数*t*, *a*, *b*, *c*，表示一条类型为*t*，连接*a*与*b*两个路口，长度为*c*公里的双向道路。其中*t*为0表示大道，*t*为1表示小道。保证1号路口和*n*号路口是连通的。

输出格式

　　输出一个整数，表示最优路线下小明的疲劳度。

样例输入

6 7
1 1 2 3
1 2 3 2
0 1 3 30
0 3 4 20
0 4 5 30
1 3 5 6
1 5 6 1

样例输出

76

样例说明

　　从1走小道到2，再走小道到3，疲劳度为52=25；然后从3走大道经过4到达5，疲劳度为20+30=50；最后从5走小道到6，疲劳度为1。总共为76。

数据规模和约定

　　对于30%的评测用例，1 ≤ *n* ≤ 8，1 ≤ *m* ≤ 10；
　　对于另外20%的评测用例，不存在小道；
　　对于另外20%的评测用例，所有的小道不相交；
　　对于所有评测用例，1 ≤ *n* ≤ 500，1 ≤ *m* ≤ 105，1 ≤ *a*, *b* ≤ *n*，*t*是0或1，*c* ≤ 105。保证答案不超过106。

**解法**

这个题和一般的最短路径问题的差别就在于计算路径的方法。对于相邻的两段路，有四种情况：大路-大路，小路-小路，大路-小路和小路-大路。既然是要最优化，自然是要使小路-小路的情况尽可能少出现。为了简化问题，可以先计算出在只允许走小路的情况下的最短路径，然后再一起考虑另外三种情况。

```java
public class Main {
    private static final int N = 510;

    private static final BigDecimal INF = BigDecimal.valueOf(10e18);

    // 走大路的情况
    private static BigDecimal[] dist0 = new BigDecimal[N];

    // 走小路的情况
    private static BigDecimal[] dist1 = new BigDecimal[N];

    // 用大路连通的邻接矩阵
    private static BigDecimal[][] adj0 = new BigDecimal[N][N];

    // 用小路连通的邻接矩阵
    private static BigDecimal[][] adj1 = new BigDecimal[N][N];

    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        int n = in.nextInt();
        int m = in.nextInt();
        for (int i = 0; i <= n; i++) {
            for (int j = 0; j <= n; j++) {
                adj0[i][j] = INF;
                adj1[i][j] = INF;
            }
        }
        for (int i = 0; i < m; i++) {
            int t = in.nextInt();
            int a = in.nextInt();
            int b = in.nextInt();
            BigDecimal c = BigDecimal.valueOf(in.nextInt());
            if (t == 1 && adj1[a][b].compareTo(c) > 0) {
                adj1[a][b] = c;
                adj1[b][a] = c;
            }
            if (t == 0 && adj0[a][b].compareTo(c) > 0) {
                adj0[a][b] = c;
                adj0[b][a] = c;
            }
        }

        // 先计算出小路->小路的情况
        for (int k = 1; k <= n; k++) {
            for (int i = 1; i <= n; i++) {
                for (int j = 1; j <= n; j++) {
                    if (adj1[i][j].compareTo(adj1[i][k].add(adj1[k][j])) > 0) {
                        adj1[i][j] = adj1[i][k].add(adj1[k][j]);
                    }
                }
            }
        }
        spfa(n);
        if (dist0[n].compareTo(dist1[n]) < 0) {
            System.out.println(dist0[n]);
        } else {
            System.out.println(dist1[n]);
        }
    }

    private static void spfa(int n) {
        Queue<Integer> q = new LinkedList<>();
        q.offer(1);
        Arrays.fill(dist0, INF);
        dist0[1] = BigDecimal.ZERO;
        Arrays.fill(dist1, INF);
        dist1[1] = BigDecimal.ZERO;
        while (!q.isEmpty()) {
            int u = q.poll();
            for (int i = 1; i <= n; i++) {
                // 大路->大路
                highway(q, u, i, dist0);
                // 小路->大路
                highway(q, u, i, dist1);
                // 大路->小路
                trail(q, u, i);
            }
        }
    }

    private static void trail(Queue<Integer> q, int u, int i) {
        if (adj1[u][i].compareTo(INF) < 0) {
            if (dist1[i].compareTo(dist0[u].add(adj1[u][i].multiply(adj1[u][i]))) > 0) {
                dist1[i] = dist0[u].add(adj1[u][i].multiply(adj1[u][i]));
                if (!q.contains(i)) {
                    q.offer(i);
                }
            }
        }
    }

    private static void highway(Queue<Integer> q, int u, int i, BigDecimal[] dist) {
        if (dist[i].compareTo(dist[u].add(adj0[u][i])) > 0) {
            dist[i] = dist0[u].add(adj0[u][i]);
            if (!q.contains(i)) {
                q.offer(i);
            }
        }
    }
}
```

超时，30分。Floyd 算法太慢。

### 20160904

问题描述

　　G国国王来中国参观后，被中国的高速铁路深深的震撼，决定为自己的国家也建设一个高速铁路系统。
　　建设高速铁路投入非常大，为了节约建设成本，G国国王决定不新建铁路，而是将已有的铁路改造成高速铁路。现在，请你为G国国王提供一个方案，将现有的一部分铁路改造成高速铁路，使得任何两个城市间都可以通过高速铁路到达，而且从所有城市乘坐高速铁路到首都的最短路程和原来一样长。请你告诉G国国王在这些条件下最少要改造多长的铁路。

输入格式

　　输入的第一行包含两个整数*n*, *m*，分别表示G国城市的数量和城市间铁路的数量。所有的城市由1到*n*编号，首都为1号。
　　接下来*m*行，每行三个整数*a*, *b*, *c*，表示城市*a*和城市*b*之间有一条长度为*c*的双向铁路。这条铁路不会经过*a*和*b*以外的城市。

输出格式

　　输出一行，表示在满足条件的情况下最少要改造的铁路长度。

样例输入

4 5
1 2 4
1 3 5
2 3 2
2 4 3
3 4 2

样例输出

11

评测用例规模与约定

　　对于20%的评测用例，1 ≤ *n* ≤ 10，1 ≤ *m* ≤ 50；
　　对于50%的评测用例，1 ≤ *n* ≤ 100，1 ≤ *m* ≤ 5000；
　　对于80%的评测用例，1 ≤ *n* ≤ 1000，1 ≤ *m* ≤ 50000；
　　对于100%的评测用例，1 ≤ *n* ≤ 10000，1 ≤ *m* ≤ 100000，1 ≤ *a*, *b* ≤ n，1 ≤ *c* ≤ 1000。输入保证每个城市都可以通过铁路达到首都。

**解法**

“从所有城市乘坐高速铁路到首都的最短路程和原来一样长”这个条件显然是要求最短路径，“最少要改造多长的铁路”则是要求生成最小子树，所以这是一个最小生成树和最短路径的综合题。在常规的最短路径算法中，松弛边的条件是$$d[v]>d[u]+w(u,v)$$，此时只需增加一个判断：当$$d[v]==d[u]+w(u,v)$$时判断当前到达$$v$$的路径上的最后一条边的权重是否大于边$$(u,v)$$的权重，如果大于则松弛。这样就能保证在求得最短路径的前提下能生成最小子树。

```java
import java.math.BigDecimal;
import java.util.*;

public class Main {
    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        int n = in.nextInt();
        int m = in.nextInt();
        Map<Integer, List<Edge>> g = new HashMap<>();
        for (int i = 0; i < m; i++) {
            int a = in.nextInt();
            int b = in.nextInt();
            int c = in.nextInt();
            addEdge(g, a, b, c);
            addEdge(g, b, a, c);
        }

        // SPFA 算法
        BigDecimal[] dist = new BigDecimal[n + 1];
        Arrays.fill(dist, BigDecimal.valueOf(1e10));
        dist[1] = BigDecimal.ZERO;
        Queue<Integer> q = new LinkedList<>();
        q.offer(1);
        Edge[] edgeTo = new Edge[n + 1];
        while (!q.isEmpty()) {
            int u = q.poll();
            if (!g.containsKey(u)) {
                continue;
            }
            for (Edge e : g.get(u)) {
                int v = e.to;
                BigDecimal cand = dist[u].add(BigDecimal.valueOf(e.weight));
                if (dist[v].compareTo(cand) > 0) {
                    dist[v] = cand;
                    edgeTo[v] = e;
                    if (!q.contains(v)) {
                        q.offer(v);
                    }
                }
                // 确保生成最小子树
                if (dist[v].equals(cand) && edgeTo[v].weight > e.weight) {
                    edgeTo[v] = e;
                }
            }
        }

        BigDecimal ans = BigDecimal.ZERO;
        // 选择每条最短路径上的所有边
        for (int i = 2; i <= n; i++) {
            for (Edge e = edgeTo[i]; e != null; e = edgeTo[e.from]) {
                if (!e.in) {
                    e.in = true;
                    ans = ans.add(BigDecimal.valueOf(e.weight));
                }
            }
        }
        System.out.println(ans);
    }

    private static void addEdge(Map<Integer, List<Edge>> g, int f, int t, int w) {
        if (!g.containsKey(f)) {
            g.put(f, new LinkedList<>());
        }
        g.get(f).add(new Edge(f, t, w));
    }
}


class Edge {
    int from;

    int to;

    int weight;

    boolean in = false;

    public Edge(int from, int to, int weight) {
        this.from = from;
        this.to = to;
        this.weight = weight;
    }
}
```

超时，80分。

# BFS & DFS

## BFS

采用邻接表表示图。

```java
class Edge {
    int from;
    
    int to;
    
    int weight;
    
    //...
}

public class BFS {
    // 顶点编号 1~N
    public void bfs(Map<Integer, List<Edge>> g, int N, int src) {
        boolean[] visited = new boolean[N + 1];
        Queue<Integer> q = new LinkedList<>();
        q.offer(src);
        visited[src] = true;
        while (!q.isEmpty()) {
            int u = q.poll();
            if (g.containsKey(u)) {
                for (Edge e : g.get(u)) {
                    if (!visited[e.to]) {
                        q.offer(e.to);
                        // 注意是入队的时候就做标记
                        visited[e.to] = true;
                    }
                }
            }
        }
    }
} 
```

**注意**，BFS 也可以用来解决某些特殊的最短路径问题。BFS 的搜索过程可以看作是以起点为中心不断向外发散，所以当搜索到目的顶点时，此时经过的路径长度一定是最短的。当然，这里的“最短”建立在所有边长权重相同的基础上。

## DFS

采用邻接表表示图。

```java
class Edge {
    int from;
    
    int to;
    
    int weight;
    
    //...
}

public class DFS {
    
    private boolean[] visited;
    
    // 顶点编号 1~N
    public void dfs(Map<Integer, List<Edge>> g, int N, int src) {
        visited = new boolean[N + 1];
        helper(g, src);
    }
    
    private void helper(Map<Integer, List<Edge>> g, int u) {
        if (!visited[u]) {
            visited[u] = true;
            if (g.containsKey(u)) {
                for (Edge e : g.get(u)) {
                    helper(g, e.to);
                }
            }
        }
    }
} 
```

## 习题

### Integer Fractorization

![img](https://img-blog.csdnimg.cn/2019020222355917.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0luTm9WYWlvbl95dQ==,size_16,color_FFFFFF,t_70)

**解法**

```java
import java.util.*;

public class Main {
    // 候选的底数
    private static List<Integer> cand = new LinkedList<>();

    private static Integer[] ans;

    private static List<Integer> tmp = new LinkedList<>();

    private static int n;

    private static int k;

    private static int maxSum = -1;

    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        n = in.nextInt();
        k = in.nextInt();
        int p = in.nextInt();
        // 初始化 cand，cand[i] = i^p
        for (int i = 0, tmp = 0; tmp <= n; tmp = (int) Math.pow(i, p)) {
            cand.add(tmp);
            i++;
        }
        // 为了保证字典序最大，从最大的候选底数开始搜索
        dfs(cand.size() - 1, 0, 0);
        if (maxSum == -1) {
            System.out.println("Impossible");
        } else {
            StringBuilder b = new StringBuilder();
            b.append(n).append(" = ");
            for (int f : ans) {
                b.append(f).append("^").append(p).append(" + ");
            }
            System.out.println(b.substring(0, b.length() - 3));
        }
    }

    private static void dfs(int i, int sum, int facSum) {
        if (sum == n && tmp.size() == k) {
            // 找到一组各项底数之和更大的更优解
            if (facSum > maxSum) {
                ans = tmp.toArray(new Integer[0]);
                maxSum = facSum;
            }
            return;
        }
        // 无解
        if (sum > n || tmp.size() > k) {
            return;
        }
        if (i > 0) {
            tmp.add(i);
            // 一个数可以使用多次，所以继续从这个数开始搜索
            dfs(i,sum + cand.get(i), facSum + i);
            // 刚才选择的数无法构成解，所以删除它，然后从下一个数开始搜索
            tmp.remove(tmp.size() - 1);
            dfs(i - 1, sum, facSum);
        }
    }
}
```

### Acute Stroke

One important factor to identify acute stroke (急性脑卒中) is the volume of the stroke core. Given the results of image analysis in which the core regions are identified in each MRI slice, your job is to calculate the volume of the stroke core.

Input Specification:     

​    Each input file contains one test case. For each case, the first line contains 4 positive integers: *M*, *N*, *L* and *T*, where *M* and *N* are the sizes of each slice (i.e. pixels of a slice are in an *M*×*N* matrix, and the maximum resolution is 1286 by 128); *L* (≤60) is the number of slices of a brain; and *T* is the integer threshold (i.e. if the volume of a connected core is less than *T*, then that core must not be counted).

​    Then *L* slices are given. Each slice is represented by an *M*×*N* matrix of 0’s and 1’s, where 1 represents a pixel of stroke, and 0 means normal. Since the thickness of a slice is a constant, we only have to count the number of 1’s to obtain the volume. However, there might be several separated core regions in a brain, and only those with their volumes no less than *T* are counted. Two pixels are **connected** and hence belong to the same region if they share a common side, as shown by Figure 1 where all the 6 red pixels are connected to the blue one.

![img](http://valcanoshan.cn/wp-content/uploads/2019/02/20190223192339_68410.jpg)

Figure 1

Output Specification: 

​    For each case, output in a line the total volume of the stroke core.

Sample Input: 

```
3 4 5 2
1 1 1 1
1 1 1 1
1 1 1 1
0 0 1 1
0 0 1 1
0 0 1 1
1 0 1 1
0 1 0 0
0 0 0 0
1 0 1 1
0 0 0 0
0 0 0 0
0 0 0 1
0 0 0 1
1 0 0 0
```

Sample Output: 

```
26
```

**解法**

典型的连通域问题，应该用 BFS 解决。注意这是在一个是三维空间进行 BFS，每一步可移动的方向除了常规的四个方向以外还有向上一层或是向下一层。同时注意题目要求的是统计所有连通域中1的个数。

```java
import java.util.LinkedList;
import java.util.Queue;
import java.util.Scanner;

public class Main {

    private static int m;

    private static int n;

    private static int l;

    private static int T;

    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        m = in.nextInt();
        n = in.nextInt();
        l = in.nextInt();
        T = in.nextInt();
        int[][][] slices = new int[l][m][n];
        for (int i = 0; i < l; i++) {
            int[][] slice = new int[m][n];
            for (int j = 0; j < m; j++) {
                for (int k = 0; k < n; k++) {
                    slice[j][k] = in.nextInt();
                }
            }
            slices[i] = slice;
        }
        System.out.println(bfs(slices));
    }

    private static int bfs(int[][][] slices) {
        int[][] dir = new int[][]{
                {1, 0, 0},
                {-1, 0, 0},
                {0, 1, 0},
                {0, -1, 0},
                {0, 0, 1},
                {0, 0, -1}
        };
        boolean[][][] visited = new boolean[l][m][n];
        int total = 0;
        for (int z = 0; z < l; z++) {
            for (int x = 0; x < m; x++) {
                for (int y = 0; y < n; y++) {
                    // 从某个未访问的1开始 BFS
                    if (!visited[z][x][y] && (slices[z][x][y] == 1)) {
                        int count = 0;
                        Queue<int[]> q = new LinkedList<>();
                        q.offer(new int[]{z, x, y});
                        visited[z][x][y] = true;
                        count++;
                        while (!q.isEmpty()) {
                            int[] head = q.poll();
                            for (int i = 0; i < 6; i++) {
                                int curZ = head[0] + dir[i][0];
                                int curX = head[1] + dir[i][1];
                                int curY = head[2] + dir[i][2];
                                if (curZ >= 0 && curZ < l
                                        && curX >= 0 && curX < m
                                        && curY >= 0 && curY < n
                                        && !visited[curZ][curX][curY]
                                        && (slices[curZ][curX][curY] == 1)) {
                                    q.offer(new int[]{curZ, curX, curY});
                                    visited[curZ][curX][curY] = true;
                                    count++;
                                }
                            }
                        }
                        if (count >= T) {
                            total += count;
                        }
                    }
                }
            }
        }
        return total;
    }
}
```



## 真题

### [20170904](http://118.190.20.162/view.page?gpid=T60)

问题描述

　　某国的军队由*N*个部门组成，为了提高安全性，部门之间建立了*M*条通路，每条通路只能单向传递信息，即一条从部门*a*到部门*b*的通路只能由*a*向*b*传递信息。信息可以通过中转的方式进行传递，即如果*a*能将信息传递到*b*，*b*又能将信息传递到*c*，则*a*能将信息传递到*c*。一条信息可能通过多次中转最终到达目的地。
　　由于保密工作做得很好，并不是所有部门之间都互相知道彼此的存在。只有当两个部门之间可以直接或间接传递信息时，他们才彼此知道对方的存在。部门之间不会把自己知道哪些部门告诉其他部门。
![img](http://118.190.20.162/RequireFile.do?fid=yHg9gf9q)
　　上图中给了一个4个部门的例子，图中的单向边表示通路。部门1可以将消息发送给所有部门，部门4可以接收所有部门的消息，所以部门1和部门4知道所有其他部门的存在。部门2和部门3之间没有任何方式可以发送消息，所以部门2和部门3互相不知道彼此的存在。
　　现在请问，有多少个部门知道所有*N*个部门的存在。或者说，有多少个部门所知道的部门数量（包括自己）正好是*N*。

输入格式

　　输入的第一行包含两个整数*N*, *M*，分别表示部门的数量和单向通路的数量。所有部门从1到*N*标号。
　　接下来*M*行，每行两个整数*a*, *b*，表示部门*a*到部门*b*有一条单向通路。

输出格式

　　输出一行，包含一个整数，表示答案。

样例输入

4 4
1 2
1 3
2 4
3 4

样例输出

2

样例说明

　　部门1和部门4知道所有其他部门的存在。

评测用例规模与约定

　　对于30%的评测用例，1 ≤ *N* ≤ 10，1 ≤ *M* ≤ 20；
　　对于60%的评测用例，1 ≤ *N* ≤ 100，1 ≤ *M* ≤ 1000；
　　对于100%的评测用例，1 ≤ *N* ≤ 1000，1 ≤ *M* ≤ 10000。

**解法**

这道题的意思就是从用多少个顶点满足从它出发能遍历整个图。但是注意题目说不能主动发送消息的部门能够通过接收消息得知其他点的存在，如果这是按照原图去找的话就找不到这样的部门。还要构造一个所有边的方向都相反的图进行遍历。

```java
public class Main {
    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        int n = in.nextInt();
        int m = in.nextInt();
        Map<Integer, List<Integer>> g1 = new HashMap<>();
        Map<Integer, List<Integer>> g2 = new HashMap<>();
        for (int i = 0; i < m; i++) {
            int a = in.nextInt();
            int b = in.nextInt();
            if (!g1.containsKey(a)) {
                g1.put(a, new LinkedList<>());
            }
            g1.get(a).add(b);
            if (!g2.containsKey(b)) {
                g2.put(b, new LinkedList<>());
            }
            g2.get(b).add(a);
        }
        int ans = 0;
        for (int i = 1; i <= n; i++) {
            boolean[] visited1 = new boolean[1010];
            bfs(g1, i, visited1);
            boolean[] visited2 = new boolean[1010];
            bfs(g2, i, visited2);
            int count = 0;
            for (int j = 1; j <= n; j++) {
                // 正反两次有一次满足就行，如果分开算那么在有环的情况下就会算重复了
                if (visited1[j] || visited2[j]) {
                    count++;
                }
            }
            if (count == n) {
                ans++;
            }
        }
        System.out.println(ans);
    }

    private static void bfs(Map<Integer, List<Integer>> g, int src, boolean[] visited) {
        Queue<Integer> q = new LinkedList<>();
        q.offer(src);
        visited[src] = true;
        while (!q.isEmpty()) {
            int u = q.poll();
            if (g.containsKey(u)) {
                for (int v : g.get(u)) {
                    if (!visited[v]) {
                        q.offer(v);
                        visited[v] = true;
                    }
                }
            }
        }
    }
}
```

超时，95

### 20150904

问题描述

　　某国有*n*个城市，为了使得城市间的交通更便利，该国国王打算在城市之间修一些高速公路，由于经费限制，国王打算第一阶段先在部分城市之间修一些单向的高速公路。
　　现在，大臣们帮国王拟了一个修高速公路的计划。看了计划后，国王发现，有些城市之间可以通过高速公路直接（不经过其他城市）或间接（经过一个或多个其他城市）到达，而有的却不能。如果城市A可以通过高速公路到达城市B，而且城市B也可以通过高速公路到达城市A，则这两个城市被称为便利城市对。
　　国王想知道，在大臣们给他的计划中，有多少个便利城市对。

输入格式

　　输入的第一行包含两个整数*n*, *m*，分别表示城市和单向高速公路的数量。
　　接下来*m*行，每行两个整数*a*, *b*，表示城市*a*有一条单向的高速公路连向城市*b*。

输出格式

　　输出一行，包含一个整数，表示便利城市对的数量。

样例输入

5 5
1 2
2 3
3 4
4 2
3 5

样例输出

3

样例说明

![img](http://118.190.20.162/RequireFile.do?fid=4HG9GgbF)
　　城市间的连接如图所示。有3个便利城市对，它们分别是(2, 3), (2, 4), (3, 4)，请注意(2, 3)和(3, 2)看成同一个便利城市对。

评测用例规模与约定

　　前30%的评测用例满足1 ≤ *n* ≤ 100, 1 ≤ *m* ≤ 1000；
　　前60%的评测用例满足1 ≤ *n* ≤ 1000, 1 ≤ *m* ≤ 10000；
　　所有评测用例满足1 ≤ *n* ≤ 10000, 1 ≤ *m* ≤ 100000。

**解法**

最直接的做法自然就是依次从每个顶点出发进行搜索，检查能到达哪个顶点。

```java
import java.math.BigDecimal;
import java.util.*;

public class Main {

    private static Map<Integer, List<Integer>> adj = new HashMap<>();

    // bi[i][j] == 1 表示可从 i 到达 j
    private static int[][] bi;

    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        int n = in.nextInt();
        int m = in.nextInt();
        for (int i = 0; i < m; i++) {
            int a = in.nextInt();
            int b = in.nextInt();
            if (!adj.containsKey(a)) {
                adj.put(a, new LinkedList<>());
            }
            adj.get(a).add(b);
        }
        bi = new int[n + 1][n + 1];
        for (int i = 1; i <= n; i++) {
            if (adj.containsKey(i)) {
                boolean[] visited = new boolean[n + 1];
                visited[i] = true;
                for (int u : adj.get(i)) {
                    dfs(i, u, visited);
                }
            }
        }
        BigDecimal ans = BigDecimal.ZERO;
        // 只遍历下三角，否则会重复计算
        for (int i = 1; i <= n; i++) {
            for (int j = 1; j < i; j++) {
                if (bi[i][j] == 1 && bi[j][i] == 1) {
                    ans = ans.add(BigDecimal.ONE);
                }
            }
        }
        System.out.println(ans);
    }

    private static void dfs(int src, int cur, boolean[] visited) {
        if (!visited[cur]) {
            visited[cur] = true;
            bi[src][cur] = 1;
            if (adj.containsKey(cur)) {
                for (int next : adj.get(cur)) {
                    dfs(src, next, visited);
                }
            }
        }
    }
}
```

超时，50分。

### 20150304

问题描述

　　给定一个公司的网络，由*n*台交换机和*m*台终端电脑组成，交换机与交换机、交换机与电脑之间使用网络连接。交换机按层级设置，编号为1的交换机为根交换机，层级为1。其他的交换机都连接到一台比自己上一层的交换机上，其层级为对应交换机的层级加1。所有的终端电脑都直接连接到交换机上。
　　当信息在电脑、交换机之间传递时，每一步只能通过自己传递到自己所连接的另一台电脑或交换机。请问，电脑与电脑之间传递消息、或者电脑与交换机之间传递消息、或者交换机与交换机之间传递消息最多需要多少步。

输入格式

　　输入的第一行包含两个整数*n*, *m*，分别表示交换机的台数和终端电脑的台数。
　　第二行包含*n* - 1个整数，分别表示第2、3、……、*n*台交换机所连接的比自己上一层的交换机的编号。第*i*台交换机所连接的上一层的交换机编号一定比自己的编号小。
　　第三行包含*m*个整数，分别表示第1、2、……、*m*台终端电脑所连接的交换机的编号。

输出格式

　　输出一个整数，表示消息传递最多需要的步数。

样例输入

4 2
1 1 3
2 1

样例输出

4

样例说明

　　样例的网络连接模式如下，其中圆圈表示交换机，方框表示电脑：
![img](http://118.190.20.162/RequireFile.do?fid=F9GfBRHL)
　　其中电脑1与交换机4之间的消息传递花费的时间最长，为4个单位时间。

**解法**

不论是主机还是交换机都看作一样的顶点，于是这个题就变成了要求寻找图中距离最远的两个顶点之间的距离。从每个顶点出发进行 DFS，比较最大深度。

```java
import java.math.BigDecimal;
import java.util.*;

public class Main {
    private static Map<Integer, List<Integer>> adj = new HashMap<>();

    private static BigDecimal maxLen = BigDecimal.ZERO;

    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        int n = in.nextInt();
        int m = in.nextInt();
        for (int u = 2; u <= n + m; u++) {
            int v = in.nextInt();
            addEdge(u, v);
            addEdge(v, u);
        }
        for (int i = 1; i <= n + m; i++) {
            dfs(i, BigDecimal.ZERO, new boolean[n + m + 1]);
        }
        System.out.println(maxLen);
    }

    private static void dfs(int u, BigDecimal curLen, boolean[] visited) {
        visited[u] = true;
        if (adj.containsKey(u)) {
            for (int v : adj.get(u)) {
                if (!visited[v]) {
                    dfs(v, curLen.add(BigDecimal.ONE), visited);
                }
                if (maxLen.compareTo(curLen) < 0) {
                    maxLen = curLen;
                }
            }
        }
    }

    private static void addEdge(int u, int v) {
        if (!adj.containsKey(u)) {
            adj.put(u, new LinkedList<>());
        }
        adj.get(u).add(v);
    }
}
```

超时，70分。

### 20140904

问题描述

　　栋栋最近开了一家餐饮连锁店，提供外卖服务。随着连锁店越来越多，怎么合理的给客户送餐成为了一个急需解决的问题。
　　栋栋的连锁店所在的区域可以看成是一个n×n的方格图（如下图所示），方格的格点上的位置上可能包含栋栋的分店（绿色标注）或者客户（蓝色标注），有一些格点是不能经过的（红色标注）。
　　方格图中的线表示可以行走的道路，相邻两个格点的距离为1。栋栋要送餐必须走可以行走的道路，而且不能经过红色标注的点。

![img](http://118.190.20.162/RequireFile.do?fid=383qHJjQ)
　　送餐的主要成本体现在路上所花的时间，每一份餐每走一个单位的距离需要花费1块钱。每个客户的需求都可以由栋栋的任意分店配送，每个分店没有配送总量的限制。
　　现在你得到了栋栋的客户的需求，请问在最优的送餐方式下，送这些餐需要花费多大的成本。

输入格式

　　输入的第一行包含四个整数n, m, k, d，分别表示方格图的大小、栋栋的分店数量、客户的数量，以及不能经过的点的数量。
　　接下来m行，每行两个整数xi, yi，表示栋栋的一个分店在方格图中的横坐标和纵坐标。
　　接下来k行，每行三个整数xi, yi, ci，分别表示每个客户在方格图中的横坐标、纵坐标和订餐的量。（注意，可能有多个客户在方格图中的同一个位置）
　　接下来d行，每行两个整数，分别表示每个不能经过的点的横坐标和纵坐标。

输出格式

　　输出一个整数，表示最优送餐方式下所需要花费的成本。

样例输入

10 2 3 3
1 1
8 8
1 5 1
2 3 3
6 7 2
1 2
2 2
6 8

样例输出

29

评测用例规模与约定

　　前30%的评测用例满足：1<=n <=20。
　　前60%的评测用例满足：1<=n<=100。
　　所有评测用例都满足：1<=n<=1000，1<=m, k, d<=n^2。可能有多个客户在同一个格点上。每个客户的订餐量不超过1000，每个客户所需要的餐都能被送到。

**解法**

为了让每个商店都优先服务离自己近的客户，应该使用 BFS。这题是典型的单位长度相同的情形下求最短路径的问题。

```java
import java.math.BigDecimal;
import java.util.LinkedList;
import java.util.Queue;
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        int n = in.nextInt();
        int m = in.nextInt();
        int k = in.nextInt();
        int d = in.nextInt();
        Queue<Store> q = new LinkedList<>();
        for (int i = 0; i < m; i++) {
            q.offer(new Store(in.nextInt(), in.nextInt(), 0));
        }
        // order[x][y] 表示该位置的订单数
        int[][] order = new int[n + 1][n + 1];
        int x, y, o;
        for (int i = 0; i < k; i++) {
            x = in.nextInt();
            y = in.nextInt();
            o = in.nextInt();
            // 注意同一个位置可以有多个客户
            order[x][y] += o;
        }
        boolean[][] visited = new boolean[n + 1][n + 1];
        for (int i = 0; i < d; i++) {
            x = in.nextInt();
            y = in.nextInt();
            visited[x][y] = true;
        }

        BigDecimal ans = BigDecimal.ZERO;
        int[][] move = new int[][]{{1, 0}, {-1, 0}, {0, 1}, {0, -1}};
        int x1, y1;
        while (!q.isEmpty()) {
            Store s = q.poll();
            // 配送到某个点的代价 = 该点的订单数 * 从商店走到这个点的步数
            ans = ans.add(BigDecimal.valueOf(order[s.x][s.y] * s.d));
            visited[s.x][s.y] = true;
            // BFS 保证每个店都会优先配送离自己最近的客户
            for (int[] dir : move) {
                x1 = dir[0] + s.x;
                y1 = dir[1] + s.y;
                if (x1 >= 1 && x1 <= n && y1 >= 1 && y1 <= n && !visited[x1][y1]) {
                    q.offer(new Store(x1, y1, s.d + 1));
                    visited[x1][y1] = true;
                }
            }
        }
        System.out.println(ans);
    }
}

class Store {
    int x;

    int y;

    // 商店到达某个格点需要走的步数
    int d;

    public Store(int x, int y, int d) {
        this.x = x;
        this.y = y;
        this.d = d;
    }
}
```

超时，80分。

### 20140304

问题描述

　　目前在一个很大的平面房间里有 n 个无线路由器,每个无线路由器都固定在某个点上。任何两个无线路由器只要距离不超过 r 就能互相建立网络连接。
　　除此以外,另有 m 个可以摆放无线路由器的位置。你可以在这些位置中选择至多 k 个增设新的路由器。
　　你的目标是使得第 1 个路由器和第 2 个路由器之间的网络连接经过尽量少的中转路由器。请问在最优方案下中转路由器的最少个数是多少?

输入格式

　　第一行包含四个正整数 n,m,k,r。(2 ≤ n ≤ 100,1 ≤ k ≤ m ≤ 100, 1 ≤ r ≤ 108)。
　　接下来 n 行,每行包含两个整数 xi 和 yi,表示一个已经放置好的无线 路由器在 (xi, yi) 点处。输入数据保证第 1 和第 2 个路由器在仅有这 n 个路由器的情况下已经可以互相连接(经过一系列的中转路由器)。
　　接下来 m 行,每行包含两个整数 xi 和 yi,表示 (xi, yi) 点处可以增设 一个路由器。
　　输入中所有的坐标的绝对值不超过 108,保证输入中的坐标各不相同。

输出格式

　　输出只有一个数,即在指定的位置中增设 k 个路由器后,从第 1 个路 由器到第 2 个路由器最少经过的中转路由器的个数。

样例输入

5 3 1 3
0 0
5 5
0 3
0 5
3 5
3 3
4 4
3 0

样例输出

2

**解法**

这个题虽然是最短路径问题，但是不妨换个角度，把这里的路径长度看作是跳数，那么这个最短路径问题就可以用 BFS 解决。为了顺利使用 BFS，还需要做一些转换，不要去思考该怎么增加路由器，而是一开始就假设 m 个路由器已经设置好了，只不过最短路径最多只能经过它们中的 k 个。

```java
import java.math.BigDecimal;
import java.util.*;

public class Main {
    private static List<Route> routes = new LinkedList<>();

    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        int n = in.nextInt();
        int m = in.nextInt();
        int k = in.nextInt();
        BigDecimal r = in.nextBigDecimal();
        routes.add(null);
        for (int i = 0; i < n + m; i++) {
            routes.add(new Route(in.nextBigDecimal(), in.nextBigDecimal()));
        }
        System.out.println(bfs(n ,m, k, r));
    }

    private static int bfs(int n, int m, int k, BigDecimal r) {
        boolean[] visited = new boolean[n + m + 1];
        Route src = routes.get(1);
        Queue<Route> q = new LinkedList<>();
        q.offer(src);
        visited[1] = true;

        Route head;
        int limit;
        while (!q.isEmpty()) {
            head = q.poll();
            // 已经搜索到了第2个路由，算法结束
            if (head.equals(routes.get(2))) {
                return head.hop - 1;
            }
            // 如果新增路由的使用个数已经到了最大值，
            // 那么在下面的搜索中就只考虑前 n 个路由
            if (head.add == k) {
                limit = n;
            } else {
                limit = n + m;
            }
            for (int i = 1; i <= limit; i++) {
                Route cur = routes.get(i);
                // 判断两个路由之间的距离是否超过阈值
                if (!visited[i] && distance(head, cur).compareTo(r.pow(2)) <= 0) {
                    visited[i] = true;
                    Route route = new Route(cur.x, cur.y);
                    route.hop = head.hop + 1;
                    if (i >= n) {
                        route.add = head.add + 1;
                    } else {
                        route.add = head.add;
                    }
                    q.offer(route);
                }
            }
        }
        return 0;
    }

    private static BigDecimal distance(Route head, Route route) {
        BigDecimal dx = head.x.subtract(route.x);
        BigDecimal dy = head.y.subtract(route.y);
        return dx.pow(2).add(dy.pow(2));
    }
}

class Route {
    BigDecimal x;

    BigDecimal y;

    // hop 表示从第1个路由到当前路由的跳数
    int hop;

    // add 表示搜索到当前路由时已经使用了新增路由中的多少个
    int add;

    public Route(BigDecimal x, BigDecimal y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Route route = (Route) o;
        return x.equals(route.x) &&
                y.equals(route.y);
    }
}
```

100分。

### 20160404

问题描述

　　小明在玩一个电脑游戏，游戏在一个*n*×*m*的方格图上进行，小明控制的角色开始的时候站在第一行第一列，目标是前往第*n*行第*m*列。
 　　方格图上有一些方格是始终安全的，有一些在一段时间是危险的，如果小明控制的角色到达一个方格的时候方格是危险的，则小明输掉了游戏，如果小明的角色到达了第*n*行第*m*列，则小明过关。第一行第一列和第*n*行第*m*列永远都是安全的。
 　　每个单位时间，小明的角色必须向上下左右四个方向相邻的方格中的一个移动一格。
 　　经过很多次尝试，小明掌握了方格图的安全和危险的规律：每一个方格出现危险的时间一定是连续的。并且，小明还掌握了每个方格在哪段时间是危险的。
 　　现在，小明想知道，自己最快经过几个时间单位可以达到第*n*行第*m*列过关。

输入格式

　　输入的第一行包含三个整数*n*, *m*, *t*，用一个空格分隔，表示方格图的行数*n*、列数*m*，以及方格图中有危险的方格数量。
 　　接下来*t*行，每行4个整数*r*, *c*, *a*, *b*，表示第*r*行第*c*列的方格在第*a*个时刻到第*b*个时刻之间是危险的，包括*a*和*b*。游戏开始时的时刻为0。输入数据保证*r*和*c*不同时为1，而且当*r*为*n*时*c*不为*m*。一个方格只有一段时间是危险的（或者说不会出现两行拥有相同的*r*和*c*）。

输出格式

　　输出一个整数，表示小明最快经过几个时间单位可以过关。输入数据保证小明一定可以过关。

样例输入

3 3 3
 2 1 1 1
 1 3 2 10
 2 2 2 10

样例输出

6

样例说明

　　第2行第1列时刻1是危险的，因此第一步必须走到第1行第2列。
 　　第二步可以走到第1行第1列，第三步走到第2行第1列，后面经过第3行第1列、第3行第2列到达第3行第3列。

评测用例规模与约定

　　前30%的评测用例满足：0 < *n*, *m* ≤ 10，0 ≤ *t* < 99。
 　　所有评测用例满足：0 < *n*, *m* ≤ 100，0 ≤ *t* < 9999，1 ≤ *r* ≤ *n*，1 ≤ *c* ≤ *m*，0 ≤ *a* ≤ *b* ≤ 100。

**解法**

规则格网条件下的最短路径（此处依然是求最短路径，因为每走一步的耗时都是1）可以用 BFS 求解。但是这里多出了一个时间上的限制，也就是第三维的限制。为了将问题转换为常规的 BFS，不妨将`visited`数组设置为三维的，第三维表示时间，`visited[i][j][t]`就表示`(i, j)`这个位置在时间 t 是否被访问（也就是题目中所说的处于危险时间段）。

```java
import java.util.LinkedList;
import java.util.Queue;
import java.util.Scanner;

public class Main {

    private static boolean[][][] visited;

    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        int n = in.nextInt();
        int m = in.nextInt();
        int t = in.nextInt();
        // 注意这里的时间维度的最大值，总之要设置为100的三倍以上，因为一个位置可能经过多次
        visited = new boolean[n + 1][m + 1][310];
        for (int i = 0; i < t; i++) {
            int r = in.nextInt();
            int c = in.nextInt();
            int a = in.nextInt();
            int b = in.nextInt();
            for (int j = a; j <= b; j++) {
                visited[r][c][j] = true;
            }
        }
        System.out.println(bfs(n, m));
    }

    private static int bfs(int n, int m) {
        Cell start = new Cell(1, 1, 0);
        Queue<Cell> q = new LinkedList<>();
        q.offer(start);
        int[][] dir = new int[][]{{1, 0}, {-1, 0}, {0, 1}, {0, -1}};

        while (!q.isEmpty()) {
            Cell cur = q.poll();
            // 已经到达终点
            if (cur.r == n && cur.c == m) {
                return cur.time;
            }
            // 搜索下一个可前往的位置
            for (int i = 0; i < 4; i++) {
                int nr = cur.r + dir[i][0];
                int nc = cur.c + dir[i][1];
                if (nr < 1 || nr > n || nc < 1 || nc > m) {
                    continue;
                }
                // 现在这个位置处于危险时间段
                if (visited[nr][nc][cur.time + 1]) {
                    continue;
                }
                q.offer(new Cell(nr, nc, cur.time + 1));
                visited[nr][nc][cur.time + 1] = true;
            }
        }
        return 0;
    }
}

class Cell {
    int r;

    int c;

    // 走到当前位置时的时间
    int time;

    public Cell(int r, int c, int time) {
        this.r = r;
        this.c = c;
        this.time = time;
    }
}
```

100分。

# 拓扑排序

只有有向无环图才能进行拓扑排序，有向无环图至少有一个拓扑排序的序列。

下面给出 Kahn 算法的模板。

```java
public class TopologicalSort {
    // 顶点数量
    // 顶点编号 1 ~ N
    private int N;

    // 邻接表，保存顶点和与其相邻的下一个顶点的对应关系
    private Map<Integer, List<Integer>> adj = new HashMap<>();

    public List<Integer> sort() {
        // 获取邻接信息

        // 计算所有顶点的入度
        int[] indegree = new int[N + 1];
        for (int i = 1; i <= N; i++) {
            if (!adj.containsKey(i)) {
                continue;
            }
            for (int v : adj.get(i)) {
                indegree[v]++;
            }
        }

        // 找出所有入度为0的顶点
        // q 在整个排序过程中保存入度为0且还没排序的顶点
        Queue<Integer> q = new LinkedList<>();
        for (int i = 1; i <= N; i++) {
            if (indegree[i] == 0) {
                q.offer(i);
            }
        }

        // 开始排序
        int count = 0;
        List<Integer> res = new LinkedList<>();
        while (!q.isEmpty()) {
            int v = q.poll();
            res.add(v);
            count++;
            if (!adj.containsKey(v)) {
                continue;
            }
            for (int u : adj.get(v)) {
                indegree[u]--;
                if (indegree[u] == 0) {
                    q.offer(u);
                }
            }
        }
        if (count != N) {
            // 存在环
            return null;
        } else {
            return res;
        }
    }
}
```

## 习题

### 确定比赛名次

题目描述

有N个比赛队（1<=N<=500），编号依次为1，2，3，。。。。，N进行比赛，比赛结束后，裁判委员会要将所有参赛队伍从前往后依次排名，但现在裁判委员会不能直接获得每个队的比赛成绩，只知道每场比赛的结果，即P1赢P2，用P1，P2表示，排名时P1在P2之前。现在请你编程序确定排名。

输入

输入有若干组，每组中的第一行为二个数N（1<=N<=500），M；其中N表示队伍的个数，M表示接着有M行的输入数据。接下来的M行数据中，每行也有两个整数P1，P2表示即P1队赢了P2队。

输出

给出一个符合要求的排名。输出时队伍号之间有空格，最后一名后面没有空格。

其他说明：符合条件的排名可能不是唯一的，此时要求输出时编号小的队伍在前；输入数据保证是正确的，即输入数据确保一定能有一个符合要求的排名。

样例输入

```
3 2
3 1
3 2
17 16
16 1
13 2
7 3
12 4
12 5
17 6
10 7
11 8
11 9
16 10
13 11
15 12
15 13
17 14
17 15
17 16
0 0
```

样例输出

```
3 1 2
17 6 14 15 12 4 5 13 2 11 8 9 16 1 10 7 3
```

**解法**

观察第二组样例的输入输出就会发现，题目实际上是当某个顶点入度为0之后就从它开始进行 DFS。同时为了满足编号小的队伍在前的限制，要先搜索编号小的邻接顶点。

```java
import java.util.*;

public class Main {
    private static LinkedList<String> ans = new LinkedList<>();

    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        for (int n = in.nextInt(); n > 0; n = in.nextInt()) {
            int m = in.nextInt();
            // 使用 SortedSet 对邻接顶点排序
            Map<Integer, SortedSet<Integer>> adj = new HashMap<>();
            for (int i = 0; i < m; i++) {
                int t1 = in.nextInt();
                int t2 = in.nextInt();
                if (!adj.containsKey(t1)) {
                    adj.put(t1, new TreeSet<>());
                }
                adj.get(t1).add(t2);
            }
            if (!adj.isEmpty()) {
                sort(adj, n);
            }
        }
        for (String s : ans) {
            System.out.println(s);
        }
    }

    private static void sort(Map<Integer, SortedSet<Integer>> adj, int n) {
        int[] indegree = new int[n + 1];
        for (SortedSet<Integer> l : adj.values()) {
            for (int u : l) {
                indegree[u]++;
            }
        }
        PriorityQueue<Integer> q = new PriorityQueue<>();
        for (int i = 1; i <= n; i++) {
            if (indegree[i] == 0) {
                q.offer(i);
            }
        }
        StringBuilder builder = new StringBuilder();
        boolean[] visited = new boolean[n + 1];
        while (!q.isEmpty()) {
            int u = q.poll();
            dfs(adj, indegree, u, visited, builder);
        }
        ans.add(builder.substring(0, builder.length() - 1));
    }

    private static void dfs(Map<Integer, SortedSet<Integer>> adj, int[] indegree, int u, boolean[] visited, StringBuilder builder) {
        if (!visited[u]) {
            visited[u] = true;
            builder.append(u).append(" ");
            if (adj.containsKey(u)) {
                for (int v : adj.get(u)) {
                    indegree[v]--;
                    if (indegree[v] == 0) {
                        dfs(adj, indegree, v, visited, builder);
                    }
                }
            }
        }
    }
}
```

正确。

# 环的检测

## 有向图

在拓扑排序的 Kahn 算法中就已经包含了环的检测，但这个方法需要将整个图进行一次拓扑排序才能知道是否存在环，比较费事。

下面给出的是基于 DFS 的算法。如果在 DFS 过程中遇到了一个已经在递归栈中的顶点，就说明有环。

```java
public class DetectCycle {
    // adj 是邻接信息，顶点编号 1~N
    public boolean detect(Map<Integer, List<Integer>> adj, int N) {
        boolean[] visited = new boolean[N + 1];
        boolean[] onStack = new boolean[N + 1];
        for (int i = 1; i <= N; i++) {
            if (dfs(adj, i, visited, onStack)) {
                return true;
            }
        }
        return false;
    }

    private boolean dfs(Map<Integer, List<Integer>> adj, int i, boolean[] visited, boolean[] onStack) {
        if (onStack[i]) {
            // 这个顶点已经在递归栈上，存在环
            return true;
        }
        if (visited[i]) {
            return false;
        } else {
            visited[i] = true;
            onStack[i] = true;
            if (adj.containsKey(i)) {
                for (int j : adj.get(i)) {
                    if (dfs(adj, j, visited, onStack)) {
                        return true;
                    }
                }
            }
            // 将顶点从递归栈移除
            onStack[i] = false;
            return false;
        }
    }
}
```

## 无向图

对于无向图，使用 union find 来检测环会比较简便。对于每一条边，如果发现两个顶点在同一个 set 中，那么就说明存在环。如果只是要判断环，那么存储边的时候就不用区分终点起点，也就是说不用把一条无向边转成两条有向边。

```java
public class DetectCycle {
    // adj 是邻接信息，顶点编号 1~N
    public boolean detect(Map<Integer, List<Integer>> adj, int N) {
        UnionFind u = new UnionFind(N);
        for (Map.Entry<Integer, List<Integer>> p : adj.entrySet()) {
            for (int v : p.getValue()) {
                int r1 = u.find(p.getKey());
                int r2 = u.find(v);
                if (r1 == r2) {
                    // 一条边的两个顶点在同一个 union 中，存在环
                    return true;
                } else {
                    u.union(r1, r2);
                }
            }
        }
        return false;
    }
}

class UnionFind {
    private int[] roots;
    
    private int[] rank;
    
    // 元素编号1~n
    public UnionFind(int n) {
        roots = new int[n + 1];
        for (int i = 1; i <= n; i++) {
            roots[i] = i;
        }
        rank = new int[n + 1];
    }
    
    public int find(int x) {
        if (x != roots[x]) {
            roots[x] = find(roots[x]);
        }
        return roots[x];
    }
    
    public void union(int x, int y) {
        int rx = find(x);
        int ry = find(y);
        if (rx != ry) {
            if (rank[x] < rank[y]) {
                int tmp = rx;
                rx = ry;
                ry = tmp;
            }
        }
        roots[ry] = rx;
        if (rank[rx] == rank[ry]) {
            rank[rx]++;
        }
    }
}
```

# 连通分量

在有向图 G 中，如果两个顶点间至少存在一条路径，称两个顶点**强连通**(strongly connected)。如果有向图 G 的每两个顶点都强连通，称 G 是一个**强连通图**。非强连通图有向图的极大强连通子图，称为**强连通分量**(strongly connected components)。

最简单的做法是用 union find。在多种求有向图的连通分量的算法中，效率比较好的是 Tarjan 算法。该算法基于 DFS。基本思想是从任意顶点开始进行 DFS（后续的 DFS 就从还没被访问过的顶点开始），顶点按照被访问的顺序入栈，当访问到之前已经访问过的一个顶点时，检查这个顶点是不是某个连通分量的根节点并将其出栈。如果某个顶点是根节点，那么在它之前出栈的且还不属于其他连通分量的顶点就与这个顶点一起构成一个连通分量。

算法的关键在于判断某个顶点是不是连通分量的根。注意，连通分量实际上并没有“根”，这个算法使用根这个说法来描述 DFS 时该连通分量中首先被访问到的顶点。为了找到根，给每个顶点一个搜索标号 index，标识它是第几个被访问到的；再给顶点一个值 lowlink，标识从它出发可到达的所有顶点中最小的 index。显然对同一顶点而言 index 大于等于 lowlink，且当从该顶点出发不能到达其他顶点时这两个值相等。当前仅当 index == lowlink 时该顶点是连通分量的根。

下面给出算法的模板，图采用邻接表的方式表达。

```java
import java.math.BigDecimal;
import java.util.*;

public class Main {

    // 邻接表
    private static Map<Integer, List<Integer>> adj = new HashMap<>();

    // 每个顶点的 index
    private static int[] index;

    // 遍历的顺序
    private static int cur = 0;

    // 每个顶点的 lowlink
    private static int[] lowlink;

    private static Stack<Integer> stack = new Stack<>();

    private static BigDecimal ans = BigDecimal.ZERO;

    public static void main(String[] args) {
        // 获取邻接信息

        // 初始化
        index = new int[n + 1];
        Arrays.fill(index, -1);
        lowlink = new int[n + 1];
        // 开始算法
        // 顶点编号 1~n
        for (int i = 1; i <= n; i++) {
            // 当前顶点没有被访问过
            if (index[i] == -1) {
                tarjan(i);
            }
        }
    }

    private static void tarjan(int src) {
        index[src] = cur;
        lowlink[src] = cur;
        cur++;
        stack.push(src);
        if (adj.containsKey(src)) {
            for (int to : adj.get(src)) {
                // DFS 
                if (index[to] == -1) {
                    tarjan(to);
                    // 更新当前顶点的 lowlink
                    lowlink[src] = Math.min(lowlink[src], lowlink[to]);
                } else if (stack.contains(to)) {
                    // 如果当前顶点的某个邻接顶点在堆栈上，那就自然和当前顶点同属一个连通分量
                    // 注意下面这里取的是 lowlink[src] 和 index[to] 的最小值
                    lowlink[src] = Math.min(lowlink[src], index[to]);
                }
            }
        }
        // 找到一个连通分量，当前顶点是根节点
        if (lowlink[src] == index[src]) {
            // 获取该连通分量的所有顶点
            int w;
            do {
                w = stack.pop();
            } while (w != src);
        }
    }
}
```

注意`lowlink[src] = Math.min(lowlink[src], index[to]);`这行代码。当前顶点的某个邻接顶点已经在堆栈上时，就是说明这两个顶点的边在当前 DFS 生成的子树中是一条 back edge。如下图：

```flow
st1=>start: 1
st2=>start: 2
st3=>start: 3
st4=>start: 4

st1->st2->st3->st4->st2
```

从1开始 DFS 时按以下顺序生成子树的边；1->2, 2->3, 3->4，当搜索到 4->2 这条边时，4是2的父节点，但是2却先于4被访问到，这条边就是一条 back edge，因此这条边就不能成为 DFS 生成的子树中的边。

因此，当搜索到了一条 back edge 时，由于这条边不会成为当前 DFS 子树的边，所以搜索到此为止。lowlink 考虑的是当前顶点能访问到的所有顶点的 index 中的最小值，而目前已经达到了搜索的最大深度，所以应该是取当前顶点的 lowlink 和其当前邻接顶点的 index 中的最小值作为 当前顶点新的 lowlink。

## 真题

### 20150904

问题描述

　　某国有*n*个城市，为了使得城市间的交通更便利，该国国王打算在城市之间修一些高速公路，由于经费限制，国王打算第一阶段先在部分城市之间修一些单向的高速公路。
　　现在，大臣们帮国王拟了一个修高速公路的计划。看了计划后，国王发现，有些城市之间可以通过高速公路直接（不经过其他城市）或间接（经过一个或多个其他城市）到达，而有的却不能。如果城市A可以通过高速公路到达城市B，而且城市B也可以通过高速公路到达城市A，则这两个城市被称为便利城市对。
　　国王想知道，在大臣们给他的计划中，有多少个便利城市对。

输入格式

　　输入的第一行包含两个整数*n*, *m*，分别表示城市和单向高速公路的数量。
　　接下来*m*行，每行两个整数*a*, *b*，表示城市*a*有一条单向的高速公路连向城市*b*。

输出格式

　　输出一行，包含一个整数，表示便利城市对的数量。

样例输入

5 5
1 2
2 3
3 4
4 2
3 5

样例输出

3

样例说明

![img](http://118.190.20.162/RequireFile.do?fid=4HG9GgbF)
　　城市间的连接如图所示。有3个便利城市对，它们分别是(2, 3), (2, 4), (3, 4)，请注意(2, 3)和(3, 2)看成同一个便利城市对。

评测用例规模与约定

　　前30%的评测用例满足1 ≤ *n* ≤ 100, 1 ≤ *m* ≤ 1000；
　　前60%的评测用例满足1 ≤ *n* ≤ 1000, 1 ≤ *m* ≤ 10000；
　　所有评测用例满足1 ≤ *n* ≤ 10000, 1 ≤ *m* ≤ 100000。

显然这个图的做好做法就是找连通分量，如果一个连通分量由 n 个顶点构成，那么这个连通分量包含的便利城市对数量就是$$C^2_n$$。

```java
import java.math.BigDecimal;
import java.util.*;

public class Main {

    private static Map<Integer, List<Integer>> adj = new HashMap<>();

    private static int[] index;

    private static int cur = 0;

    private static int[] lowlink;

    private static Stack<Integer> stack = new Stack<>();

    private static BigDecimal ans = BigDecimal.ZERO;

    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        int n = in.nextInt();
        int m = in.nextInt();
        for (int i = 0; i < m; i++) {
            int a = in.nextInt();
            int b = in.nextInt();
            if (!adj.containsKey(a)) {
                adj.put(a, new LinkedList<>());
            }
            adj.get(a).add(b);
        }

        index = new int[n + 1];
        Arrays.fill(index, -1);
        lowlink = new int[n + 1];
        for (int i = 1; i <= n; i++) {
            if (index[i] == -1) {
                tarjan(i);
            }
        }
        System.out.println(ans);
    }

    private static void tarjan(int src) {
        index[src] = cur;
        lowlink[src] = cur;
        cur++;
        stack.push(src);
        if (adj.containsKey(src)) {
            for (int to : adj.get(src)) {
                if (index[to] == -1) {
                    tarjan(to);
                    lowlink[src] = Math.min(lowlink[src], lowlink[to]);
                } else if (stack.contains(to)) {
                    lowlink[src] = Math.min(lowlink[src], index[to]);
                }
            }
        }
        if (lowlink[src] == index[src]) {
            BigDecimal count = BigDecimal.ZERO;
            int w;
            do {
                w = stack.pop();
                count = count.add(BigDecimal.ONE);
            } while (w != src);
            // 计算组合数
            ans = ans.add(count.multiply(count.subtract(BigDecimal.ONE)).divide(BigDecimal.valueOf(2)));
        }
    }
}
```

运行错误，80分。

# 最小生成树

从代码的角度来看，Kruskal 算法更容易实现，所以采用该算法解题，模板为：

```java
public class Main {

    // 判断下大概会有多少个顶点，N 要稍大于顶点数的最大值
    private static final int N = (int) (1e5 + 10);  

    private static List<Edge> edges = new ArrayList<>(N);

    private static int[] uf = new int[N];

    public static void main(String[] args) {
        // 获取边的数据
        int m; // 边数 
        int n; // 结点数
        
        // 按边的权重升序排序
        Collections.sort(edges, new Comparator<Edge>() {
            @Override
            public int compare(Edge o1, Edge o2) {
                return o1.price - o2.price;
            }
        });
        
        // 初始化  union
        Arrays.fill(uf, -1);
        
        int count = 0;
        for (int i = 0; i < m; i++) {
            // find
            int r1 = find(edges.get(i).from);
            int r2 = find(edges.get(i).to);
            // 这条边的两个结点不在同一个 union 里面，不构成环，选择这条边
            // union
            if (r1 != r2) {
                uf[r1] = r2;
                count++;
                // n 个节点的生成树有 n - 1 条边
                if (count == n - 1) {
                    break;
                }
            }
        }
    }

    // 借助 union find 判断是否构成环
    private static int find(int x) {
        if (uf[x] == -1) {
            return x;
        } else {
            uf[x] = find(uf[x]);
            return uf[x];
        }
    }
}

class Edge {
    int from;
    int to;
    int price;

    public Edge(int from, int to, int time) {
        this.from = from;
        this.to = to;
        this.price = time;
    }
}
```

## 习题

### 继续畅通工程

题目描述

省政府“畅通工程”的目标是使全省任何两个村庄间都可以实现公路交通（但不一定有直接的公路相连，只要能间接通过公路可达即可）。现得到城镇道路统计表，表中列出了任意两城镇间修建道路的费用，以及该道路是否已经修通的状态。现请你编写程序，计算出全省畅通需要的最低成本。

输入

测试输入包含若干测试用例。每个测试用例的第1行给出村庄数目N ( 1< N < 100 )；随后的 N(N-1)/2 行对应村庄间道路的成本及修建状态，每行给4个正整数，分别是两个村庄的编号（从1编号到N），此两村庄间道路的成本，以及修建状态：1表示已建，0表示未建。

当N为0时输入结束。

输出

每个测试用例的输出占一行，输出全省畅通需要的最低成本。

样例输入

```
4
1 2 1 1
1 3 6 0
1 4 2 1
2 3 3 0
2 4 5 0
3 4 4 0
3
1 2 1 1
2 3 2 1
1 3 1 0
0
```

样例输出

```
3
0
```

```java
import java.util.*;

public class Main {
    private static PriorityQueue<Edge> edges = new PriorityQueue<>(new Comparator<Edge>() {
        @Override
        public int compare(Edge o1, Edge o2) {
            // 如果两条边都没有建或者都已经建了，按照权重排序
            if ((o1.existing == 1 && o2.existing == 1)
                    || (o1.existing == 0 && o2.existing == 0)) {
                return o1.weight - o2.weight;
            } else {
                // 如果有一条边没有建立，则已经建好的排在前面
                return o2.existing - o1.existing;
            }
        }
    });

    private static int[] roots;

    private static int[] rank;

    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        while (true) {
            int N = in.nextInt();
            if (N == 0) {
                return;
            }
            for (int i = 0; i < N * (N - 1) / 2; i++) {
                int v1 = in.nextInt();
                int v2 = in.nextInt();
                int w = in.nextInt();
                int status = in.nextInt();
                addEdge(v1, v2, w, status);
                addEdge(v2, v1, w, status);
            }
            roots = new int[N + 1];
            for (int i = 1; i <= N; i++) {
                roots[i] = i;
            }
            rank = new int[N + 1];
            int count = 0;
            int total = 0;
            while (!edges.isEmpty()) {
                Edge e = edges.poll();
                int rf = find(e.from);
                int rt = find(e.to);
                if (rf != rt) {
                    union(rf, rt);
                    if (e.existing == 0) {
                        total += e.weight;
                    }
                    count++;
                    if (count == N - 1) {
                        System.out.println(total);
                        break;
                    }
                }
            }
            edges.clear();
        }
    }

    private static void addEdge(int from, int to, int weight, int existing) {
        edges.add(new Edge(from, to, weight, existing));
    }

    private static int find(int x) {
        if (x != roots[x]) {
            roots[x] = find(roots[x]);
        }
        return roots[x];
    }

    private static void union(int x, int y) {
        int rx = find(x);
        int ry = find(y);
        if (rx != ry) {
            if (rank[rx] < rank[ry]) {
                int tmp = rx;
                rx = ry;
                ry = tmp;
            }
            roots[ry] = rx;
            if (rank[ry] == rank[rx]) {
                rank[rx]++;
            }
        }
    }

}

class Edge {
    int from;

    int to;

    int weight;

    int existing;

    public Edge(int from, int to, int weight, int existing) {
        this.from = from;
        this.to = to;
        this.weight = weight;
        this.existing = existing;
    }
}
```

正确。

## 真题

### [数据中心](http://118.190.20.162/view.page?gpid=T83)

![img](http://118.190.20.162/RequireFile.do?fid=DagRh9nq)

样例输入

4
5
1
1 2 3
1 3 4
1 4 5
2 3 8
3 4 2

样例输出

4

样例说明

　　下图是样例说明。
![img](http://118.190.20.162/RequireFile.do?fid=2LYA5Jrq)

题目描述得很复杂，但是观察样例后发现实际上就是求最小生成树：生成最小生成树时优先选择权重小的边，这样自然能保证题目中的总耗时最小，而总耗时就是最小生成树中的最大权重。

```java
import java.util.*;

public class Main {

    private static final int N = (int) (1e5 + 10);

    private static List<Edge> edges = new ArrayList<>(N);

    private static int[] uf = new int[N];

    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        int n = in.nextInt();
        int m = in.nextInt();
        in.nextInt();
        for (int i = 0; i < m; i++) {
            edges.add(new Edge(in.nextInt(), in.nextInt(), in.nextInt()));
        }
        Collections.sort(edges, new Comparator<Edge>() {
            @Override
            public int compare(Edge o1, Edge o2) {
                return o1.time - o2.time;
            }
        });
        Arrays.fill(uf, -1);
        int count = 0;
        int ans = 0;
        for (int i = 0; i < m; i++) {
            int r1 = find(edges.get(i).from);
            int r2 = find(edges.get(i).to);
            if (r1 != r2) {
                uf[r1] = r2;
                ans = Math.max(ans, edges.get(i).time);
                count++;
                if (count == n - 1) {
                    break;
                }
            }
        }
        System.out.println(ans);
    }

    private static int find(int x) {
        if (uf[x] == -1) {
            return x;
        } else {
            uf[x] = find(uf[x]);
            return uf[x];
        }
    }
}

class Edge {
    int from;
    int to;
    int time;

    public Edge(int from, int to, int time) {
        this.from = from;
        this.to = to;
        this.time = time;
    }
}
```

超时， 90 分。

### 地铁修建

问题描述

　　A市有n个交通枢纽，其中1号和n号非常重要，为了加强运输能力，A市决定在1号到n号枢纽间修建一条地铁。
　　地铁由很多段隧道组成，每段隧道连接两个交通枢纽。经过勘探，有m段隧道作为候选，两个交通枢纽之间最多只有一条候选的隧道，没有隧道两端连接着同一个交通枢纽。
　　现在有n家隧道施工的公司，每段候选的隧道只能由一个公司施工，每家公司施工需要的天数一致。而每家公司最多只能修建一条候选隧道。所有公司同时开始施工。
　　作为项目负责人，你获得了候选隧道的信息，现在你可以按自己的想法选择一部分隧道进行施工，请问修建整条地铁最少需要多少天。

输入格式

　　输入的第一行包含两个整数*n*, *m*，用一个空格分隔，分别表示交通枢纽的数量和候选隧道的数量。
　　第2行到第*m*+1行，每行包含三个整数*a*, *b*, *c*，表示枢纽*a*和枢纽*b*之间可以修建一条隧道，需要的时间为*c*天。

输出格式

　　输出一个整数，修建整条地铁线路最少需要的天数。

样例输入

6 6
1 2 4
2 3 4
3 6 7
1 4 2
4 5 5
5 6 6

样例输出

6

样例说明

　　可以修建的线路有两种。
　　第一种经过的枢纽依次为1, 2, 3, 6，所需要的时间分别是4, 4, 7，则整条地铁线需要7天修完；
　　第二种经过的枢纽依次为1, 4, 5, 6，所需要的时间分别是2, 5, 6，则整条地铁线需要6天修完。
　　第二种方案所用的天数更少。

评测用例规模与约定

　　对于20%的评测用例，1 ≤ *n* ≤ 10，1 ≤ *m* ≤ 20；
　　对于40%的评测用例，1 ≤ *n* ≤ 100，1 ≤ *m* ≤ 1000；
　　对于60%的评测用例，1 ≤ *n* ≤ 1000，1 ≤ *m* ≤ 10000，1 ≤ *c* ≤ 1000；
　　对于80%的评测用例，1 ≤ *n* ≤ 10000，1 ≤ *m* ≤ 100000；
　　对于100%的评测用例，1 ≤ *n* ≤ 100000，1 ≤ *m* ≤ 200000，1 ≤ *a*, *b* ≤ *n*，1 ≤ *c* ≤ 1000000。

　　所有评测用例保证在所有候选隧道都修通时1号枢纽可以通过隧道到达其他所有枢纽。

**解法**

最直接的做法就是求最小生成树，当顶点1和顶点 n 同处一个连通分量时就可得到权重最长的边，即答案。

注意这题要求的是**最大的权重尽可能小**，如果问的是**权重总和最小**，自然还是要求最短路径。

```java
import java.util.*;

public class Main {
    private static int[] roots;

    private static int[] rank;

    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        int n = in.nextInt();
        int m = in.nextInt();
        List<Edge> edges = new LinkedList<>();
        for (int i = 0; i < m; i++) {
            int a = in.nextInt();
            int b = in.nextInt();
            int c = in.nextInt();
            edges.add(new Edge(a, b, c));
        }
        Collections.sort(edges, new Comparator<Edge>() {
            @Override
            public int compare(Edge o1, Edge o2) {
                return o1.weight - o2.weight;
            }
        });
        roots = new int[n + 1];
        for (int i = 1; i <= n; i++) {
            roots[i] = i;
        }
        rank = new int[n + 1];
        for (Edge e : edges) {
            int rx = find(e.from);
            int ry = find(e.to);
            if (rx != ry) {
                union(rx, ry);
                if (find(1) == find(n)) {
                    System.out.println(e.weight);
                    break;
                }
            }
        }
    }

    private static int find(int x) {
        if (x != roots[x]) {
            roots[x] = find(roots[x]);
        }
        return roots[x];
    }

    private static void union(int x, int y) {
        int rx = find(x);
        int ry = find(y);
        if (rx != ry) {
            if (rx < ry) {
                int tmp = rx;
                rx = ry;
                ry = tmp;
            }
        }
        roots[ry] = rx;
        if (rank[rx] == rank[ry]) {
            rank[rx]++;
        }
    }

}


class Edge {
    int from;

    int to;

    int weight;

    public Edge(int from, int to, int weight) {
        this.from = from;
        this.to = to;
        this.weight = weight;
    }
}
```

超时，75分。

### 20141204

问题描述

　　雷雷承包了很多片麦田，为了灌溉这些麦田，雷雷在第一个麦田挖了一口很深的水井，所有的麦田都从这口井来引水灌溉。
　　为了灌溉，雷雷需要建立一些水渠，以连接水井和麦田，雷雷也可以利用部分麦田作为“中转站”，利用水渠连接不同的麦田，这样只要一片麦田能被灌溉，则与其连接的麦田也能被灌溉。
　　现在雷雷知道哪些麦田之间可以建设水渠和建设每个水渠所需要的费用（注意不是所有麦田之间都可以建立水渠）。请问灌溉所有麦田最少需要多少费用来修建水渠。

输入格式

　　输入的第一行包含两个正整数n, m，分别表示麦田的片数和雷雷可以建立的水渠的数量。麦田使用1, 2, 3, ……依次标号。
　　接下来m行，每行包含三个整数ai, bi, ci，表示第ai片麦田与第bi片麦田之间可以建立一条水渠，所需要的费用为ci。

输出格式

　　输出一行，包含一个整数，表示灌溉所有麦田所需要的最小费用。

样例输入

4 4
1 2 1
2 3 4
2 4 2
3 4 3

样例输出

6

样例说明

　　建立以下三条水渠：麦田1与麦田2、麦田2与麦田4、麦田4与麦田3。

评测用例规模与约定

　　前20%的评测用例满足：n≤5。
　　前40%的评测用例满足：n≤20。
　　前60%的评测用例满足：n≤100。
　　所有评测用例都满足：1≤n≤1000，1≤m≤100,000，1≤ci≤10,000。

**解法**

很明显这就是一个最小生成树问题。

```java
import java.math.BigDecimal;
import java.util.*;

public class Main {

    private static List<Edge> edges = new LinkedList<>();

    private static BigDecimal ans = BigDecimal.ZERO;

    private static int[] roots;

    private static int[] rank;

    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        int n = in.nextInt();
        int m = in.nextInt();
        for (int i = 0; i < m; i++) {
            edges.add(new Edge(in.nextInt(), in.nextInt(), in.nextInt()));
        }
        Collections.sort(edges, new Comparator<Edge>() {
            @Override
            public int compare(Edge o1, Edge o2) {
                return o1.weight - o2.weight;
            }
        });

        roots = new int[n + 1];
        for (int i = 1; i <= n; i++) {
            roots[i] = i;
        }
        rank = new int[n + 1];
        int count = 0;
        for (Edge e : edges) {
            int rx = find(e.u);
            int ry = find(e.v);
            if (rx != ry) {
                ans = ans.add(BigDecimal.valueOf(e.weight));
                union(rx, ry);
                count++;
                if (count == n - 1) {
                    System.out.println(ans);
                    return;
                }
            }
        }
    }

    private static void union(int x, int y) {
        int rx = find(x);
        int ry = find(y);
        if (rx != ry) {
            if (rank[rx] < rank[ry]) {
                int tmp = rx;
                rx = ry;
                ry = tmp;
            }
            roots[ry] = rx;
            if (rank[rx] == rank[ry]) {
                rank[rx]++;
            }
        }
    }

    private static int find(int x) {
        if (x != roots[x]) {
            roots[x] = find(roots[x]);
        }
        return roots[x];
    }
}


class Edge {
    int u;

    int v;

    int weight;

    public Edge(int u, int v, int weight) {
        this.u = u;
        this.v = v;
        this.weight = weight;
    }
}
```

正确，100分。

# 遍历树

## 前、中、后、层序

```java
class Node {
    int val;
    
    Node left;
    
    Node right;
    
    // ...
}

public class TravseTree {
    public void preorder(Node root) {
        if (root == null) {
            return;
        }
        // 按需操作
        // ...
        preorder(root.left);
        preorder(root.right);
    }
    
    public void inorder(Node root) {
        if (root == null) {
            return;
        }
        preorder(root.left);
        // 按需操作
        // ...
        preorder(root.right);
    }
    
	public void inorder(Node root) {
        if (root == null) {
            return;
        }
        preorder(root.left);
        preorder(root.right);
        // 按需操作
        // ...
    }
    
    public void levelorder(Node root) {
        if (root == null) {
            return;
        }
        Queue<Node> q = new LinkedList<>();
        q.offer(root);
        while (!q.isEmpty()) {
            Node n = q.poll();
            // 按需操作...
            if (n.left != null) {
                q.offer(n.left);
            }
            if (n.right != null) {
                q.offer(n.right);
            }
        }
    }
}
```

# 哈夫曼树

```java
class HuffmanCoding {
    private Map<Character, String> codes = new HashMap<>();

    // 哈夫曼树的树根
    private Node root = null;

    // 构造哈夫曼树
    public Node encode(Map<Character, Integer> charToFreq) {
        PriorityQueue<Node> q = new PriorityQueue<>(new Comparator<Node>() {
            @Override
            public int compare(Node o1, Node o2) {
                return o1.freq - o2.freq;
            }
        });
        for (Map.Entry<Character, Integer> e : charToFreq.entrySet()) {
            q.offer(new Node(e.getValue(), e.getKey(), null, null));
        }
        while (q.size() > 1) {
            Node min1 = q.poll();
            Node min2 = q.poll();
            // 中间结点的字符必须是不会出现在字符集里的
            Node sum = new Node(min1.freq + min2.freq, '*', min1, min2);
            q.offer(sum);
        }
        root = q.poll();
    }

    // 获取字符的编码
    public String getCode(char c) {
        if (!codes.containsKey(c)) {
            preorder(root, c, "");
        }
        return codes.get(c);
    }

    private void preorder(Node r, char c, String code) {
        if (r.left == null && r.right == null) {
            if (r.c == c) {
                codes.put(c, code);
            }
            return;
        }
        preorder(r.left, c, code + "0");
        preorder(r.right, c, code + "1");
    }
}
```

# Union Find

为了提高速度，这里采用 union by rank。

```java
public class UnionFind {
    private int[] roots;
    
    private int[] rank;
    
    // 元素编号1~n
    public void init(int n) {
        roots = new int[n + 1];
        for (int i = 1; i <= n; i++) {
            roots[i] = i;
        }
        rank = new int[n + 1];
    }
    
    public int find(int x) {
        if (x != roots[x]) {
            roots[x] = find(roots[x]);
        }
        return roots[x];
    }
    
    public void union(int x, int y) {
        int rx = find(x);
        int ry = find(y);
        if (rx != ry) {
            if (rank[rx] < rank[ry]) {
                int tmp = rx;
                rx = ry;
                ry = tmp;
            }
            roots[ry] = rx;
        	if (rank[rx] == rank[ry]) {
            	rank[rx]++;
        	}
        }
    }
}
```



# 动态规划

动态规划算法的核心就是记住已经解决过的子问题的解来节约时间。

```
A * "1+1+1+1+1+1+1+1 =？" *
A : "上面等式的值是多少"
B : *计算* "8!"
A : *在上面等式的左边写上 "1+" *
A : "此时等式的值为多少"
B : *quickly* "9!"
A : "你怎么这么快就知道答案了"
B : "只要在8的基础上加1就行了"
A : "所以你不用重新计算因为你记住了第一个等式的值为8!动态规划算法也可以说是 '记住求过的解来节省时间'"
```

适于使用动态规划算法的问题具有两个特征：（1）原始问题能够划分为若干个子问题，每个子问题的最优解都能求出来（2）在求解的过程中会反复求解某些子问题，将这些子问题的解记录下来以节约时间。

## 最长路径

对于边的权重为正且不存在环的有向图，只需把权重变为负就可以利用 Bellman-Ford 或者 SPFA 算法求解最长路径。但是利用动态规划的思想可以更为简便地解决有**有向无环图**的最长路径问题。注意，这个方法对无向图是没有用的。

（1）求任意一对顶点之间的最长路径

令数组`dp[i]`表示从顶点 i 出发能够获得的最长路径长度，假设顶点 i 的下一个邻接顶点是 j，那么显然`dp[i] = max(dp[i], dp[j] + weight(i, j))`。注意从一个出度为0的顶点出发是找不到任何最长路径的，所以在`dp`数组中这样的顶点对应的值是0。如果需要获知最长路径经过的顶点序列，那么可以用一个数组表示`next[i]`表示从顶点 i 出发的最长路径上的下一个顶点。

代码中用邻接列表表示图。

```java
int DP(int i) {
    // 已经计算得到了 dp[i]
    if (dp[i] > 0) {
        return dp[i];
    }
    
    // 遍历顶点 i 的所有邻接顶点
    if (adj.containsKey(i)) {
        for (Edge e : adj.get(i)) {
            int tmp = dp(e.to) + e.weight;
            if (tmp > dp[i]) {
                dp[i] = tmp;
                next[i] = e.to;
            }
        }
    }
    return DP[i];
}
```

注意，如果按照一定的顺序遍历顶点 i 的邻接顶点，那么便可得到字典序最大/小的最长路径。

（2）固定终点

在没有固定终点的情形中，`dp[i] = 0`表示的是顶点 i 出度为0；但是在固定终点的情形中，`dp[i] = 0`表示的是顶点 i 就是终点。所以此时不能把`dp[]`的初始值设置为0，而应该为负无穷；并且不能通过`dp[i] > 0`判断是否已经计算过`dp[i]`，因为`dp[i]`的初始值是负数，`dp[i] > 0`只能说明还正在计算。

```java
int DP(int i) {
    // 数组 done 标记一个顶点 dp[i] 是否已经计算
    if (done[i]) {
        return dp[i];
    }
    done[i] = true;
    if (adj.containsKey(i)) {
        for (Edge e : adj.get(i)) {
            int tmp = dp(e.to) + e.weight;
            if (tmp > dp[i]) {
                dp[i] = tmp;
                next[i] = e.to;
            }
        }
    }
    return DP[i];
}
```

其实，这里所说的“最长路径”只是针对 DAG 而言，如果将这种思想推广，那么便可用于一般性的“先升序排序，然后查找比某元素大的元素的个数”问题。

## 习题

### 矩形嵌套

题目描述

有n个矩形，每个矩形可以用a,b来描述，表示长和宽。矩形X(a,b)可以嵌套在矩形Y(c,d)中当且仅当a<c,b<d或者b<c,a<d（相当于旋转X90度）。例如（1,5）可以嵌套在（6,2）内，但不能嵌套在（3,4）中。你的任务是选出尽可能多的矩形排成一行，使得除最后一个外，每一个矩形都可以嵌套在下一个矩形内。

输入

第一行是一个正正数N(0<N<10)，表示测试数据组数，
每组测试数据的第一行是一个正正数n，表示该组测试数据中含有矩形的个数(n<=1000)
随后的n行，每行有两个数a,b(0<a,b<100)，表示矩形的长和宽

输出

每组测试数据都输出一个数，表示最多符合条件的矩形数目，每组输出占一行

样例输入

```
1
10
1 2
2 4
5 8
6 10
7 9
3 1
5 8
12 10
9 7
2 2
```

样例输出

```
5
```

**解法**

```java
import java.util.*;

public class Main {
    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        int N = in.nextInt();
        for (int i = 0; i < N; i++) {
            int n = in.nextInt();
            int[][] rectangles = new int[n][2];
            for (int j = 0; j < n; j++) {
                int a = in.nextInt();
                int b = in.nextInt();
                // 令矩形的宽小于长
                rectangles[j] = new int[]{Math.min(a, b), Math.max(a, b)};
            }
            // 对所有矩形按照宽进行排序
            Arrays.sort(rectangles, new Comparator<int[]>() {
                @Override
                public int compare(int[] o1, int[] o2) {
                    return o1[0] - o2[0];
                }
            });
            // dp[i] 表示有多个矩形能嵌套矩形 i
            int[] dp = new int[n];
            int ans = 0;
            // 现在只需比较矩形的长
            for (int j = rectangles.length - 1; j >= 0; j--) {
                for (int k = j + 1; k < rectangles.length; k++) {
                    if (rectangles[k][0] > rectangles[j][0]
                            && rectangles[k][1] > rectangles[j][1]) {
                        dp[j] = Math.max(dp[j], dp[k] + 1);
                    }
                }
                ans = Math.max(ans, dp[j] + 1);
            }
            System.out.println(ans);
        }
    }
}
```

## 真题

### 20161204

问题描述

　　给定一段文字，已知单词*a*1, *a*2, …, *an*出现的频率分别*t*1, *t*2, …, *tn*。可以用01串给这些单词编码，即将每个单词与一个01串对应，使得任何一个单词的编码（对应的01串）不是另一个单词编码的前缀，这种编码称为前缀码。
 　　使用前缀码编码一段文字是指将这段文字中的每个单词依次对应到其编码。一段文字经过前缀编码后的长度为：
 　　*L*=*a*1的编码长度×*t*1+*a*2的编码长度×*t*2+…+ *an*的编码长度×*tn*。
 　　定义一个前缀编码为字典序编码，指对于1 ≤ *i* < *n*，*ai*的编码（对应的01串）的字典序在*ai*+1编码之前，即*a*1, *a*2, …, *an*的编码是按字典序升序排列的。
 　　例如，文字E A E C D E B C C E C B D B E中， 5个单词A、B、C、D、E出现的频率分别为1, 3, 4, 2, 5，则一种可行的编码方案是A:000, B:001, C:01, D:10, E:11，对应的编码后的01串为1100011011011001010111010011000111，对应的长度*L*为3×1+3×3+2×4+2×2+2×5=34。
 　　在这个例子中，如果使用哈夫曼(Huffman)编码，对应的编码方案是A:000, B:01, C:10, D:001, E:11，虽然最终文字编码后的总长度只有33，但是这个编码不满足字典序编码的性质，比如C的编码的字典序不在D的编码之前。
 　　在这个例子中，有些人可能会想的另一个字典序编码是A:000, B:001, C:010, D:011, E:1，编码后的文字长度为35。
 　　请找出一个字典序编码，使得文字经过编码后的长度*L*最小。在输出时，你只需要输出最小的长度*L*，而不需要输出具体的方案。在上面的例子中，最小的长度*L*为34。

输入格式

　　输入的第一行包含一个整数*n*，表示单词的数量。
 　　第二行包含*n*个整数，用空格分隔，分别表示*a*1, *a*2, …, *an*出现的频率，即*t*1, *t*2, …, *tn*。请注意*a*1, *a*2, …, *an*具体是什么单词并不影响本题的解，所以没有输入*a*1, *a*2, …, *an*。

输出格式

　　输出一个整数，表示文字经过编码后的长度*L*的最小值。

样例输入

5
 1 3 4 2 5

样例输出

34

样例说明

　　这个样例就是问题描述中的例子。如果你得到了35，说明你算得有问题，请自行检查自己的算法而不要怀疑是样例输出写错了。

评测用例规模与约定

　　对于30%的评测用例，1 ≤ *n* ≤ 10，1 ≤ *ti* ≤ 20；
 　　对于60%的评测用例，1 ≤ *n* ≤ 100，1 ≤ *ti* ≤ 100；
 　　对于100%的评测用例，1 ≤ *n* ≤ 1000，1 ≤ *ti* ≤ 10000。

**解法**

要解决这个问题首先要理解动态规划中的一个经典问题：石子问题。假设有 n 堆石子，每堆含有的石子数量不一样。合并两堆石子的代价是这两堆石子的总数。如果要将所有石子合并为一堆，最小的代价是多少。

比如有下面4堆石子，要将它们合并成一堆，那么其中一种可能的合并方案为：

```pseudocode
- [1 2] 3 4 -> 这一步的代价是3
- 3 [3 4]   -> 这一步的代价是7
- [3 7]     -> 这一步的代价是10
总共的代价是20
```

最直接的想法就是每次都合并石子数量最少的两堆，这样总共的代价肯定是最小的，这个思想也就是哈夫曼编码的思想。但如果题目要求每次只能合并相邻的两堆石子，那么就不能按照哈夫曼编码的算法来解。

不妨假设现在第1堆到第 k 堆和第 k+1 堆到第 n 堆的石子都已经分别合并好了，那么自然现在只需将这两堆合并。也就是说，这个问题可以分解为若干个子问题，每个子问题各自求最优解，于是就考虑用动态规划。

设`dp[i][j]`为合并第 i 堆到第 j 堆石子的代价，那么当`dp[i][j] > dp[i][k] + dp[k + 1][j] + sum[i][j], i != j（sum[i][j]表示第 i 堆到第 j 堆总共有多少石子）` 时就找到了一个更优解，这样最终肯定能找到最优解。 

```java
int mergeStones(int[] nums) {
    // dp[i][j]: 合并第 i 堆到第 j 堆的最优解
    int[][] dp = new int[nums.length + 1][nums.length + 1];
    for (int i = 1; i < dp.length; i++) {
        dp[i] = fill(dp[i].length);
    }
    // 石子数量的前缀和
    int[] sum = new int[nums.length + 1];
    for (int i = 0; i < nums.length; i++) {
        sum[i] = sum[i - 1] + nums[i];
        dp[i][i] = 0;
    }
    return helper(1, nums.length, sum, dp);
}

int helper(int i, int j, int[] sums, int[][] dp) {
    if (dp[i][j] == Integer.MAX_VALUE) {
        // 寻找合并第 i 堆到第 j 堆的最优解
        for (int k = i, k < j; k++) {
            dp[i][j] = Math.min(dp[i][j], helper(i, k) + helper(k + 1, j) + sum[j] - sum[i -1]);
        }
    }
    return dp[i][j];
}

int[] fill(int n) {
    int[] arr = new int[n];
    Arrays.fill(arr, Integer.MAX_VALUE);
    return arr;
}
```

实际上，i 和 j 不断变化，这也就成了一个滑动窗口问题。

理解了上面的算法就可以解决压缩编码问题，因为二者的本质是一样的。压缩编码的题目要求字典序不变，也就意味着每次都只能合并相邻的两项，这样就能保证顺序不改变。

```java
import java.util.Arrays;
import java.util.Scanner;

public class Main {
    private static int[] sum;

    private static int[][] dp;

    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        int n = in.nextInt();
        sum = new int[n + 1];
        dp = new int[n + 1][n + 1];
        for (int i = 1; i < n + 1; i++) {
            dp[i] = fill(n + 1);
        }
        int[] freq = new int[n + 1];
        for (int i = 1; i <= n; i++) {
            freq[i] = in.nextInt();
            sum[i] = sum[i - 1] + freq[i];
            dp[i][i] = 0;
        }
        System.out.println(solve(1, n));
    }

    private static int solve(int i, int j) {
        if (dp[i][j] == Integer.MAX_VALUE) {
            for (int k = i; k < j; k++) {
                dp[i][j] = Math.min(dp[i][j], solve(i, k) + solve(k + 1, j) + sum[j] - sum[i - 1]);
            }
        }
        return dp[i][j];
    }

    private static int[] fill(int n) {
        int[] arr = new int[n];
        Arrays.fill(arr, Integer.MAX_VALUE);
        return arr;
    }
}
```

100分。

# 差分约束系统

如果一个系统由$$n$$个变量和$$m$$个约束条件组成，其中每个约束条件形如 $$x_{j}-x_{i}\leq b_{k}(i,j\in [1,n],k\in [1,m])$$则称其为差分约束系统。亦即，差分约束系统是求解关于一组变量的特殊不等式组的方法。 

解决办法是转换成单源最短路径问题。观察$${\displaystyle x_{j}-x_{i}\leq b_{k}}$$，发现其跟最短路径算法中使用的不等式$$d[v]≤d[u]+w[u,v]$$很相似（在最短路径算法中，求$$d[v]$$时如果$$d[v]≥d[u]+w[u,v]$$那么就令$$d[v]=d[u]+w[u,v]$$，也就是说求出最短路径后$$d[v]≤d[u]+w[u,v]$$）。将不等式变形为$$d[v]-d[u]≤w[u,v]$$，因此以$${\displaystyle x_{i}}$$为顶点，对于约束条件$${\displaystyle x_{j}-x_{i}\leq b_{k}}$$建立一条有向边$$i->j$$，权重为$${\displaystyle b_{k}}$$。增加一个辅助源点，从虚拟顶点到每个顶点都有一条权重为0的边。从辅助源点开始执行 Bellman-Ford 算法或者 SPFA 算法，$$d[i]$$即为$$x_i​$$的解。差分约束系统的解有多组，通常是按照给定的其他条件求出一组特解。

利用 Bellamn-Ford 算法求解的伪代码：

```pseudocode
# initialization
for each v in V do 
    d[v] ← ∞; 
d[source] ← 0
# Relaxation
for i =1,...,|V|-1 do
    for each edge (u,v) in E do
        d[v] ← min{d[v], d[u]+w(u,v)}
# Negative cycle checking
for each edge (u, v) in E do 
    if d[v]> d[u] + w(u,v) then 
        no solution
```

SPFA 算法的伪码：

```pseudocode
 procedure Shortest-Path-Faster-Algorithm(G, s)
      for each vertex v ≠ s in V(G)
          d(v) := ∞
      d(s) := 0
      # Q is a FIFO queue
      offer s into Q
      while Q is not empty
          u := poll Q
          for each edge (u, v) in E(G)
              if d(u) + w(u, v) < d(v) then
                  d(v) := d(u) + w(u, v)
                 if v is not in Q then
                     offer v into Q
```

但在解决差分系统问题时，SPFA 与常规的版本不同。前面提到解决这类问题时要引入一个辅助源点，为了在代码中省略这一步，需要先把所有顶点入队，把距离置为0，然后就是常规的对边进行松弛的操作。

**常见变形**

（1）求最小值 => 转换为求最长路径

将不等式变形为$${\displaystyle x_{i}-x_{j}≥ b_{k}}$$的形式，建立$$j->i$$的有向边，权重为$$b_k$$。如果不等式组中有$$x_i – x_j > k$$，因为一般题目都是对整形变量的约束，化为$$x_i – x_j ≥ k+1$$即可。如果有$$x_i – x_j = k$$，那么可以变为如下两个：$$x_i – x_j ≥ k$$, $$x_j – x_i ≥ -k$$，建立两条边即可。

（2） 求最大值 => 转换为求最短路径

将不等式变形为$${\displaystyle x_{j}-x_{i}\leq b_{k}}$$的形式，建立$$i->j$$的有向边，权重为$$b_k$$。如果存在非标准形式的不等式，采用与上面类似的方法进行标准化。

（3）是否有解 => 转换为判断是否存在环

如果是求最短路径，判断有无负环；如果是求最长路径，判断有无正环。用 SPFA 算法判断环，n 个点中如果同一个点入队超过 n 次，那么即存在环。

## 真题

### [201809-4](http://118.190.20.162/view.page?gpid=T76)

问题描述

　　在一条街上有n个卖菜的商店，按1至n的顺序排成一排，这些商店都卖一种蔬菜。
　　第一天，每个商店都自己定了一个正整数的价格。店主们希望自己的菜价和其他商店的一致，第二天，每一家商店都会根据他自己和相邻商店的价格调整自己的价格。具体的，每家商店都会将第二天的菜价设置为自己和相邻商店第一天菜价的平均值（用去尾法取整）。
　　注意，编号为1的商店只有一个相邻的商店2，编号为n的商店只有一个相邻的商店n-1，其他编号为i的商店有两个相邻的商店i-1和i+1。
　　给定第二天各个商店的菜价，可能存在不同的符合要求的第一天的菜价，请找到符合要求的第一天菜价中字典序最小的一种。
　　字典序大小的定义：对于两个不同的价格序列(a1, a2, ..., an)和(b1, b2, b3, ..., bn)，若存在i (i>=1), 使得ai<bi，且对于所有j<i，aj=bj，则认为第一个序列的字典序小于第二个序列。

输入格式

　　输入的第一行包含一个整数n，表示商店的数量。
　　第二行包含n个正整数，依次表示每个商店第二天的菜价。

输出格式

　　输出一行，包含n个正整数，依次表示每个商店第一天的菜价。

样例输入

8
2 2 1 3 4 9 10 13

样例输出

2 2 2 1 6 5 16 10

数据规模和约定

　　对于30%的评测用例，2<=n<=5，第二天每个商店的菜价为不超过10的正整数；
　　对于60%的评测用例，2<=n<=20，第二天每个商店的菜价为不超过100的正整数；
　　对于所有评测用例，2<=n<=300，第二天每个商店的菜价为不超过100的正整数。
　　请注意，以上都是给的第二天菜价的范围，第一天菜价可能会超过此范围。

**解法**

根据题意列出不等式（假设有5家店，$$a_i$$为第二天的菜价，$$x_i$$为第一天的菜价）：

```
a1* 2≤ x1+x2 ≤ a1* 2+1
a2* 3≤ x1+x2+x3 ≤ a2* 3+2
a3* 3≤ x2+x3+x4 ≤ a3* 3+2
a4* 3≤ x3+x4+x5 ≤ a4* 3+2
a5* 2≤ x4+x5 ≤ a5* 2+1
x1≥1
x2≥1
x3≥1
x4≥1
x5≥1
```

引入中间变量$$s_i=x_0+x_1+…+x_i,x_0=0$$，即可将上面的第一个不等式转化成： $$a_1* 2≤ s_2-s_0 ,a_1* 2+1≤s_0-s_2$$，其它的也一样。因为要求字典序最小，即求最小值，那么全部转化成大于等于的形式，然后求解最长路径。下面用 SPFA 算法求解。

```java
import java.util.*;

public class Main {
    // 比最大顶点数稍大
    private static final int N = 300 + 10;
    // 比估计的最大边数稍大
    private static final int M = 2000 + 10;

    private static Edge[] edges = new Edge[M];

    // head[i]： 顶点 i 是 head[i] 这条边的起点
    private static int[] head = new int[N];

    private static int cur = 0;

    private static int[] dist = new int[N];

    private static int[] prices = new int[N];

    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        int n = in.nextInt();
        for (int i = 1; i <= n; i++) {
            prices[i] = in.nextInt();
        }

        Arrays.fill(head, -1);

        // 对前两个菜价单独处理
        addEdge(0, 2, 2 * prices[1]);
        addEdge(2, 0, -(2 * prices[1] + 1));
        // 对末尾两个菜价单独处理
        addEdge(n - 2, n, 2 * prices[n]);
        addEdge(n, n - 2, -(2 * prices[n] + 1));
        // 中间的菜价统一处理
        for (int i = 2; i < n; i++) {
            addEdge(i - 2, i + 1, 3 * prices[i]);
            addEdge(i + 1, i - 2, -(3 * prices[i] + 2));
        }
        // 每一个菜价大于等于1
        for (int i = 1; i <= n; i++) {
            addEdge(i - 1, i, 1);
        }

        // 求最长路径
        spfa(n);

        for (int i = 1; i <= n; i++) {
            prices[i] = dist[i] - dist[i - 1];
        }
        for (int i = 1; i < n; i++) {
            System.out.print(prices[i] + " ");
        }
        System.out.println(prices[n]);
    }

    private static void spfa(int n) {
        Queue<Integer> q = new LinkedList<>();
        for (int i = 0; i <= n; i++) {
            q.offer(i);
            dist[i] = 0;
        }
        while (!q.isEmpty()) {
            int u = q.poll();
            for (int i = head[u]; i != -1; i = edges[i].next) {
                int v = edges[i].to;
                // 求最长路径
                if (dist[v] < dist[u] + edges[i].weight) {
                    dist[v] = dist[u] + edges[i].weight;
                    if (!q.contains(v)) {
                        q.offer(v);
                    }
                }
            }
        }
    }

    private static void addEdge(int from, int to, int weight) {
        edges[cur] = new Edge(to, weight, head[from]);
        head[from] = cur;
        cur++;
    }
}

class Edge {
    // 边的终点
    int to;
    // 边的权重
    int weight;
    // 下一条要处理的边
    int next;

    public Edge(int to, int weight, int next) {
        this.to = to;
        this.weight = weight;
        this.next = next;
    }
}
```

# 欧拉回路

如果一个图能够[遍历](https://zh.wikipedia.org/wiki/%E5%9B%BE%E7%9A%84%E9%81%8D%E5%8E%86)完所有的边而没有重复，这样的图就称为**欧拉图**。这时遍历的路径称作**欧拉路径**（一个[环](https://zh.wikipedia.org/wiki/%E5%9B%BE%E8%AE%BA)或者一条链），如果路径闭合（一个圈），则称为**欧拉回路**。

（1）无向图

连通的无向图 G 有欧拉路径的充要条件是：G中奇顶点（连接的边数量为奇数的顶点）的数目等于0或者2。如果有2个奇顶点，那么一个为欧拉路径的起点，另一个为终点。

连通的无向图 G 是欧拉[环](https://zh.wikipedia.org/wiki/%E5%9B%BE)（存在欧拉回路）的充要条件是：G中每个顶点的度都是偶数。

（2）有向图

连通的有向图存在一条从顶点 u到 v 的（不闭合的）欧拉路径的充要条件是： u的出度比入度多1，v 的出度比入度少1，而其它顶点的出度和入度都相等。

连通的有向图是欧拉回路的充要条件是每个顶点的出度和入度都相等。

首先判断图是否连通（用 union find），再利用欧拉定理判断出一个图存在欧拉通路或回路后，选择一个正确的起始顶点后 DFS 遍历所有的边（每条边只遍历一次），在搜索前进方向上将遍历过的边按顺序记录下来。这组边就是一条欧拉通路或回路。

## 真题

### 20151204

问题描述

　　为了增加公司收入，F公司新开设了物流业务。由于F公司在业界的良好口碑，物流业务一开通即受到了消费者的欢迎，物流业务马上遍及了城市的每条街道。然而，F公司现在只安排了小明一个人负责所有街道的服务。
　　任务虽然繁重，但是小明有足够的信心，他拿到了城市的地图，准备研究最好的方案。城市中有*n*个交叉路口，*m*条街道连接在这些交叉路口之间，每条街道的首尾都正好连接着一个交叉路口。除开街道的首尾端点，街道不会在其他位置与其他街道相交。每个交叉路口都至少连接着一条街道，有的交叉路口可能只连接着一条或两条街道。
　　小明希望设计一个方案，从编号为1的交叉路口出发，每次必须沿街道去往街道另一端的路口，再从新的路口出发去往下一个路口，直到所有的街道都经过了正好一次。

输入格式

　　输入的第一行包含两个整数*n*, *m*，表示交叉路口的数量和街道的数量，交叉路口从1到*n*标号。
　　接下来*m*行，每行两个整数*a*, *b*，表示和标号为*a*的交叉路口和标号为*b*的交叉路口之间有一条街道，街道是双向的，小明可以从任意一端走向另一端。两个路口之间最多有一条街道。

输出格式

　　如果小明可以经过每条街道正好一次，则输出一行包含*m*+1个整数*p*1, *p*2, *p*3, ..., *pm*+1，表示小明经过的路口的顺序，相邻两个整数之间用一个空格分隔。如果有多种方案满足条件，则输出字典序最小的一种方案，即首先保证*p*1最小，*p*1最小的前提下再保证*p*2最小，依此类推。
　　如果不存在方案使得小明经过每条街道正好一次，则输出一个整数-1。

样例输入

4 5
1 2
1 3
1 4
2 4
3 4

样例输出

1 2 4 1 3 4

样例说明

　　城市的地图和小明的路径如下图所示。
![img](http://118.190.20.162/RequireFile.do?fid=HgNYQ5G9)

样例输入

4 6
1 2
1 3
1 4
2 4
3 4
2 3

样例输出

-1

样例说明

　　城市的地图如下图所示，不存在满足条件的路径。
![img](http://118.190.20.162/RequireFile.do?fid=67NLAqAY)

评测用例规模与约定

　　前30%的评测用例满足：1 ≤ *n* ≤ 10, *n*-1 ≤ *m* ≤ 20。
　　前50%的评测用例满足：1 ≤ *n* ≤ 100, *n*-1 ≤ *m* ≤ 10000。
　　所有评测用例满足：1 ≤ *n* ≤ 10000，*n*-1 ≤ *m* ≤ 100000。

# 树的直径



树的直径就是距离最远的两个节点之间的距离。

```
Input :     1
          /   \
        2      3
      /  \
    4     5

Output : 3(4->3)

Input :     1
          /   \
        2      3
      /  \ .    \
    4     5 .    6

Output : 4(4->6)
```

以二叉树为例，计算树的直径，思路：计算直径就是要找一条最长路径，而树中的最长路径一定经过根，所以最长路径的长度就是根节点的左子树的最大深度加上右子树的最大深度。

```java
class Solution {
    private int ans = 0;
    
    public int diameterOfBinaryTree(TreeNode root) {
        helper(root);
        return ans;
    }
    
    private int helper(TreeNode root) {
        if (root == null) {
            return 0;
        }
        int left = helper(root.left);
        int right = helper(root.right);
        ans = Math.max(ans, left + right);
        return 1 + Math.max(left, right);
    }
}
```

## 真题

### 20150304

问题描述

　　给定一个公司的网络，由*n*台交换机和*m*台终端电脑组成，交换机与交换机、交换机与电脑之间使用网络连接。交换机按层级设置，编号为1的交换机为根交换机，层级为1。其他的交换机都连接到一台比自己上一层的交换机上，其层级为对应交换机的层级加1。所有的终端电脑都直接连接到交换机上。
　　当信息在电脑、交换机之间传递时，每一步只能通过自己传递到自己所连接的另一台电脑或交换机。请问，电脑与电脑之间传递消息、或者电脑与交换机之间传递消息、或者交换机与交换机之间传递消息最多需要多少步。

输入格式

　　输入的第一行包含两个整数*n*, *m*，分别表示交换机的台数和终端电脑的台数。
　　第二行包含*n* - 1个整数，分别表示第2、3、……、*n*台交换机所连接的比自己上一层的交换机的编号。第*i*台交换机所连接的上一层的交换机编号一定比自己的编号小。
　　第三行包含*m*个整数，分别表示第1、2、……、*m*台终端电脑所连接的交换机的编号。

输出格式

　　输出一个整数，表示消息传递最多需要的步数。

样例输入

4 2
1 1 3
2 1

样例输出

4

样例说明

　　样例的网络连接模式如下，其中圆圈表示交换机，方框表示电脑：
![img](http://118.190.20.162/RequireFile.do?fid=F9GfBRHL)
　　其中电脑1与交换机4之间的消息传递花费的时间最长，为4个单位时间。

**解法**

显然这个题就是要求树的直径，只不过这是一个多叉树。

```java
import java.math.BigDecimal;
import java.util.*;

public class Main {
    private static Map<Integer, Node> tree = new HashMap<>();

    private static BigDecimal maxLen = BigDecimal.ZERO;

    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        int n = in.nextInt();
        int m = in.nextInt();
        Node root = new Node(1);
        tree.put(1, root);
        for (int u = 2; u <= n + m; u++) {
            int v = in.nextInt();
            if (!tree.containsKey(v)) {
                tree.put(v, new Node(v));
            }
            if (!tree.containsKey(u)) {
                tree.put(u, new Node(u));
            }
            tree.get(v).children.add(tree.get(u));
        }
        depth(root);
        System.out.println(maxLen);
    }

    private static BigDecimal depth(Node root) {
        if (root == null) {
            return BigDecimal.ZERO;
        }
        List<BigDecimal> tmp = new LinkedList<>();
        for (Node child : root.children) {
            tmp.add(depth(child));
        }
        // 如果大于该节点的孩子多于2，只取最深的两棵子树
        if (tmp.size() > 2) {
            Collections.sort(tmp, Collections.reverseOrder());
            tmp = tmp.subList(0, 2);
        }
        BigDecimal sum = BigDecimal.ZERO;
        BigDecimal max = BigDecimal.ZERO;
        for (BigDecimal b : tmp) {
            sum = sum.add(b);
            if (max.compareTo(b) < 0) {
                max = b;
            }
        }
        if (sum.compareTo(maxLen) > 0) {
            maxLen = sum;
        }
        return max.add(BigDecimal.ONE);
    }
}

class Node {
    int id;

    List<Node> children = new LinkedList<>();

    public Node(int id) {
        this.id = id;
    }
}
```

100分。