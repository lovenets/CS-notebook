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

# Functional programming

## pure functions

A function is a pure if it meets two qualifications: 

- It always returns the same result if given the same arguments. This is called *reference transparency*.
- It can't cause any side effects. That is, the function can't make any changes that are observable outside the function itself. For example, by changing an externally accessible mutable object or writing to a file.

### 1. referentially transparent

To return the same result when called with the same argument, pure functions rely only on:

- their own arguments
- immutable values 

The following function is pure:

```clojure
(defn wisdom
    [words]
    (str words ", Daniel-san")
```

### 2. no side effects

To perform a side effect is to change the association between a name and its value within a given scope. Side effects are potentially harmful because they introduce uncertainty what the names in your code are referring to. This leads to situations where it's very difficult for you to trace why and how a name came to be associated with a value, which makes it hard to debug.

## immutable data structures

### 1. recursion instead of for/while

```clojure
(defn sum
    ([vals] (sum vals 0))
    ([vals accumulator] 
     (if (empty? vals)
         accumulator
         (sum (rest vals) (+ (first vals) accumulator)))))
```

Note that you should generally use `recur`considering performance.

```clojure
(defn sum
    ([vals] (sum vals 0))
    ([vals accumulator] 
     (if (empty? vals)
         accumulator
         (recur (rest vals) (+ (first vals) accumulator)))))
```

Using `recur`is quite important when you're recursively operating on a large collection. The reason is that Clojure doesn't support tail recursion optimization.

Also, Clojure's immutable data structures are implemented using structural sharing so don't worry about any crash because of GC or whatever.

### 2. function composition instead of attribute mutation

```clojure
(require '[clojure.string :as s])
(defn clean 
    [text]
    (s/replace (s/trim text) #"lol" "LOL"))
```

Instead of mutating an object, the `clean`function works by passing an immutable value `text`to a pure function `s/trim`which returns an immutable value. 

Combining functions like this--so that the return value of a function is passed as an argument to another--is called *function composition*. In general, functional programming encourages you to build more complex functions by combining simpler functions. 

## derive new functions

### 1. comp

Using math notations, we'd say that using `comp`on the functions $$f_1,f_2,...f_n$$, create a new function $$g$$ such that $$g(x_1,x_2,...x_n)=f_1(f_2(f_n(x_1,x_2,...,x_n)))$$. Notice that the first function can take any number of arguments, whereas the remaining functions must be able to take only one argument. 

```clojure
(def times-inc (comp inc *))
(times-inc 2 3) ;;=> 7
```

If one of the functions we want to compose needs to take more than on arguments, we wrap it in an anonymous function. 

### 2. memoize

Memoization lets you take the advantage of referential transparency by storing the arguments passed to a function and the returned value of the function. That way, subsequent calls to the function with the same arguments can return the result immediately. 

 ```clojure
(def memo-sum (memoize #(reduce + &%)))
 ```

# Clojure's Model

## Clojure's evaluation model

### conceptual model

Clojure (like all Lisps) reads textual source code, producing Clojure data structures. These data structures are then evaluated: Clojure traverses the data structures and performs actions like function application or var lookup based on the type of the data structure.

When compiling, Clojure evaluates trees ie. AST using Clojure **lists** and the nodes are Clojure values.

Lists are ideal for constructing trees. The first element of a list is used as the root and each subsequent  element is treated as a branch. 

![evaluation in Clojure](img\evaluation in Clojure.jpg)

We can evaluate custom data structures with `eval`:

```clojure
(eval '(+ 1 2)) ;;=> 3
```

### reader

We can read text without evaluating it and pass the result to other functions. Use `read-string`to do this:

```clojure
(list? (read-string "(+ 1 2)"))
```

Usually the reader will transform strings to corresponding data structures which is a one-to-one relationship. However, it just can be more complex.

#### reader macro

```clojure
(read-string "#(+ 1 %)") ;;=> (fn* [p1_423#] (+ 1 p1_423#))
```

In the above example, the reader uses a *reader macro* to transform the string "#(+ 1 %)". Reader macros are sets of rules for transforming text into data structures. They often allow you to represent data structures in more compact  ways because **they take an abbreviated reader form and expand it in to a full expression.** They're designated by macro characters like ', #, and @. 

### evaluator 

We can think of Clojure's evaluator as a function that takes a data structure as an argument, processes it using corresponding rules and return a result. For example, to evaluate a list, Clojure looks at the first element of the list and calls a function, macro or special expression. Any other values (including strings, numbers, and keywords) simply evaluate themselves. 

Let's say we type `(+ 1 2)`in the REPL. Because it's a list, the evaluator starts by evaluating the first element in the list i.e. plus symbol and the evaluator resolves that by returning the plus function. Because the first element is a function, the evaluator evaluates each of the operands. The operands 1 and 2 evaluate to themselves of course.  The the evaluator calls the plus function with 1 and 2 as the arguments and returns the result. 

#### things evaluated to themselves

Whenever Clojure evaluates data structures that aren't a list or symbol, the result is the data structure itself.

#### symbols

Clojures uses the term symbols to name functions, macros, data and anything else you can use, and evaluates them by resolving them. 

In general, Clojure resolves a symbol by:

1. Looking up whether the symbol names a expression e.g. `if` expression. If it doesn't...
2. Looking up whether the symbol corresponds to a local binding i.e. `let` expression. If it doesn't...
3. Trying to find a namespace mapping introduced by `def`. If it doesn't...
4. Throwing an exception.

#### lists

If the data structure is a non-empty list, it's evaluated as a call to the 1st element in the list. The way the call is performed depends on the nature of that 1st element. 

1.function calls

```clojure
(+ 1 2)
```

Clojure sees that the 1st element of list is a function so it proceeds to evaluate the rest of the elements in the list. The operands 1 and 2 both evaluate to themselves and are passed as arguments. 

2.special expressions 

Special expressions like `if`don't follow the same rules as normal functions. For example, when you call a function, each operand gets evaluated. However, with `if`you don't want each operand to be evaluated. 

Another important special expression is quote. 

```clojure
'(1 2 3)
```

In this case, Clojure will not evaluate the data structure after quote instead. It will just return the data structure itself. 

>  Special expressions are special because they implement the core behavior that can't be implemented with functions. 

3.macros

Macros give you a convenient way to manipulate lists before Clojure evaluates them. They are executed between the reader and the evaluator so they can manipulate the data structures that that reader spits out and transform with those data structures before passing them to the evaluator. 

```clojure
(defmacro ignore-last-operand
    [function-call]
    (butlast function-call))

(ignore-last-operand (+ 1 2 10))
;;=> 3

(ignore-last (+ 1 2 (println "What's up?")))
;;=> 3
```

When you call a macro, the operands are **not** evaluated. In particular, symbols are not resolved; they are passed as symbols. Lists are not evaluated either; that is, the 1st element in the list is not called as a function, special expression or macro. Rather, the unevaluated list data structure is passed in. 

Another difference is that the data structure returned by a function is not evaluated but the one returned by a macro is. The process of determining the return value of a macro is called macro expansion and we can use `macroexpand`to see what data structure a macro returns.

 ```clojure
(macroexpand '(ignore-last-operand (+ 1 2 3))) 
;;=> (+ 1 2)
 ```

### Syntax abstraction 

```clojure
(defmacro infix 
    [infixed]
    (list (second infixed)
          (first infixed)
          (last infixed)))

(infixed (1 + 2))
;;=> 3
```

What happened in the above example is pictured here:

![macro](img\macro.jpg)

This is called syntax abstraction used by Clojure to extend itself to fit into our program. 

```clojure
(defn read-resource
    [path]
    (read-string (slurp (clojure.java.io/resource path))))
```

This function is hard to understand. When we have too many nested parentheses, codes will be ugly. 

```clojure
(defn read-resource
    [path]
    (-> path
        clojure.java.io/resource
        slurp
        read-string))
```

`->`macro, aka the threading or stabby macro, lets us write our codes from top to bottom, which we are used to. 

# Macro

In general, macro definition look much like function definitions. They have a name, an optional document string, an argument list, and a body. The body will almost always return a list. This makes sense because macros are used to transform a data structure into a form Clojure can evaluate. 

One key difference between functions and macros is that functions argument are fully evaluated before they are passed to the function, whereas macros receive arguments as **unevaluated** data. 

## building lists for evaluation 

Macro writing is all about building a list for Clojure to evaluate. For one, we need to quote expressions to get **unevaluated data  structures**. More generally, we need to be extra careful about the difference between a symbol and its value.

 ```clojure
(defmacro my-print
    [expr]
    (list let [result expr]
          (list println result)
          result))
 ```

If we tried this code, we'd get an exception. The reason is that the macro body tries to get the *value* that the *symbol* let refers to, whereas we just want to return the let itself. There are other problems which are also caused by mixing symbols with values. Use the single quote character can make this work.

```clojure
(defmacro my-print
    [expr]
    (list 'let ['result expr]
          (list 'println 'result)
          'result))
```

Single quote character tells Clojure to turn off evaluation for whatever follows, in this case preventing Clojure from trying to resolve the symbols and instead just returning the symbols. 

### simple quoting 

We'll almost always use quoting within macros to obtain an unevaluated symbols. Here is the source code of macro `when`.

```clojure
(defmacro when
    {:added "1.0"}
    [test & body]
    (list 'if test (cons 'do body)))
```

Notice that the macro quotes both `if` and `do`. 

### syntax quoting 

Syntax quoting is more powerful. It can return unevaluated data structures similar to normal quoting. However, syntax quoting will return the fully qualified symbols (that is, with the symbol's namespace included). This can help you avoid namespace collision.

```clojure
`+
;;=> clojure.core/+
```

Syntax quoting a list will recursively quote all the elements. There is also a difference. Syntax quoting allows us to unquote the expressions using `~`. 

```clojure
`(+ 1 ~(inc 2))
;;=> (+ 1 3)
```

We can use syntax quoting to write concise codes.

```clojure
(defmacro code-critic
    [bad good]
    `(do (println "Great, this is bad code:"
                  (quote ~bad))
         (println "Bad, this is good code:"
                  (quote ~good))))
```

In this case, we want to quote everything except for the symbols `good`and `bad`. With syntax quoting, we can just wrap the whole `do`expression in a quote and simply unquote the two symbols we want to evaluate. 

 To sum up, our macros will mostly return lists. We can build up the list to be returned by using `list` function or by using syntax quoting. Syntax quoting usually leads to clearer and more concise code. It's important to distinguish a symbol and the value it evaluates to. And if we want macros to return multiple expressions make sure to wrap them in a `do`.

### unquote splicing 

```clojure
(defn criticize
  [criticism code]
  `(println ~criticism (quote ~code)))

(defmacro code-critic
  [bad good]
  `(do ~(map #(apply criticize %)
              [["good, this is bad:" bad] ["bad, this is good:" good]])))
```

If we run this, it will cause an exception `NullPointerException`. Let's expand the macro:

```clojure
(macroexpand '(code-critic (1 + 1) (+ 1 1)))
;;=> (do ((clojure.core/println "good, this is bad:" (quote (1 + 1))) (clojure.core/println "bad, this is good:" (quote (+ 1 1)))))
```

We can see that `println`function is executed and because it returns `nil`, `do`will try to call `nil`. But we don't want `println`to be executed before `do`is executed. We can use unquote splicing `~@`. Unquote splicing unwraps a seqable data structure, placing its contents directly within the enclosing syntax-quoted data structure. 

```clojure
`(+ ~@(list 1 2 3))
;;=> (clojure.core/+ 1 2 3)

`(+ ~(list 1 2 3))
;;=> (clojure.core/+ (1 2 3))
```

So we can use it to correct `code-critic`:

```clojure
(defmacro code-critic
  [bad good]
  `(do ~@(map #(apply criticize %)
              [["good, this is bad:" bad] ["bad, this is good:" good]])))

(macroexpand '(code-critic (1 + 1) (+ 1 1)))
;;=> (do (clojure.core/println "good, this is bad:" (quote (1 + 1))) (clojure.core/println "bad, this is good:" (quote (+ 1 1))))
```

Now `println`will be called by `do`.

## things to watch out for

### variable capture 

Variable capture occurs when a macro introduces a binding that eclipses an existing binding. 

```clojure
(def message "Good job!")

(defmacro with-mischief
    [& stuff-to-do]
    `(let [message "Oh, big deal!"]
         ~@stuff-to-do))

(with-mischief 
    (println "Here's how I feel about that thing you did: " message))
;;=> CompilerException java.lang.RuntimeException: Can't let qualified name: user/message
```

Syntax quoting can prevent you from accidentally capturing variables within macros. 

We can deal with this problem using `gensym`, which can produce an unique name:

```clojure
(gensym)
;;=>  G__1165
(gensym)
;;=> G__1168
(gensym "message") ;; give it a prefix
;;=> => message1171
```

There is a more concise way called `auto-gensym`:

```clojure
`(blarge# blarge#)
;;=> (blarge__1172__auto__ blarge__1172__auto__) 
```

Now we can correct the codes:

```clojure
(defmacro without-mischief
	[& stuff-to-do]
	(let [macro-message (gensym 'message)]
		`(let [~macro-message "Oh, big deal!"]
			~@stuff-to-do
			(println "I still need to say: " ~macro-message))))
(without-mischief
	(println "Here's how I feel about that thing you did: " message))
; => Here's how I feel about that thing you did: Good job!
; => I still need to say: Oh, big deal!
```

### double evaluation 

This problem occurs when a form passed to a macro as an argument gets evaluated more than once. 

```clojure
(defmacro report 
    [to-try]
    `(if ~to-try
         (println (quote ~to-try) "was successful:" ~to-try)
         (println (quote ~to-try) "was not successful:" ~to-try	)))

(report (do (Thread/sleep 1000) (+ 1 1)))
```

When we try to call `report`, the program will sleep for 2s and the reason can be seen below.

```clojure
(if (do (java.lang.Thread/sleep 1000) (clojure.core/+ 1 1))
    (clojure.core/println (quote (do (java.lang.Thread/sleep 1000) (clojure.core/+ 1 1))) "was successful:" (do (java.lang.Thread/sleep 1000) (clojure.core/+ 1 1))) 
    (clojure.core/println (quote (do (java.lang.Thread/sleep 1000) (clojure.core/+ 1 1))) "was not successful:" (do (java.lang.Thread/sleep 1000) (clojure.core/+ 1 1))))
```

We expand the expression `(report (do (Thread/sleep 1000) (+ 1 1)))`and find that `Thread/sleep 1000`gets evaluated twice: once right after `if`and again when `println`gets called. 

Here's how we could avoid this problem: 

```clojure
(defmacro report
    [to-try]
    `(let [result# ~to-try]
         (if result#
             (println (quote ~to-try) "was successful:" result#)
             (println (quote ~to-try) "was unsuccessful:" result#))))
```

By placing `to-try`in a let expression, we only evaluate that code once and bind the result to an auto-gensym'd symbol. 

### macros all the way down

We may have to write more and more macros to get anything done. This is a consequence of the fact that macro expansion happens before evaluation. 

