# Basic Syntax

## expression

All Clojure code is written in two forms:

- Literal value of data structures such as numbers, strings, maps etc.
- `(operator operand1 operand2 ... operandn)`

We rarely write literals alone because they will do nothing on their own. So we focus on writing complete expressions which take the second form. 

Note: Clojure uses **whitespaces** to separate operands and it treats commas as whitespaces. Also, don't forget opening and closing parentheses. 

```clojure
(+ 1 2) ; => 3
```

## control flow

### 1. if

```clojure
(if boolean-expr
    then-expr
    optional-else-expr)
```

```clojure
(if true
    (+ 1 2)
    "I can't do nothing.") ; => 3

(if false
    (+ 1 2)
    "Let's do it.") ; => "Let's do it."
```

Notice that both then-branch and else-branch can contain only ONE experssion. 

For example in Java :

```java
if (true) {
    // then-branch
    int a = 1;
    a++;
} else {
    // else-branch
    int a = 1;
    a--;
}
```

But in Clojure, we can not write codes in this way.

### 2. do

`do` operator lets us wrap up multiple expressions in parentheses and run each of them.

```clojure
(if true
    (do (+ 1 2) (println "hello"))
    (do (- 1 1) (println "world"))
```

### 3. when

`when` operator acts like the combination of `if` and `do` but without else-branch.

```clojure
(when true
    (print "hello,")
    (println "world."))
```

### 4. loop 

```clojure
(loop [x 10]
  (when (> x 1)
    (println x)
    (recur (- x 2))))

;;=> 10 8 6 4 2
```

You can see firstly we bind x to 10, which we often call "initializing counter" in other languages. Then we evaluate the expressions in a lexical context in which the symbols in the binding-forms are bound to their respective init-exprs or parts therein. Acts as a recur target. 

The code below may be kind of confused:

```clojure
(loop [x 0]
    (if (< x 3)
        (println x)
        (recur (inc x))))

;;=> 0
```

In this case, Clojure will print x only once because the true branch of if expression can only be evaluate once. In a loop, we can put the expressions we want to evaluate every round before the condition expression.

```clojure
(loop [x 0]
    (println x)
    (if (>= x 2)
        (println "Bye~")
        (recur (inc x))))

;;=> 0 1 2 Bye~
```

### 5. bool values

- `=` is equality operator and `not=` is not equality.
- Both `nil` and `false` represent logical falsiness whereas all other values are logically truthy
- `nil?` is a function check if a value is `nil`
- `or` operator returns either the first truthy value or the last value
- `and`operator returns the first truthy value or the last truthy value if no values are falsey.

## naming values

In Clojure we don't say assigning a value to a variable. We just bind a value to a name using `def`.

```clojure
(def x 1)
```

We should treat `def` as if it defines **constants**.

# Data Structures

All of Clojure's data structures are **immutable**.

## number

Clojure has pretty sophisticated numerical support. 

```clojure
93
1.2
1/5 ; Clojure represent ratios directly
```

## string

Clojure only allows double quotes to declare strings. Clojure doesn't have string interpolation and it only allows concentration via the `str` function.

```clojure
(str "hello " "world")
```

## map

Map values can be any type.

Maps can be nested.

1.hash map

There are two ways to construct a hash map.

```clojure
{"plus-function" +}  ; a map literal 

(hash-map "plus-function" +) ; use hash-map function
```

(1) look up values

We can look up values with `get` function.

```clojure
(def myMap (hash-map "plus +"))  ; bind myMap to a hash map

((get myMap "plus") 1 2)  ; output 3
```

If a key doesn't bind to a value, Clojure will return `nil` or you can specify a default value.

```clojure
(get myMap "plus" +) ; functon + is the default value
```

The `get-in` function can look up values in nested maps.

```clojure
(get-in {"name" {"first" "mason" "last" "maxwell"}} ["name" "first"]) ; output "mason"
```

Actually you can treat a map as a function and keys as its arguments.

```clojure
({"name" "mason" "age" 18} "name") ; output "mason"
```

(2) keyword

Clojure's keywords have a special form: `:key`. You can use it as a function, which acts like `get` function. This is a brief and self-expressed way.

```clojure
(:a {:a 1 :b 2}) ; output 1
```

2.sorted map

## vector

A vector is like an array and it's a 0-indexed collection.

```clojure
[1 2 3]
```

(1) create

```clojure
[1 2 3] ; literal

(vector 1 2 3) ; use vector function

(conj [1 2 3] 4) ; use conj function to add elements
```

(2) look up values

We can use `get` function to look up values with given indices.

```clojure
(get [1 2 3] 0) ; output 1
```

## list

(1) create

```clojure
'(1 2 3) ; literal

(list 1 2 3) ; use list function

(conj '(1 2 3) 4) ; use conj function to add elements
```

(2) look up values

```clojure
(nth '(1 2 3) 0) ; output 1
```

NOTE: using `nth`to retrieve a value from a list is slower than using `get`to retrieve a value from a vector.

## set

1.hash set

(1) create

```clojure
#{1 2 3} ; literal

(hash-set 1 2 2 3) ; use hash-set function
                   ; the result is #{1 2}
 
(set [1 2 3]) ; convert a vector to a set 
                   
(conj #{1} 2) ; use conj function to add elements
```

(2) look up values

```clojure
(contains? mySet :a) ; whether the set contains :a

(get mySet :a) ; get :a

(:a mySet) ; if the set contains then return it
```

2.sorted set

# Function

## call functions

Calling a function is also a valid expression whose operator is the function you want to call.

A common error is `<x> cannot be cast to clojure.lang.IFn`, which means you are trying to use something as a function while it's not.

Of course Clojure supports high-order functions.

```clojure
(map inc '(1 2 3)) ; (2 3 4)

(filter even? '(1 2 4)) ; (2)
```

NOTE: special expressions like `if` and macros can not be used as arguments.

## define functions

- `defn`
- function name
- an optional string describing the function
- parameters 
- function body

```clojure
(defn foo
    "do something"
    [x]
    (println x))
```

### 1. docstring

the docstring is a useful way to document your function.

### 2. parameters

```clojure
(defn no-param
    []
    "I take no parameters.")  ; 0 parameter

(defn specify-number-params
    [x y z]
    (str "I take 3 parameters: " x ", " y ", " z)) ; specify the number of parameters

(defn arbitary-params
    [& params]
    (str "I take arbitary parameters: " params)) ; parameters will be collected in a sequence and notice that there must be a space between & and the name of sequence
    
```

Clojure supports arity overloading, which means you can define a function so a different function body will run depending on the arity.

```clojure
(defn multi-arity
    ;; 3-arity
    ([x y z] (println x y z))
    ;; 2-arity
    ([x y] (println x y))
    ;; 1-arity
    ([x] (println x)))
```

Arity overloading is one way to provide default values for arguments. In this way you can define a function in terms of itself:

```clojure
(defn messenger
  ([]     (messenger "Hello world!"))
  ([msg]  (println msg)))
```

In the above code, o-arity `message` calls 1-arity `message`.

### 3. destructuring

```clojure
(defn chooser
    [[_1st _2nd & other]]
    (str "the first: " _1st)
    (str "the second: " _2nd)
    (str "the rest: " other))

(chooser [1 2 3 4 5])
;; output 
;; the first: 1
;; the second: 2
;; the rest: (3 4 5)
```

What happens in the above code is that the function receives a sequence as an argument and then the function will destruct the sequence to associate meaningful names with different parts of the sequence. 

```clojure
(defn location
    [{lat :lat lng :lng}]
    (str "latitude: " lat " longitude: " lng))
```

You can also destruct maps. In the above code, we associate names with values corresponding to keys.

```clojure
(defn smart-location
    [{:keys [lat lng] :as location}]
    (str "latitude: " lat " longitude: " lng)
    (println location))
```

We can retain access with maps using `:as`keyword.

## anonymous function

### 1. `fn`

```clojure
(fn [x] (str "this is an anonymous function named " x))
```

### 2. `#`

```clojure
(#(+ %1 %2) 1 2)
```

We can use `#` to create an anonymous function. `%` indicates the argument passed to the function. If the anonymous function takes multiple arguments, we can distinguish them like `%1`,`%2` and so on. We can also pass a variable parameter with `%&`.

## let

The form of let-expression is:

```clojure
(let [binding-exxpr] (body))
```

An example:

```clojure
(let [x 1] x) ; output 1
```

Here we bind 1 to the name x and then we return x.

You can also destruct a sequence:

```clojure
(let [[_1st & _rest] a_vector] [_1st _rest])
```

We bind the first value of vector to the name _1st and we use rest parameters to store rest elements. 

There two main usages of `let`.

### 1. create a local scope

```clojure
(def x 1)
(let [x 0] x) ; output 0
```

The x in the `let` expression equals 0 not 1. Notice that the names declared in `let` expression can only be used in `let` expression.

### 2. reduce the cost 

`let` allows you to evaluate an expression only once and reuse the result. This is essentially important when you need to reuse the result of an expensive function call.

# Abstraction

You can think of abstractions as named collections of operations. If you can perform all of an abstraction's operations on an object, then the object is an instance of the abstraction.

## seq functions

If a data structure can support following three operations, then we say this data structure is a sequence or simply a seq:

- first: get the first element in the data structure
- rest: get the rest elements in the data structure
- cons: add a new element to the head of the data structure

Clojure treats lists, vectors, sets and maps as sequences, which means in Clojure the term "sequence" can have multiple meanings. In programming, indirection is a generic term for the mechanisms a language employs so that one name can have multiple, related meanings. 

> Polymorphism is one way that Clojure provides indirection. Basically, polymorphic functions dispatch to different function bodies based on the type of the argument supplied.

When it comes to sequences, Clojure will do some type conversion to achieve indirection.

```clojure
(seq '(1 2 3)) ;;=> (1 2 3)

(seq [1 2 3]) ;;=> (1 2 3)

(seq #{1 2 3}) ;;=> (1 2 3)

(seq '(1 2 3)) ;;=> (1 2 3)
```

By the way, we can use `into`to convert a sequence to another data structure, we say vector: 

```clojure
(into {} (seq {:name "mason"})) ;;=> {:name "mason"}
```

Clojure has a seq library full of useful functions.

(1) map

`map`function can take multiple sequences:

```clojure
(map str ["a" "b"] ["A" "B"]) ;;=> ("aA" "bB")
```

When we pass `map`multiple sequences, the elements of the first sequence will be passed as the first argument of the mapping function, the elements of the second sequence will be passed as the second argument and so on. Just make sure that mapping function can take a number of arguments equal to the number of sequences you pass to `map`.

(2) reduce

Just like the reduce function you can see in other languages such as Scala:

```clojure
(reduce + [1 2 3]) ;;-> 6
```

(3) take, drop, take-while, drop-while

`take`returns the first n elements of a sequence while `drop`returns the sequence without the first n elements.

```clojure
(take 3 [1 2 3 4 5 6]) ;;=> (1 2 3)
(drop 3 [1 2 3 4 5 6]) ;;=> (4 5 6)
```

`take-while`and`drop-while`use a predicate function to determine whether it should stop taking or dropping. 

```clojure
(take-while #(< % 3) [1 2 3 4 5 6]) ;;=> (1 2)
(drop-while #(< % 3) [1 2 3 4 5 6]) ;;=> (3 4 5 6)
```

(3) filter and some

Of course `filter`returns all elements test true for a predicate function.

```clojure
(filter #(< % 3) [1 2 3 4 5 6]) ;;=> (1 2)
```

`some`tells you whether a sequence contains elements test true for a predicate function and returns the first true value which is returned by the predicate function:

```clojure
(some #(and (< % 3) %) [1 2 3 4]) ;;=> 1
(some #(> % 10) [1 2 3 4]) ;;=> nil
```

(4) sort, sort-by

`sort`will sort elements in ascending order.

```clojure
(sort [3 1 2]) ;;=> (1 2 3)
```

`sort-by`will allow you to apply a key function to the sequence and use the values it returns to determine the sorting order.

```clojure
(sort-by count ["aaa" "v" "vv"]) ;;=> ("v" "vv" "aaa")
```

(5) concat

```clojure
(concat [1 2] [3 4]) ;;=> (1 2 3 4)
```

## lazy sequence

A lazy sequence's members aren't computed until you try to access them. 

You can think of a lazy sequence consisting of two parts: a recipe for how to generate the elements of a sequence and the elements which have been generated so far.

```clojure
(defn even-numbers
    ([] (even-numbers 0))
    ([n] (cons n (lazy-seq (even-numbers (+ n 2))))))

(take 10 (even-numbers))
;;=> 0 2 4 6 8 10 12 14 16 18
```

## collection abstraction

All of Clojure's core data structure-vector, map, list and set-take part in both sequence abstraction and collection abstraction. The sequence abstraction is about operating on members individually, whereas the collection abstraction is about the data structure as a whole.

(1) into 

`into`can take two collections and add all the elements from the second to the first no matter what types the two collections are.

```clojure
(into [] {:a 1 :b 2}) ;;=> [[:a 1] [:b 2]]

(into [0] [1 2 3]) ;;=> [0 1 2 3]
```

(2) conj

`conj`is similar to `into`but the second argument of `conj`is a rest parameter.

```clojure
(conj [0] 1 2 3) ;;=> [0 1 2 3]

(conj {:a 1} [:b 2] [:c 3]) ;;=> {:a 1, :b 2, :c 3}
```

## function functions

###  1. apply

`apply`can turn a collection to separate arguments.

```clojure
(max [1 2 3]) ;;=> [1 2 3]

(apply max [1 2 3]) ;;=> 3
```

### 2. partial

`partial`takes a function and any number of arguments. It then returns a new function which calls the original function with the original arguments you supplied it along with the new arguments.

```clojure
(def add10 (partial + 10))
(add10 3) ;;=> 13
```

In general, we can use `partial`when we find we are repeating the same combination of function and arguments in many different contexts. 

### 3. complement

`complement`only creates the negation of a Boolean function:

```clojure
(def my-pos? #(> % 0))
(def my-neg? (complement my-pos?))
(my-neg? -1) ;;=> true
```

