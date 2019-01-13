# Basic Syntax

## Basic Types

In Kotlin, everything is an object. Numbers, characters and booleans can be represented as primitive types at runtime but to the user they are look like ordinary classes. 

### 1. Numbers

| Type   | Bit Width | Literal       |
| ------ | --------- | ------------- |
| Double | 64        | 132.5         |
| Float  | 32        | 132.5f/123.5F |
| Long   | 64        | 132L          |
| Int    | 32        | 132           |
| Short  | 16        |               |
| Byte   | 8         |               |

(1) underscores in numeric literals 

You can use underscores to make number constants more readable.

```kotlin
val M = 1_000_000
```

(2) explicit conversions 

Smaller types can be implicitly converted to bigger types. This means that we cannot assign a value of type `Byte`to an `Int`variable without an explicit conversion. 

```kotlin
val b: Byte = 1
val i: Int = b.toInt() 
```

Every number type supports similar conversions. 

Arithmetical operations are overloaded for appropriate conversions.

```kotlin
val l = 1L + 3 // Long + Int => Long
```

(3) operations 

Kotlin supports the standard set of arithmetical operations over numbers which are declared as members of appropriate numbers. 

As of bitwise operations, there are no special characters for them but just named functions that can be called in infix form.

- `shl`: signed shift left
- `shr`: signed shift right
- `ushr`: unsigned shift right
- `and`: bitwise and
- `or`: bitwise or
- `xor`: bitwise xor
- `inv`: bitwise inversion

(4) comparing floating point numbers

Besides traditional comparisons, we also can check range:

```kotlin
1 in 0..10 // true because 1 is in [0, 10]
```

There are some special rules:

- `NaN`is considered equal to itself
- `NaN`is considered greater than any other element including `POSITIVE_INFINITY`
- `-0.0`is considered less than `0.0`

### 2. Characters 

Characters are represented by the type `Char` and they can not be treated directly as numbers. But we can explicitly convert a character to an `Int` number.

```kotlin
fun decimalDigital(c: Char): Int {
    if (c !in '0'..'9') {
        throw IllegalArgumentException("Out of range")
    }
    return c.toInt() - '0'.toInt()
}
```

### 3. Booleans

Booleans are boxed if a nullable reference is needed. 

### 4. Arrays 

Arrays in Kotlin are represented by the `Array`class, which has `get`and`set`function (that turn into `[]` by operator overloading conversions) and `size`property along with a few other useful member functions. 

(1) create

We can use a library function `arrayOf()`which takes elements as arguments. `arrayOfNulls()`function can be used to create an array of a given size filled with null elements. 

Another option is to use `Array`constructor that takes the array size and the function which can produce the initial value of each array element given its index.

```kotlin
// Creates an Array<String> with values ["0", "1", "4", "9", "16"]
val asc = Array(5, { i -> (i * i).toString() })
asc.forEach { println(it) }
```

(2) arrays of primitives 

Kotlin also has specialized classes to represent arrays of primitive types without boxing overhead: `ByteArray`, `ShortArray`, `IntArray` and so on. These classes have no inheritance relation to the `Array` class, but they have the same set of methods and properties. Each of them also has a corresponding factory function:

```kotlin
val x: IntArray = intArrayOf(1, 2, 3)
x[0] = x[1] + x[2]
```

### 5. Strings

Strings are represented by the type `String`. Strings are immutable. Elements of a string are characters that can be accessed by the indexing operation: `s[i]`. A string can be iterated over with a *for*-loop:

```kotlin
for (c in str) {
    println(c)
}
```

You can concatenate strings using the `+` operator. This also works for concatenating strings with values of other types, as long as the **first** element in the expression is a string:

```kotlin
val s = "abc" + 1
println(s + "def")
```

(1) escaped strings 

An escaped string is very much like a Java string. Escaping is done in the conventional way, with a backslash. 

```kotlin
val s = "Hello, world!\n"
```

(2) raw strings 

A raw string is delimited by a triple quote (`"""`), contains no escaping and can contain newlines and any other characters:

```kotlin
val text = """
    for (c in "foo")
        print(c)
"""
```

By default `|` is used as margin prefix, but you can choose another character and pass it as a parameter, like `trimMargin(">")`.

```kotlin
val text = """
    |Tell me and I forget.
    |Teach me and I remember.
    |Involve me and I learn.
    |(Benjamin Franklin)
    """.trimMargin()
// Tell me and I forget.
// Teach me and I remember.
// Involve me and I learn.
// (Benjamin Franklin)
```

(3) string templates 

A template expression starts with a dollar sign ($) and consists of either a simple name.

If you need to represent a literal `$` character in a raw string (which doesn't support backslash escaping), you can use the following syntax:

```kotlin
val price = """
${'$'}9.99
"""
```

## Packages 

A source file may start with a package declaration:

```kotlin
package foo.bar

fun baz() { ... }
class Goo { ... }

// ...
```

All the contents (such as classes and functions) of the source file are contained by the package declared. So, in the example above, the full name of `baz()` is `foo.bar.baz`, and the full name of `Goo` is `foo.bar.Goo`. If the package is not specified, the contents of such a file belong to "default" package that has no name.

### 1. Default Imports 

A number of packages are imported into every Kotlin file by default. Additional packages are imported depending on the target platform.

### 2. Imports 

We can import either a single name, e.g.

```kotlin
import foo.Bar // Bar is now accessible without qualification
```

or all the accessible contents of a scope (package, class, object etc):

```kotlin
import foo.* // everything in 'foo' becomes accessible
```

If there is a name clash, we can disambiguate by using *as* keyword to locally rename the clashing entity:

```kotlin
import foo.Bar // Bar is accessible
import bar.Bar as bBar // bBar stands for 'bar.Bar'
```

The `import` keyword is not restricted to importing classes; you can also use it to import other declarations:

- top-level functions and properties;
- functions and properties declared in [object declarations](https://kotlinlang.org/docs/reference/object-declarations.html#object-declarations);
- [enum constants](https://kotlinlang.org/docs/reference/enum-classes.html).

### 3. Visibility of Top-level Declarations

If a top-level declaration is marked *private*, it is private to the file it's declared in.

## Control Flow

### 1. if

*if* is an expression, i.e. it returns a value. 

*if* branches can be blocks, and the last expression is the value of a block:

```kotlin
val max = if (a > b) {
    print("Choose a")
    a
} else {
    print("Choose b")
    b
}
```

If you're using *if* as an expression rather than a statement (for example, returning its value or assigning it to a variable), the expression is required to have an `else` branch.

### 2. when

*when* replaces the switch operator of C-like languages.

```kotlin
when (x) {
    0, 1 -> print("x == 0 or x = 1")  // comma
    2 -> print("x == 2")
    parseInt(x) -> println("s encodes x") // not only constants
    in 1..10 -> println("x is in the range") // check a range
    in listOf(1, 2, 3) -> print("x is valid") // check a collection
    is String -> println("x is a string") // check its type
    else -> { // Note the block
        print("x is neither 1 nor 2")
    }
}
```

*when* matches its argument against all branches sequentially until some branch condition is satisfied. *when* can be used either as an expression or as a statement.

 *when* can also be used as a replacement for an *if*-*else* *if* chain. If no argument is supplied, the branch conditions are simply boolean expressions, and a branch is executed when its condition is true:

```kotlin
when {
    x.isOdd() -> print("x is odd")
    x.isEven() -> print("x is even")
    else -> print("x is funny")
}
```

Since Kotlin 1.3, it is possible to capture *when* subject in a variable using following syntax:

```kotlin
fun Request.getBody() =
        when (val response = executeRequest()) {
            is Success -> response.body
            is HttpError -> throw HttpException(response.status)
        }
```

### 3. for

*for* iterates through anything that provides an iterator, i.e.

- has a member- or extension-function `iterator()`, whose return type
  - has a member- or extension-function `next()`, and
  - has a member- or extension-function `hasNext()` that returns `Boolean`.

```kotlin
for (item: Int in ints) {
    // ...
}
```

To iterate over a range of numbers, use a [range expression](https://kotlinlang.org/docs/reference/ranges.html):

```kotlin
for (i in 1..3) {
    println(i)
}
for (i in 6 downTo 0 step 2) {
    println(i)
}
```

If you want to iterate through an array or a list with an index, you can do it this way:

```kotlin
for (i in array.indices) {
    println(array[i])
}

// Alternatively,
for ((index, value) in array.withIndex()) {
    println("the element at $index is $value")
}
```

### 4. while

```kotlin
while (x > 0) {
    x--
}

do {
    val y = retrieveData()
} while (y != null) // y is visible here!
```

### 5. returns and jumps 

Kotlin has three structural jump expressions:

- *return*. By default returns from the nearest enclosing function or [anonymous function](https://kotlinlang.org/docs/reference/lambdas.html#anonymous-functions).
- *break*. Terminates the nearest enclosing loop.
- *continue*. Proceeds to the next step of the nearest enclosing loop.

All of these expressions can be used as part of larger expressions:

```kotlin
val s = person.name ?: return
```

The type of these expressions is the [Nothing type](https://kotlinlang.org/docs/reference/exceptions.html#the-nothing-type).

(1) Break and Continue Labels

Any expression in Kotlin may be marked with a *label*. Labels have the form of an identifier followed by the `@` sign. 

Now, we can qualify a *break* or a *continue* with a label:

```kotlin
loop@ for (i in 1..100) {
    for (j in 1..100) {
        if (...) break@loop
    }
}
```

(2) Return at Labels

Qualified *return*s allow us to return from an outer function. The most important use case is returning from a lambda expression.

```kotlin
fun foo() {
    listOf(1, 2, 3, 4, 5).forEach lit@{
        if (it == 3) return@lit // local return to the caller of the lambda, i.e. the forEach loop
        print(it)
    }
    print(" done with explicit label")
}
```

Now, it returns only from the lambda expression. Oftentimes it is more convenient to use implicit labels: such a label has the same name as the function to which the lambda is passed.

```kotlin
fun foo() {
    listOf(1, 2, 3, 4, 5).forEach {
        if (it == 3) return@forEach // local return to the caller of the lambda, i.e. the forEach loop
        print(it)
    }
    print(" done with implicit label")
}
```

When returning a value, the parser gives preference to the qualified return, i.e.

```kotlin
return@a 1
```

means "return `1` at label `@a`".

# Classes

## Classes

Classes in Kotlin are declared using the keyword *class*. The class declaration consists of the class name, the class header (specifying its type parameters, the primary constructor etc.) and the class body, surrounded by curly braces. Both the header and the body are optional; if the class has no body, curly braces can be omitted.

```kotlin
class Empty
```

### 1. Constructors

A class in Kotlin can have a **primary constructor** and one or more **secondary constructors**. 

(1) primary constructors 

The primary constructor is part of the class header: it goes after the class name (and optional type parameters).

```kotlin
class Person constructor(firstName: String) { ... }
```

If the primary constructor does not have any annotations or visibility modifiers, the `constructor` keyword can be omitted.

Initialization code can be placed in **initializer blocks**, which are prefixed with the `init` keyword. During an instance initialization, the initializer blocks are executed in the same order as they appear in the class body, interleaved with the property initializers.

```kotlin
// Note that parameters of the primary constructor can be used in the initializer blocks.
class InitOrderDemo(name: String) {
    val firstProperty = "First property: $name".also(::println)
    
    init {
        println("First initializer block that prints ${name}")
    }
    
    val secondProperty = "Second property: ${name.length}".also(::println)
    
    init {
        println("Second initializer block that prints ${name.length}")
    }
}

// First property: hello
// First initializer block that prints hello
// Second property: 5
// Second initializer block that prints 5
```

Much the same way as regular properties, the **properties** declared in the primary constructor can be mutable (`var`) or read-only (`val`).

(2) Secondary Constructors

The class can also declare **secondary constructors**, which are prefixed with `constructor`.

```kotlin
class Person {
    constructor(parent: Person) {
        parent.children.add(this)
    }
}
```

If the class has a primary constructor, each secondary constructor needs to delegate to the primary constructor, either directly or indirectly through another secondary constructor(s). Delegation to another constructor of the same class is done using the *this* keyword.

```kotlin
class Person(val name: String) {
    constructor(name: String, parent: Person) : this(name) {
        parent.children.add(this)
    }
}
```

Note that initializer blocks are still executed before the secondary constructor body.

(3) default constructors

If a non-abstract class does not declare any constructors (primary or secondary), it will have a generated primary constructor with no arguments. The visibility of the constructor will be public. If you do not want your class to have a public constructor, you need to declare an empty primary constructor with non-default visibility.

```kotlin
class DontCreateMe private constructor () { 
    //... 
}
```

### 2. Creating instances of classes

Note that Kotlin does not have a `new` keyword.

## Inheritance

All classes in Kotlin have a common superclass `Any`, that is the default superclass for a class with no supertypes declared. `Any` is not `java.lang.Object`; in particular, it does not have any members other than `equals()`, `hashCode()` and `toString()`.

To declare an explicit supertype, we place the type after a colon in the class header. If the derived class has a primary constructor, the base class can (and must) be initialized right there, using the parameters of the primary constructor.

```kotlin
class Derived(p: Int) : Base(p)
```

If the class has no primary constructor, then each secondary constructor has to initialize the base type using the `super` keyword, or to delegate to another constructor which does that.

### 1. Overriding Methods

Kotlin requires explicit modifiers for overridable members (we call them `open`) and for overrides.

```kotlin
open class Base {
    open fun v() { ... }
    fun nv() { ... }
}
class Derived() : Base() {
    override fun v() { ... }
}
```

If there is no *open* modifier on a function, like `Base.nv()`, declaring a method with the same signature in a subclass is illegal.

A member marked `override` is itself open, i.e. it may be overridden in subclasses. If you want to prohibit re-overriding, use `final.`

```kotlin
open class AnotherDerived() : Base() {
    final override fun v() { ... }
}
```

### 2. Overriding Properties

Each declared property can be overridden by a property with an initializer or by a property with a getter method.

```kotlin
open class Foo {
    open val x: Int get() { ... }
}

class Bar1 : Foo() {
    override val x: Int = ...
}
```

You can also override a `val` property with a `var` property, but not vice versa.

Note that you can use the `override` keyword as part of the property declaration in a primary constructor.

### 3. Derived class initialization order

During construction of a new instance of a derived class, the base class initialization is done as the first step (preceded only by evaluation of the arguments for the base class constructor) and thus happens before the initialization logic of the derived class is run.

### 4. Calling the superclass implementation

Code in a derived class can call its superclass functions and property accessors implementations using the `super` keyword.

```kotlin
open class Foo {
    open fun f() { println("Foo.f()") }
    open val x: Int get() = 1
}

class Bar : Foo() {
    override fun f() { 
        super.f()
        println("Bar.f()") 
    }
    
    override val x: Int get() = super.x + 1
}
```

Inside an inner class, accessing the superclass of the outer class is done with the *super* keyword qualified with the outer class name: `super@Outer`.

### 5. Overriding Rules

 ```kotlin
open class A {
    open fun f() { print("A") }
    fun a() { print("a") }
}

interface B {
    fun f() { print("B") } // interface members are 'open' by default
    fun b() { print("b") }
}

class C() : A(), B {
    // The compiler requires f() to be overridden:
    override fun f() {
        super<A>.f() // call to A.f()
        super<B>.f() // call to B.f()
    }
}
 ```

It's fine to inherit from both `A` and `B`, and we have no problems with `a()` and `b()` since `C` inherits only one implementation of each of these functions. But for `f()` we have two implementations inherited by `C`, and thus we have to override `f()` in `C` and provide our own implementation that eliminates the ambiguity.

## Properties  and Fields 

### 1. Declaring Properties 

```kotlin
class Address {
    var name: String = ...
    var street: String = ...
    var city: String = ...
    var state: String? = ...
    var zip: String = ...
}
```

### 2. Getters and Setters 

(1) default

The full syntax for declaring a property is

```kotlin
var <propertyName>[: <PropertyType>] [= <property_initializer>]
    [<getter>]
    [<setter>]
```

The initializer, getter and setter are optional. Property type is optional if it can be inferred from the initializer (or from the getter return type, as shown below).

The full syntax of a read-only property declaration differs from a mutable one in two ways: it starts with `val` instead of `var` and does not allow a setter:

```kotlin
val simple: Int? // has type Int, default getter, must be initialized in constructor
val inferredType = 1 // has type Int and a default getter
```

(2) custom

If we define a custom getter, it will be called every time we access the property (this allows us to implement a computed property).

```kotlin
val isEmpty: Boolean
    get() = this.size == 0
```

If we define a custom setter, it will be called every time we assign a value to the property.

```kotlin
var stringRepresentation: String
    get() = this.toString()
    set(value) {
        setDataFromString(value) // parses the string and assigns values to other properties
    }
```

If you need to change the visibility of an accessor or to annotate it, but don't need to change the default implementation, you can define the accessor without defining its body:

```kotlin
var setterVisibility: String = "abc"
    private set // the setter is private and has the default implementation

var setterWithAnnotation: Any? = null
    @Inject set // annotate the setter with Inject
```

## Interfaces 

An interface is defined using the keyword `interface`. It can have properties but these need to be abstract or to provide accessor implementations.

```kotlin
interface MyInterface {
    fun bar()
    fun foo() {
      // optional body
    }
}
```

### 1. Implementing Interfaces

A class or object can implement one or more interfaces.

### 2. Properties in Interfaces

You can declare properties in interfaces. A property declared in an interface can either be abstract, or it can provide implementations for accessors.

```kotlin
interface MyInterface {
    val prop: Int // abstract

    val propertyWithImplementation: String
        get() = "foo"

    fun foo() {
        print(prop)
    }
}

class Child : MyInterface {
    override val prop: Int = 29
}
```

## Visibility Modifiers

Classes, objects, interfaces, constructors, functions, properties and their setters can have *visibility modifiers*. (Getters always have the same visibility as the property.) The default visibility, used if there is no explicit modifier, is `public`.

### 1. Packages

Functions, properties and classes, objects and interfaces can be declared on the "top-level", i.e. directly inside a package.

- If you do not specify any visibility modifier, `public` is used by default, which means that your declarations will be visible everywhere;
- If you mark a declaration `private`, it will only be visible inside the file containing the declaration;
- If you mark it `internal`, it is visible everywhere in the same [module](https://kotlinlang.org/docs/reference/visibility-modifiers.html#modules);

### 2. Classes and Interfaces

For members declared inside a class:

- `private` means visible inside this class only (including all its members);
- `protected` — same as `private` + visible in subclasses too. If you override a `protected` member and do not specify the visibility explicitly, the overriding member will also have `protected` visibility;
- `internal` — any client *inside this module* who sees the declaring class sees its `internal` members;
- `public` — any client who sees the declaring class sees its `public` members.

### 3. Constructors

To specify a visibility of the primary constructor of a class, use the following syntax (note that you need to add an explicit constructor` keyword).

```kotlin
class C private constructor(a: Int) { ... }
```

### 4. Modules 

The `internal` visibility modifier means that the member is visible within the same module. More specifically, a module is a set of Kotlin files compiled together:

- an IntelliJ IDEA module;
- a Maven project;
- a Gradle source set (with the exception that the `test` source set can access the internal declarations of `main`);
- a set of files compiled with one invocation of the `<kotlinc>` Ant task.

## Extensions 

### 1. Extension Functions

To declare an extension function, we need to prefix its name with a *receiver type*, i.e. the type being extended. 

```kotlin
fun MutableList<Int>.swap(index1: Int, index2: Int) {
    val tmp = this[index1] // 'this' corresponds to the list
    this[index1] = this[index2]
    this[index2] = tmp
}

val l = mutableListOf(1, 2, 3)
l.swap(0, 2) // 'this' inside 'swap()' will hold the value of 'l'
```

### 2. Extension Properties

```kotlin
val <T> List<T>.lastIndex: Int
    get() = size - 1
```

**Initializers are not allowed for extension properties**. Their behavior can only be defined by explicitly providing getters/setters.

### 3. Extensions are resolved **statically**

Extensions do not actually modify classes they extend. We would like to emphasize that extension functions are dispatched **statically**, i.e. they are not virtual by receiver type. This means that the extension function being called is determined by the type of the expression on which the function is invoked, not by the type of the result of evaluating that expression at runtime. 

```kotlin
open class C

class D: C()

fun C.foo() = "c"

fun D.foo() = "d"

fun printFoo(c: C) {
    println(c.foo())
}

printFoo(D()) // "c"
```

If a class has a member function, and an extension function is defined which has the same receiver type, the same name is applicable to given arguments, the **member always wins**.

```kotlin
class C {
    fun foo() { println("member") }
}

fun C.foo() { println("extension") } // "member"
```

### 4. Nullable Receiver

Note that extensions can be defined with a nullable receiver type. Such extensions can be called on an object variable even if its value is null, and can check for `this == null` inside the body. This is what allows you to call `toString()` in Kotlin without checking for null: the check happens inside the extension function.

### 5. Note on visibility

- An extension declared on top level of a file has access to the other `private` top-level declarations in the same file;
- If an extension is declared outside its receiver type, such an extension cannot access the receiver's `private` members.

## Data Classes 

```kotlin
data class User(val name: String, val age: Int)
```

The compiler automatically derives the following members from all properties declared in the primary constructor:

- `equals()`/`hashCode()` pair;
- `toString()` of the form `"User(name=John, age=42)"`;
- [`componentN()` functions](https://kotlinlang.org/docs/reference/multi-declarations.html) corresponding to the properties in their order of declaration;
- `copy()` function (see below).

To ensure consistency and meaningful behavior of the generated code, data classes have to fulfill the following requirements:

- The primary constructor needs to have at least one parameter;
- All primary constructor parameters need to be marked as `val`or `var`;
- Data classes cannot be abstract, open, sealed or inner;
- (before 1.1) Data classes may only implement interfaces.

On the JVM, if the generated class needs to have a parameterless constructor, default values for all properties have to be specified.

```kotlin
data class User(val name: String = "", val age: Int = 0)
```

### 1. Properties Declared in the Class Body

Note that the compiler only uses the properties defined inside the primary constructor for the automatically generated functions. To exclude a property from the generated implementations, declare it inside the class body.

```kotlin
data class Person(val name: String) {
    var age: Int = 0
}
```

Only the property `name` will be used inside the `toString()`, `equals()`, `hashCode()`, and `copy()` implementations, and there will only be one component function `component1()`. So two persons are equal as long as they have the same name.

### 2. Copying

It's often the case that we need to copy an object altering *some* of its properties, but keeping the rest unchanged. This is what `copy()`function is generated for.

```kotlin
data class User(val name: String = "", val age: Int = 0)

fun copy(name: String = this.name, age: Int = this.age) = User(name, age)

val jack = User(name = "Jack", age = 1)
val olderJack = jack.copy(age = 2)
```

## Sealed Classes

They are, in a sense, an extension of enum classes: the set of values for an enum type is also restricted, but each enum constant exists only as a single instance, whereas a subclass of a sealed class can have multiple instances which can contain state.

To declare a sealed class, you put the `sealed` modifier before the name of the class. A sealed class can have subclasses, but all of them must be declared in the same file as the sealed class itself.

```kotlin
sealed class Expr
data class Const(val number: Double) : Expr()
data class Sum(val e1: Expr, val e2: Expr) : Expr()
object NotANumber : Expr()
```

A sealed class is [abstract](https://kotlinlang.org/docs/reference/classes.html#abstract-classes) by itself, it cannot be instantiated directly and can have *abstract* members.

The key benefit of using sealed classes comes into play when you use them in a [`when` expression](https://kotlinlang.org/docs/reference/control-flow.html#when-expression). If it's possible to verify that the statement covers all cases, you don't need to add an `else` clause to the statement. However, this works only if you use `when` as an expression (using the result) and not as a statement.

```kotlin
fun eval(expr: Expr): Double = when(expr) {
    is Const -> expr.number
    is Sum -> eval(expr.e1) + eval(expr.e2)
    NotANumber -> Double.NaN
    // the `else` clause is not required because we've covered all the cases
}
```

## Generics 

### 1. Generic Classes

As in Java, classes in Kotlin may have type parameters:

```kotlin
class Box<T>(t: T) {
    var value = t
}
```

In general, to create an instance of such a class, we need to provide the type arguments:

```kotlin
val box: Box<Int> = Box<Int>(1)
```

But if the parameters may be inferred, e.g. from the constructor arguments or by some other means, one is allowed to omit the type arguments:

```kotlin
val box = Box(1) // 1 has type Int, so the compiler figures out that we are talking about Box<Int>
```

### 2. Generic Functions 

Type parameters are placed **before** the name of the function:

```kotlin
fun <T> singletonList(item: T): List<T> {
    // ...
}

fun <T> T.basicToString() : String {  // extension function
    // ...
}
```

To call a generic function, specify the type arguments at the call site **after** the name of the function:

```kotlin
val l = singletonList<Int>(1)
```

Type arguments can be omitted if they can be inferred from the context.

### 3. Generic constraints

```kotlin
fun <T : Comparable<T>> sort(list: List<T>) {  ... }
```

The type specified after a colon is the **upper bound**: only a subtype of `Comparable<T>` may be substituted for `T`.

If the same type parameter needs more than one upper bound, we need a separate **where**-clause:

```kotlin
fun <T> copyWhenGreater(list: List<T>, threshold: T): List<String>
    where T : CharSequence,
          T : Comparable<T> {
    return list.filter { it > threshold }.map { it.toString() }
}
```

## Enum Classes

```kotlin
enum class Direction {
    NORTH, SOUTH, WEST, EAST
}
```

Each enum constant is an object. Enum constants are separated with commas.

### 1. Initialization 

Since each enum is an instance of the enum class, they can be initialized as:

```kotlin
enum class Color(val rgb: Int) {
        RED(0xFF0000),
        GREEN(0x00FF00),
        BLUE(0x0000FF)
}
```

### 2. Anonymous Classes

```kotlin 
enum class ProtocolState {
    WAITING {
        override fun signal() = TALKING
    },

    TALKING {
        override fun signal() = WAITING
    }; // semicolon sseparates the enum constant definitions from the member definitions

    abstract fun signal(): ProtocolState
}
```

### 3. Implementing Interfaces in Enum Classes

An enum class may implement an interface (but not derive from a class), providing either a single interface members implementation for all of the entries, or separate ones for each entry within its anonymous class.

```kotlin
enum class IntArithmetics : BinaryOperator<Int>, IntBinaryOperator {
    PLUS {
        override fun apply(t: Int, u: Int): Int = t + u
    },
    TIMES {
        override fun apply(t: Int, u: Int): Int = t * u
    };

    override fun applyAsInt(t: Int, u: Int) = apply(t, u)
}
```

### 4.  Working with Enum Constants

```kotlin
enum class RGB { RED, GREEN, BLUE }

val r = RGB.RED  // This is a shorthand for RGB.valueOf("RED")

RGB.values().forEach { print(it.toString() + ", ") } // RED GREEN BLUE
```

Every enum constant has properties to obtain its name and position in the enum class declaration:

```kotlin
val name: String
val ordinal: Int
```

The enum constants also implement the [Comparable](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-comparable/index.html) interface, with the natural order being the order in which they are defined in the enum class.

## Objects

### 1. Object declarations

[Singleton](http://en.wikipedia.org/wiki/Singleton_pattern) may be useful in several cases, and Kotlin (after Scala) makes it easy to declare singletons.

```kotlin
object DataProviderManager {
    fun registerDataProvider(provider: DataProvider) {
        // ...
    }

    val allDataProviders: Collection<DataProvider>
        get() = // ...
}
```

Object declaration's initialization is thread-safe.

### 2. Companion Objects

```kotlin
class MyClass {
    companion object Factory {
        fun create(): MyClass = MyClass()
    }
}
```

Members of the companion object can be called by using simply the class name as the qualifier:

```kotlin
val instance = MyClass.create()
```

The name of the companion object can be omitted, in which case the name `Companion` will be used.

Note that, even though the members of companion objects look like static members in other languages, at runtime those are still instance members of real objects, and can, for example, implement interfaces.

# Functions and Lambdas 

## Functions 

### 1. Function Declarations

Functions in Kotlin are declared using the `fun` keyword.

```kotlin
fun double(x: Int): Int {
    return 2 * x
}
```

### 2. Parameters

Function parameters are defined using Pascal notation, i.e. *name*: *type*. Parameters are separated using commas. Each parameter must be explicitly typed.

(1) Default Arguments

Default values are defined using the **=** after type along with the value.

```kotlin
fun read(b: Array<Byte>, off: Int = 0, len: Int = b.size) { ... }
```

When overriding a method with default parameters values, the default parameter values must be omitted from the signature:

```kotlin
open class A {
    open fun foo(i: Int = 10) { ... }
}

class B : A() {
    override fun foo(i: Int) { ... }  // no default value allowed
}
```

If a default parameter precedes a parameter with no default value, the default value can be used only by calling the function with [named arguments](https://kotlinlang.org/docs/reference/functions.html#named-arguments):

```kotlin
fun foo(bar: Int = 0, baz: Int) { ... }

foo(baz = 1) // The default value bar = 0 is used
```

But if a last argument [lambda](https://kotlinlang.org/docs/reference/lambdas.html#lambda-expression-syntax) is passed to a function call outside the parentheses, passing no values for the default parameters is allowed:

```kotlin
fun foo(bar: Int = 0, baz: Int = 1, qux: () -> Unit) { ... }

foo(1) { println("hello") } // Uses the default value baz = 1 
foo { println("hello") }    // Uses both default values bar = 0 and baz = 1
```

### 3. Named Arguments

When a function is called with both positional and named arguments, all the positional arguments should be placed before the first named one. For example, the call `f(1, y = 2)` is allowed, but `f(x = 1, 2)` is not.

[Variable number of arguments (*vararg*)](https://kotlinlang.org/docs/reference/functions.html#variable-number-of-arguments-varargs) can be passed in the named form by using the **spread** operator:

```kotlin
fun foo(vararg strings: String) { ... }

foo(strings = *arrayOf("a", "b", "c"))
```

### 4. Unit-returning functions

If a function does not return any useful value, its return type is `Unit`. `Unit` is a type with only one value - `Unit`. This value does not have to be returned explicitly. The `Unit` return type declaration is also optional.

### 5. Single-Expression functions 

When a function returns a single expression, the curly braces can be omitted and the body is specified after a **=** symbol.

```kotlin
fun double(x: Int): Int = x * 2
```

Explicitly declaring the return type is [optional](https://kotlinlang.org/docs/reference/functions.html#explicit-return-types) when this can be inferred by the compiler.

### 6. Explicit return types

Functions with block body must always specify return types explicitly, unless it's intended for them to return `Unit`.

### 7. Variable number of arguments (Varargs)

A parameter of a function (normally the last one) may be marked with `vararg` modifier allowing a variable number of arguments to be passed to the function. Inside a function a `vararg`-parameter of type `T` is visible as an array of `T`, i.e. the `ts` variable in the example above has type `Array<out T>`.

When we call a `vararg`-function, we can pass arguments one-by-one, or, if we already have an array and want to pass its contents to the function, we use the **spread** operator (prefix the array with `*`).

```kotlin
val a = arrayOf(1, 2, 3)
val list = asList(-1, 0, *a, 4)
```

Only one parameter may be marked as `vararg`. If a `vararg`parameter is not the last one in the list, values for the following parameters can be passed using the named argument syntax, or, if the parameter has a function type, by passing a lambda outside parentheses.

### 8. Infix notation

Functions marked with the `infix` keyword can also be called using the infix notation (omitting the dot and the parentheses for the call). Infix functions must satisfy the following requirements:

- They must be member functions or [extension functions](https://kotlinlang.org/docs/reference/extensions.html);
- They must have a single parameter;
- The parameter must not [accept variable number of arguments](https://kotlinlang.org/docs/reference/functions.html#variable-number-of-arguments-varargs)and must have no [default value](https://kotlinlang.org/docs/reference/functions.html#default-arguments).

Note that infix function calls have lower precedence than the arithmetic operators, type casts, and the `rangeTo`operator. On the other hand, infix function call's precedence is higher than that of the boolean operators `&&` and `||`, `is`- and `in`-checks, and some other operators. 

### 9. Function Scope 

In Kotlin functions can be declared at top level in a file, meaning you do not need to create a class to hold a function. Kotlin functions can also be declared local, as member functions and extension functions.

(1) Local Functions 

Kotlin supports local functions, i.e. a function inside another function. Local function can access local variables of outer functions (i.e. the closure).

(2) Member Functions

A member function is a function that is defined inside a class or object.

### 10. Tail recursive functions

When a function is marked with the `tailrec` modifier and meets the required form, the compiler optimises out the recursion, leaving behind a fast and efficient loop based version instead:

```kotlin
val eps = 1E-10 // "good enough", could be 10^-15

tailrec fun findFixPoint(x: Double = 1.0): Double
        = if (Math.abs(x - Math.cos(x)) < eps) x else findFixPoint(Math.cos(x))
```

You cannot use tail recursion when there is more code after the recursive call, and you cannot use it within try/catch/finally blocks. Currently tail recursion is only supported in the JVM backend.

## Higher-Order Functions and Lambdas

### 1. Higher-Order Functions

A higher-order function is a function that takes functions as parameters, or returns a function.

### 2. Function types

- All function types have a parenthesized parameter types list and a return type: `(A, B) -> C` denotes a type that represents functions taking two arguments of types `A` and `B` and returning a value of type `C`. The parameter types list may be empty, as in `() -> A`. The [`Unit` return type](https://kotlinlang.org/docs/reference/functions.html#unit-returning-functions) cannot be omitted.
- Function types can optionally have an additional *receiver* type, which is specified before a dot in the notation: the type `A.(B) -> C` represents functions that can be called on a receiver object of `A` with a parameter of `B` and return a value of `C`. [Function literals with receiver](https://kotlinlang.org/docs/reference/lambdas.html#function-literals-with-receiver) are often used along with these types.
- [Suspending functions](https://kotlinlang.org/docs/reference/coroutines.html#suspending-functions) belong to function types of a special kind, which have a *suspend* modifier in the notation, such as `suspend () -> Unit` or `suspend A.(B) -> C`

### 3. Lambda Expressions and Anonymous Functions

Lambda expressions and anonymous functions are 'function literals', i.e. functions that are not declared, but passed immediately as an expression.

(1) Lambda expression syntax

```kotlin
val sum = { x: Int, y: Int -> x + y }
```

A lambda expression is always surrounded by curly braces, parameter declarations in the full syntactic form go inside curly braces and have optional type annotations, the body goes after an `->` sign. If the inferred return type of the lambda is not `Unit`, the last (or possibly single) expression inside the lambda body is treated as the return value.

(2) Passing a lambda to the last parameter

A lambda expression that is passed as the corresponding argument can be placed outside the parentheses:

```kotlin
val product = items.fold(1) { acc, e -> acc * e }
```

If the lambda is the only argument to that call, the parentheses can be omitted entirely.

(3) `it`: implicit name of a single parameter

It's very common that a lambda expression has only one parameter.

If the compiler can figure the signature out itself, it is allowed not to declare the only parameter and omit `->`. The parameter will be implicitly declared under the name `it`:

```kotlin
ints.filter { it > 0 } // this literal is of type '(it: Int) -> Boolean'
```

(4) Returning a value from a lambda expression

We can explicitly return a value from the lambda using the [qualified return](https://kotlinlang.org/docs/reference/returns.html#return-at-labels) syntax. Otherwise, the value of the last expression is implicitly returned.

(5) Underscore for unused variables (since 1.1)

If the lambda parameter is unused, you can place an underscore instead of its name:

```kotlin
map.forEach { _, value -> println("$value!") }
```

(6) Anonymous functions

An anonymous function looks very much like a regular function declaration, except that its name is omitted. The parameters and the return type are specified in the same way as for regular functions, except that the parameter types can be omitted if they can be inferred from context:

```kotlin
ints.filter(fun(item) = item > 0)
```

Note that anonymous function parameters are always passed inside the parentheses. The shorthand syntax allowing to leave the function outside the parentheses works only for lambda expressions.

A *return* statement without a label always returns from the function declared with the`fun`keyword. This means that a`return`inside a lambda expression will return from the enclosing function, whereas a`return`inside an anonymous function will return from the anonymous function itself.

(7) Closures

A lambda expression or anonymous function can access its *closure*, i.e. the variables declared in the outer scope.

```kotlin
var sum = 0
ints.filter { it > 0 }.forEach {
    sum += it
}
print(sum)
```

## Inline Functions 

Using [higher-order functions](https://kotlinlang.org/docs/reference/lambdas.html) imposes certain runtime penalties: each function is an object, and it captures a closure, i.e. those variables that are accessed in the body of the function. Memory allocations (both for function objects and classes) and virtual calls introduce runtime overhead.

```kotlin
lock(l) { foo() }
```

Instead of creating a function object for the parameter and generating a call, the compiler could emit the following code:

```kotlin
l.lock()
try {
    foo()
}
finally {
    l.unlock()
}
```

To make the compiler do this, we need to mark the `lock()` function with the `inline` modifier:

```kotlin
inline fun <T> lock(lock: Lock, body: () -> T): T { ... }
```

The `inline` modifier affects both the function itself and the lambdas passed to it: all of those will be inlined into the call site.

### noinline

In case you want only some of the lambdas passed to an inline function to be inlined, you can mark some of your function parameters with the `noinline` modifier.

Inlinable lambdas can only be called inside the inline functions or passed as inlinable arguments, but `noinline` ones can be manipulated in any way we like: stored in fields, passed around etc.

### Non-local returns

If the function the lambda is passed to is inlined, the return can be inlined as well, so it is allowed:

```kotlin
fun foo() {
    inlined {
        return // OK: the lambda is inlined
    }
}
```

Such returns (located in a lambda, but exiting the enclosing function) are called *non-local* returns. We are used to this sort of construct in loops, which inline functions often enclose:

```kotlin
fun hasZeros(ints: List<Int>): Boolean {
    ints.forEach {
        if (it == 0) return true // returns from hasZeros
    }
    return false
}
```

`break` and `continue` are not yet available in inlined lambdas.

### Reified type parameters

Sometimes we need to access a type passed to us as a parameter. To enable this, inline functions support *reified type parameters*, so we can write something like this:

```kotlin
inline fun <reified T> TreeNode.findParentOfType(): T? {
    var p = parent
    while (p != null && p !is T) {
        p = p.parent
    }
    return p as T?
}
```

Since the function is inlined, no reflection is needed, normal operators like `!is` and `as` are working now. 

Note that normal functions (not marked as inline) cannot have reified parameters. 

### Inline properties

The `inline` modifier can be used on accessors of properties that don't have a backing field.

```kotlin
val foo: Foo
    inline get() = Foo()

var bar: Bar
    get() = ...
    inline set(v) { ... }
```

 You can annotate individual property accessors. You can also annotate an entire property. 

```kotlin
inline var bar: Bar
    get() = ...
    set(v) { ... }
```

At the call site, inline accessors are inlined as regular inline functions.

