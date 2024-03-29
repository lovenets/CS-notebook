# 作用

大 O 表示法用来表示在__最坏情况下__算法执行==关键操作的次数==和操作对象数目之间的关系。大 O 表示法最重要的作用在于指示出算法运行时间的**增速**。

比如说对一个长度为 n 的数组而言，顺序查找算法最多需要比较 n 次，所以它的时间复杂度是 $O(n)$；二分查找最多需要比较 $logn$次，所以它的时间复杂度是 $O(logn)$。

# 算法速度

当讨论算法速度的时候，实际上是在说随着输入的增加，算法运行时间将以什么样的速度增加。因此 $O(logn)$ 的算法比 $O(n)$ 的算法快，实际上是指当 n 越大时前者比后者快得越多。

# 求解时间复杂度

步骤如下：

1.求解算法语句总的执行次数 T 关于输入规模（理解为操作数的个数）n 的函数 $F(n)$。

2.去掉 $F(n)$ 的常数项和低次项，只保留最高次项，而且最高次项的系数也要去掉，得到一个新的函数 $f(n)$。如果 $F(n)$ 本身就是一个常数函数，那么 $f(n) = 1$。 

3.算法的时间复杂度就是 $O(f(n))$。

从上面的步骤可以看出，虽然一开始需要计算算法所有语句总的执行次数，但是经过化简之后只留下了 $F(n)$ 的最高次项，所以按照我的理解就是可以只关注算法中执行次数最多的语句，一般也就是最关键的操作。

常见算法时间复杂度的比较：

$O(1) < O(logn) < O(n) < O(nlogn) < O(n^2) < O(n^3) < O(n!) < O(n^n) $

![排序算法时间复杂度](img\排序算法时间复杂度.jpg)

In general, doing something with every item in one dimension is linear, doing something with every item in two dimensions is quadratic, and dividing the working area in half is logarithmic (such as binary search). There are other Big O measures such as cubic, exponential, and square root, but they're not nearly as common. Big O notation is described as O ( ) where is the measure. The quicksort algorithm would be described as $$O ( N * log ( N ) )$$.