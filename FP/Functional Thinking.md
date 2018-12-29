> OO makes code understandable by encapsulating moving parts. FP makes code understandable by minimizing moving parts. 

# Shift of Paradigm

Traditional imperative programming relies on the low-level details of how iteration, transformation and reduction word. However, functional programming thinks about a higher level of abstraction. 

- It encourages you to **categorize problems differently, seeing commonalities**.
- It allows the runtime to be more intelligent about optimization. 
- It allows solutions that aren't possible when the developer is elbow deep in the details of the engine. 

## 1. Higher-order Function

Higher-order functions are a kind of higher-order abstraction., which can eliminate friction. It's actually syntactic sugar plus more.(Yet syntactic sugar is also important because syntax is the way you express your ideas in a language.)

## 2. Common Building Blocks

These common functions are useful in functional languages and frameworks.

### filter 

`filter` functions produce a subset of a collection based on supplied filtering criteria.

**Scala**

- `filter`
- `find`: returns the first matched value which is wrapped in a `Optional`
- `partition`: partitions a collection in two subset based on criteria
- `takewhile`: returns the largest subset of values from the front of the collection that satisfy the prediction 
- `dropwhile`: skips the largest number of elements that satisfy the predicate

### map

`map`functions transform a collection into a new one by suppling a function to each of the elements. The returned collection is the same size as the original collection but with the updated values.

**Scala**

- `map`
- `flatmap`: when you struggle with nested lists, use `flatmap` instead.

### reduce/fold

`fold`functions are used when you need to supply an initial value for the accumulator , whereas`reduce`functions start with nothing. 

**Scala**

- `reduceLeft`/`reduceRight`
- `foldLeft`/`foldRight`

# Cede Low-level Details

One of the values of functional thinking is the ability to cede control of low-level details to the runtime. 

## 1. Iteration to Higher-order Functions

If you can express which operation you want to perform in a higher-order function, the language will apply it efficiently.  

That doesn't mean that developers should cede all responsibility for understanding what happens in low-level abstractions. In many cases, you still must understand the implications of using abstractions like Java 8 Stream. 

## 2. Closures 

A closure is a function that carries an implicit binding to all the variables referenced within it. From an implementation standpoint, the closure instance holds onto an encapsulated copy of whatever is in scope when the closure is created. However, it's a bad idea to create a closure just so that you can manipulate its interior state. Binding constant or immutable values is more common. In this way, you can make the language manage state for you.

Closures are also an excellent way of deferred execution. For example, the correct variables or functions might not be in scope at definition time but are at execution time. By wrapping the execution context in a closure, you can wait until the proper time to execute it. 

```scala
def mulBy(factor: Double) = (x: Double) => x * factor
val triple = mulBy(3)
val double = mulBy(2)
triple(3) // 9
double(2) // 4
```

## 3. Currying and Partial Application 

- Currying describes the conversion of a multiargument function into a chain of single-argument functions. It describes the transformation not the invocation of the converted function. 
- Partial application describes the conversion of a multiargument function into one that accepts fewer arguments, with values for the elided arguments supplied in advance. It partially applies some arguments to a function, returning a function with a signature consisting of the remaining arguments.

For example, the fully curried version of the `process(x, y, z)`function is `process(x)(y)(z)`. Using partial application for a single argument on `process(x, y, z)`yields a function that accept two arguments: `process(y, z)`.

**Scala**

(1) curry

```scala
def modN(N: Int)(x: Int) = ((x % N) == 0)
```

(2) partial application

```scala
def price(product: String) = {
  product match {
    case "apple" => 140
    case "orange" => 223
  }
}

def withTax(cost: Double, state: String) = {
  state match {
    case "NY" => cost * 2
    case "FL" => cost * 3
  }
}

val locallyTaxed = withTax(_, "NY")
val costOfApples = locallyTaxed(price("apple"))
println(costOfApples) // 280.0
```

(3) partial (constrained) functions

The Scala `PartialFunction` trait is designed to work seamlessly with patter matching. You can use it to define a function which works only for a defined subset of values and types.

Partial function are different from partially applied functions. Partial functions have a limited range of allowable values, such as `1/x`.

```scala
val isOdd: PartialFunction[Int, Any] = {
    case x if x % 2 != 0 => x + " is odd."
}
```

A more detailed example:

```scala
    val answerUnit = new PartialFunction[Int, Int] {
      def apply(d: Int): Int = 42 / d

      def isDefinedAt(d: Int): Boolean = d != 0
    }
```

Implementation of the `PartialFunction`trait who use `case`can call `isDefinedAt`which is implicitly defined. 

**Common Uses**

Currying and partial application do have a place in real-world programming. 

(1) Function factories 

```scala
def adder = (x: Int, y: Int) => x + y
def increment = adder.curried(1)
println(s"increment 7: ${increment(7)}") // 8
```

(2) Template Method design pattern

Using partial application to supply known behavior and leaving the other arguments free for implementation specifics mimics the implementation of this OO design pattern.

(3) Implicit values

you can use currying to supply implicit values. For example, when you interact with a persistence framework, you must pass the data source as the first argument. By using partial application, you can supply the value implicitly.

## 4. Recursion 

Many functional languages think of a list as a combination of the first (the head) element plus the remainder of the list (the tail) instead of thinking of a list as indexed slots. Thinking about a list  as head and tail allow us to iterate through it recursively. 

```scala
def recurseList(list: List[Any]): Unit = {
  if (list.isEmpty) {
    return
  }
  println(s"${list.head}")
  recurseList(list.tail)
}

def filterR(list: List[Int], pred: Int => Boolean): List[Int] = {
  if (list.isEmpty) {
    list
  } else if (pred(list.head)) {
    filterR(list.tail, pred).::(list.head)
  } else {
    filterR(list.tail, pred)
  }
}
```

In the imperative way, we must mind the state. In the recursive way, the language manages the return value, building it up on the **stack** as the recursive return for each method invocation. 

Recursion does illustrate an important trend in programming: offloading moving parts by ceding it to the runtime. If I'm never allowed to touch the intermediate results of the list, I cannot introduce bugs in the way that I interact with it. 

## 5. Streams and Work Reordering 

A Stream acts in many ways like a collection, but it has no backing values, and instead uses a stream of values from a source to a destination. Some operation such as `map`and `filter`are lazy, meaning that they defer execution as long as possible. In fact, they don't try to produce results until a downstream terminal "asks" for them. 

For the lazy operations, intelligent runtimes can reorder the result of the operation for us in order to make the program more efficient. Therefore, we don't need to struggle with mundane details and we just focus on the problem domain. 

## Memoization & Laziness



 