# Formatting 

Go takes care of most formatting issues. 

The `gofmt` program (also available as `go fmt`, which operates at the package level rather than source file level) reads a Go program and emits the source in a standard style of indentation and vertical alignment, retaining and if necessary reformatting comments. For example, we have a unformatting  source file:

```go
type T struct {
    name string // name of the object
    value int // its value
}
```

Run `gofmt`command in the command line or terminal then we can see a formatted input:

```go
type T struct {
    name    string // name of the object
    value   int    // its value
}
```

**Some brief formatting rules:**

(1) Indentation

We use tabs for indentation and `gofmt` emits them by default. Use spaces only if you must.

(2) Line length

Go has no line length limit. Don't worry about overflowing a punched card. If a line feels too long, wrap it and indent with an extra tab.

(3) Parentheses

Go needs fewer parentheses than C and Java: control structures do not have parentheses in their syntax. Also, the operator precedence hierarchy is shorter and clearer, so

```
x<<8 + y<<16
```

means what the spacing implies, unlike in the other languages.

# Commentary

Go provides C-style `/* */` block comments and C++-style `//` line comments. Line comments are the norm; block comments appear mostly as package comments, but are useful within an expression or to disable large swaths of code.

The program—and web server—`godoc` processes Go source files to extract documentation about the contents of the package. The nature and style of these comments determines the quality of the documentation `godoc` produces.

1.package comments

(1) Every package should have a *package comment*, a block comment preceding the package clause. For multi-file packages, the package comment only needs to be present in one file, and any one will do. The package comment should introduce the package and provide information relevant to the package as a whole. If the package is simple, the package comment can be brief.

(2) Depending on the context, `godoc` might not even reformat comments, so make sure they look good straight up: use correct spelling, punctuation, and sentence structure, fold long lines, and so on.

(3) Inside a package, any comment immediately preceding a top-level declaration serves as a *doc comment* for that declaration. Every exported (capitalized) name in a program should have a doc comment.

(4) If every doc comment begins with the name of the item it describes, the output of `godoc` can usefully be run through `grep`.

```bash
$ godoc regexp | grep parse
    Compile parses a regular expression and returns, if successful, a Regexp
    parsed. It simplifies safe initialization of global variables holding
    cannot be parsed. It simplifies safe initialization of global variables
$
```

2.declaration comments

 A single doc comment can introduce a group of related constants or variables. Since the whole declaration is presented, such a comment can often be perfunctory.

```go
// Error codes returned by failures to parse an expression.
var (
    ErrInternal      = errors.New("regexp: internal error")
    ErrUnmatchedLpar = errors.New("regexp: unmatched '('")
    ErrUnmatchedRpar = errors.New("regexp: unmatched ')'")
    ...
)
```

Grouping can also indicate relationships between items, such as the fact that a set of variables is protected by a mutex.

```go
var (
    countLock   sync.Mutex
    inputCount  uint32
    outputCount uint32
    errorCount  uint32
)
```

# Names

1.package names

(1)  When a package is imported, the package name becomes an accessor for the contents. After

```go
import "bytes"
```

the importing package can talk about `bytes.Buffer`.

(2) It's helpful if everyone using the package can use the same name to refer to its contents, which implies that the package name should be good: short, concise, evocative. By convention, packages are given lower case, single-word names; there should be no need for underscores or mixedCaps.

(3) Use the package structure to help you choose good names. For instance, the function to make new instances of `ring.Ring`—which is the definition of a *constructor* in Go—would normally be called `NewRing`, but since `Ring` is the only type exported by the package, and since the package is called `ring`, it's called just `New`, which clients of the package see as `ring.New`.

2.getters

(1) It's neither idiomatic nor necessary to put `Get` into the getter's name. If you have a field called `owner` (lower case, unexported), the getter method should be called `Owner`(upper case, exported), not `GetOwner`. 

(2) A setter function, if needed, will likely be called `SetOwner`. Both names read well in practice:

```go
owner := obj.Owner()
if owner != user {
    obj.SetOwner(user)
}
```

3.interface names

By convention, one-method interface are named by the method name plus an -er suffix or similar modification to construct an agent noun: `Reader`, `Writer`, `Formatter`, `CloseNotifier` etc.

There are a number of such names and it's productive to honor them and the function names they capture. `Read`, `Write`, `Close`, `Flush`, `String` and so on have canonical signatures and meanings. To avoid confusion, don't give your method one of those names unless it has the same signature and meaning. Conversely, if your type implements a method with the same meaning as a method on a well-known type, give it the same name and signature; call your string-converter method `String`not `ToString`.

4.MixedCaps

The convention in Go is to use `MixedCaps` or `mixedCaps` rather than underscores to write multiword names.

# Semicolon

The lexer uses a simple rule to insert semicolons automatically as it scans, so the input text is mostly free of them.

(1) If the newline comes after a token that could end a statement, insert a semicolon. 

(2) Idiomatic Go programs have semicolons only in places such as `for` loop clauses, to separate the initializer, condition, and continuation elements. 

(3) One consequence of the semicolon insertion rules is that you cannot put the opening brace of a control structure (`if`, `for`, `switch`, or `select`) on the next line. If you do, a semicolon will be inserted before the brace, which could cause unwanted effects. 

# Control structures

1.if

When an `if` statement doesn't flow into the next statement—that is, the body ends in `break`, `continue`, `goto`, or `return`—the unnecessary `else` is omitted.

```go
f,err := os.Open(name)
if err != nil {
    return err
}
// use f
```

2.redeclaration and reassignment

In a `:=` declaration a variable `v` may appear even if it has already been declared, provided:

- this declaration is in the same scope as the existing declaration of `v` (if `v` is already declared in an outer scope, the declaration will create a new variable §),
- the corresponding value in the initialization is assignable to `v`, and
- there is at least one other variable in the declaration that is being declared anew.

§ It's worth noting here that in Go the scope of function parameters and return values is the same as the function body, even though they appear lexically outside the braces that enclose the body.

3.for

(1) 

```go
// Like a C for
for init; condition; post { }

// Like a C while
for condition { }

// Like a C for(;;)
for { }
```

(2) If you only need the first item in the range (the key or index), drop the second:

```go
for key := range m {
    if key.expired() {
        delete(m, key)
    }
}
```

If you only need the second item in the range (the value), use the *blank identifier*, an underscore, to discard the first:

```go
sum := 0
for _, value := range array {
    sum += value
}
```

(3) If you want to run multiple variables in a `for` you should use parallel assignment (although that precludes `++` and `--`).

```go
// Reverse a
for i, j := 0, len(a)-1; i < j; i, j = i+1, j-1 {
    a[i], a[j] = a[j], a[i]
}
```

4.switch

A switch can also be used to discover the dynamic type of an **interface** variable. 

```go
var t interface{}
t = functionOfSomeType()
switch t := t.(type) {
default:
    fmt.Printf("unexpected type %T\n", t)     // %T prints whatever type t has
case bool:
    fmt.Printf("boolean %t\n", t)             // t has type bool
case int:
    fmt.Printf("integer %d\n", t)             // t has type int
case *bool:
    fmt.Printf("pointer to boolean %t\n", *t) // t has type *bool
case *int:
    fmt.Printf("pointer to integer %d\n", *t) // t has type *int
}
```

# Functions

1.multiple return values

(1) error handling

```go
func (file *File) Write(b []byte) (n int, err error)
```

(2)  obviate the need to pass a pointer to a return value to simulate a reference parameter

```go
func nextInt(b []byte, i int) (int, int) {
    for ; i < len(b) && !isDigit(b[i]); i++ {
    }
    x := 0
    for ; i < len(b) && isDigit(b[i]); i++ {
        x = x*10 + int(b[i]) - '0'
    }
    return x, i
}
```

Here's a simple-minded function to grab a number from a position in a byte slice, returning the number and the next position.

2.named result parameters

When named, they are initialized to the zero values for their types when the function begins; if the function executes a `return` statement with no arguments, the current values of the result parameters are used as the returned values.

The names are not mandatory but they can make code shorter and clearer: they're documentation. If we name the results of `nextInt` it becomes obvious which returned `int` is which.

```go
func nextInt(b []byte, pos int) (value, nextPos int) {
```

3.defer

(1) The arguments to the deferred function (which include the receiver if the function is a method) are evaluated when the **defer** executes, not when the *call* executes. (It means that the arguments will be evaluated the instant the program runs to the defer statements.) Besides avoiding worries about variables changing values as the function executes, this means that a single deferred call site can defer multiple function executions.

(2) Deferred functions are executed in LIFO order.

```go
func trace(s string) string {
    fmt.Println("entering:", s)
    return s
}

func un(s string) {
    fmt.Println("leaving:", s)
}

func a() {
    defer un(trace("a"))
    fmt.Println("in a")
}

func b() {
    defer un(trace("b"))
    fmt.Println("in b")
    a()
}

func main() {
    b()
}


// prints
// entering: b
// in b
// entering: a
// in a
// leaving: a
// leaving: b
```

# Data

1.allocation with `new`

`new(T)` allocates zeroed storage for a new item of type `T` and returns its address, a value of type `*T`.

Since the memory returned by `new` is zeroed, it means a user of the data structure can create one with `new` and get right to work.

2.constructors and composite literals

taking the address of a composite literal allocates a fresh instance each time it is evaluated, so it's perfectly OK to return the address of a local variable.

```go
return &File{fd, name, nil, 0}
```

Composite literals can also be created for arrays, slices, and maps, with the field labels being indices or map keys as appropriate. In these examples, the initializations work regardless of the values of `Enone`, `Eio`, and `Einval`, as long as they are distinct.

```go
a := [...]string   {Enone: "no error", Eio: "Eio", Einval: "invalid argument"}
s := []string      {Enone: "no error", Eio: "Eio", Einval: "invalid argument"}
m := map[int]string{Enone: "no error", Eio: "Eio", Einval: "invalid argument"}
```

3.allocation with `make`

It creates **slices, maps, and channels only**, and it returns an *initialized*(not *zeroed*) value of type `T` (not `*T`).

4.arrays

In particular, if you pass an array to a function, it will receive a *copy* of the array, not a pointer to it. The value property can be useful but also expensive; if you want C-like behavior and efficiency, you can pass a pointer to the array.

But even this style isn't idiomatic Go. Use slices instead.

5.slices

If a function takes a slice argument, changes it makes to the elements of the slice will be visible to the caller, analogous to passing a pointer to the underlying array. 

6.2D slices

If the slices might grow or shrink, they should be allocated independently to avoid overwriting the next line; if not, it can be more efficient to construct the object with a single allocation.

```go
// Allocate the top-level slice.
picture := make([][]uint8, YSize) // One row per unit of y.
// Loop over the rows, allocating the slice for each row.
for i := range picture {
	picture[i] = make([]uint8, XSize)
}
```

And now as one allocation, sliced into lines:

```go
// Allocate the top-level slice, the same as before.
picture := make([][]uint8, YSize) // One row per unit of y.
// Allocate one large slice to hold all the pixels.
pixels := make([]uint8, XSize*YSize) // Has type []uint8 even though picture is [][]uint8.
// Loop over the rows, slicing each row from the front of the remaining pixels slice.
for i := range picture {
	picture[i], pixels = pixels[:XSize], pixels[XSize:]
}
```

7.maps

(1) The key can be of any type for which the equality operator is defined, such as integers, floating point and complex numbers, strings, pointers, interfaces (as long as the dynamic type supports equality), structs and arrays.

(2) If you pass a map to a function that changes the contents of the map, the changes will be visible in the caller.

(3) A set can be implemented as a map with value type `bool`. Set the map entry to `true` to put the value in the set, and then test it by simple indexing.

```go
attended := map[string]bool{
    "Ann": true,
    "Joe": true,
    ...
}

if attended[person] { // will be false if person is not in the map
    fmt.Println(person, "was at the meeting")
}
```

8.printing

If you want to control the default format for a custom type, all that's required is to define a method with the signature `String() string` on the type.

```go
func (t *T) String() string {
    return fmt.Sprintf("%d/%g/%q", t.a, t.b, t.c)
}
fmt.Printf("%v\n", t)
```

If you need to print **values** of type `T` as well as pointers to `T`, the receiver for `String` must be of value type.

9.append

Schematically, it's like this:

```go
func append(slice []T, elements ...T) []T
```

where *T* is a placeholder for any given type. You can't actually write a function in Go where the type `T` is determined by the caller. That's why `append` is built in: it needs support from the compiler.

But what if we wanted to do what our `Append` does and append a slice to a slice? Easy: use `...` at the call site.

```go
x := []int{1,2,3}
y := []int{4,5,6}
x = append(x, y...)
fmt.Println(x)
```

Without that `...`, it wouldn't compile because the types would be wrong; `y` is not of type `int`.

# Initialization

1.constants

(1) They are created at compile time, even when defined as locals in functions, and can only be numbers, characters (runes), strings or booleans. 

(2) The expressions that define them must be constant expressions, evaluatable by the compiler.

2.variables

Variables can be initialized just like constants but the initializer can be a general expression computed at run time.

3.`init`function

(1) Actually each file can have multiple `init` functions.

(2) `init` is called after all the variable declarations in the package have evaluated their initializers, and those are evaluated only after all the imported packages have been initialized.

(3) A common use of `init`functions is to verify or repair correctness of the program state before real execution begins.

```go
func init() {
    if user == "" {
        log.Fatal("$USER not set")
    }
    if home == "" {
        home = "/home/" + user
    }
    if gopath == "" {
        gopath = home + "/go"
    }
    // gopath may be overridden by --gopath flag on command line.
    flag.StringVar(&gopath, "gopath", gopath, "override default GOPATH")
}
```

# Methods

pointers vs. values

(1)  Methods can be defined for any named type (except a pointer or an interface); the receiver does not have to be a struct.

(2) The rule about pointers vs. values for receivers is that value methods can be invoked on pointers and values, but pointer methods can only be invoked on pointers.

This rule arises because pointer methods can modify the receiver; invoking them on a value would cause the method to receive a copy of the value, so any modifications would be discarded.

The language takes care of the common case of invoking a pointer method on a value by inserting the address operator automatically.

# Interfaces and other types

1.interfaces

(1) Interfaces in Go provide a way to specify the behavior of an object: if something can do *this*, then it can be used *here*.

(2) A type can implement multiple interfaces

2.conversions

(1) A conversion may not create a new value, it just temporarily acts as though the existing value has a new type. (There are other legal conversions, such as from integer to floating point, that do create a new value.)

(2) It's an idiom in Go programs to convert the type of an expression to access a different set of methods. That's more unusual in practice but can be effective.

```go
type Sequence []int

// Method for printing - sorts the elements before printing
func (s Sequence) String() string {
    sort.IntSlice(s).Sort()
    return fmt.Sprint([]int(s))
}
```

3.interfaces and methods

Interfaces are just sets of methods, which can be defined for (almost) any type.

#  The blank identifier

1.multiple assignment

If an assignment requires multiple values on the left side, but one of the values will not be used by the program, a blank identifier on the left-hand-side of the assignment avoids the need to create a dummy variable and makes it clear that the value is to be discarded.

2.unused imports and variables

To silence complaints about the unused imports, use a blank identifier to refer to a symbol from the imported package. Similarly, assigning the unused variables to the blank identifier will silence the unused variable error.

3.import for side effect

But sometimes it is useful to import a package only for its side effects, without any explicit use. For example, during its `init`function, the `net/http/pprof` package registers HTTP handlers that provide debugging information. It has an exported API, but most clients need only the handler registration and access the data through a web page. To import the package only for its side effects, rename the package to the blank identifier.

4.interface checks

If it's necessary only to ask whether a type implements an interface, without actually using the interface itself, perhaps as part of an error check, use the blank identifier to ignore the type-asserted value:

```go
if _, ok := val.(json.Marshaler); ok {
    fmt.Printf("value %v of type %T implements json.Marshaler\n", val, val)
}
```


One place this situation arises is when it is necessary to guarantee within the package implementing the type that it actually satisfies the interface.

```go
var _ json.Marshaler = (*RawMessage)(nil)
```

In this declaration, the assignment involving a conversion of a `*RawMessage` to a `Marshaler`requires that `*RawMessage` implements `Marshaler`, and that property will be checked at compile time. Should the `json.Marshaler` interface change, this package will no longer compile and we will be on notice that it needs to be updated.

Don't do this for every type that satisfies an interface, though. By convention, such declarations are only used when there are **no static conversions** already present in the code, which is a rare event.

# Embedding

(1) Only interfaces can be embedded within interfaces.

```go
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}

// ReadWriter is the interface that combines the Reader and Writer interfaces.
type ReadWriter interface {
    Reader
    Writer
}
```

(2) We can embed the structs directly without giving them filed names. By doing this, we avoid promoting the methods of the fields and satisfying some interfaces. 

When we embed a type, the methods of that type become methods of the outer type, but when they are invoked the receiver of the method is the inner type, not the outer one.

(3) Embedding can also be a simple convenience.

```go
type Job struct {
    Command string
    *log.Logger
}
```

The `Job` type now has the `Log`, `Logf` and other methods of `*log.Logger`. We could have given the `Logger` a field name, of course, but it's not necessary to do so. And now, once initialized, we can log to the `Job`:

```go
job.Log("starting now...")
```

If we need to refer to an embedded field directly, the type name of the field, ignoring the package qualifier, serves as a field name, as it did in the `Read` method of our `ReadWriter`struct. Here, if we needed to access the `*log.Logger` of a `Job` variable `job`, we would write `job.Logger`, which would be useful if we wanted to refine the methods of `Logger`.

```go
func (job *Job) Logf(format string, args ...interface{}) {
    job.Logger.Logf("%q: %s", job.Command, fmt.Sprintf(format, args...))
}
```

Embedding types introduces the problem of name conflicts but the rules to resolve them are simple. First, a field or method `X` hides any other item `X` in a more deeply nested part of the type. If `log.Logger` contained a field or method called `Command`, the `Command` field of `Job` would dominate it.

Second, if the same name appears at the same nesting level, it is usually an error; it would be erroneous to embed `log.Logger` if the `Job` struct contained another field or method called `Logger`. However, there is no problem if a field is added that conflicts with another field in another subtype if neither field is ever used.

# Concurrency

1.shared by communicating

> Do not communicate by sharing memory; instead, share memory by communicating.

Go encourages a different approach in which shared values are passed around on channels and, in fact, never actively shared by separate threads of execution. Only one goroutine has access to the value at any given time. Data races cannot occur, by design.

2.goroutine

They're called *goroutines* because the existing terms—threads, coroutines, processes, and so on—convey inaccurate connotations. A goroutine has a simple model: it is **a function executing concurrently with other goroutines in the same address space**. It is lightweight, costing little more than the allocation of stack space. And the stacks start small, so they are cheap, and grow by allocating (and freeing) heap storage as required.

3.channel

(1) A channel can allow the launching goroutine to wait for things to complete.

```go
c := make(chan int)  // Allocate a channel.
// Start the sort in a goroutine; when it completes, signal on the channel.
go func() {
    list.Sort()
    c <- 1  // Send a signal; value does not matter.
}()
doSomethingForAWhile()
<-c   // Wait for sort to finish; discard sent value.
```

(2) A buffered channel can be used like a semaphore, for instance to limit throughput.

```go
var sem = make(chan int, MaxOutstanding)

func Serve(queue chan *Request) {
    for req := range queue {
        sem <- 1
        go func(req *Request) {
            process(req)
            <-sem
        }(req)
    }
}
```

In a Go `for` loop, the loop variable is reused for each iteration, so the `req`variable is shared across all goroutines. That's not what we want. We need to make sure that `req` is unique for each goroutine so we pass  the value of `req` as an argument to the closure in the goroutine.

Another approach that manages resources well is to start a fixed number of `handle` goroutines all reading from the request channel. The number of goroutines limits the number of simultaneous calls to `process`. 

```go
func handle(queue chan *Request) {
    for r := range queue {
        process(r)
    }
}

func Serve(clientRequests chan *Request, quit chan bool) {
    // Start handlers
    for i := 0; i < MaxOutstanding; i++ {
        go handle(clientRequests)
    }
    <-quit  // Wait to be told to exit.
}
```

4.channels of channels

```go
type Request struct {
    args        []int
    f           func([]int) int
    resultChan  chan int
}
```

The client provides a function and its arguments, as well as a channel inside the request object on which to receive the answer.

```go
func sum(a []int) (s int) {
    for _, v := range a {
        s += v
    }
    return
}

request := &Request{[]int{3, 4, 5}, sum, make(chan int)}
// Send request
clientRequests <- request
// Wait for response.
fmt.Printf("answer: %d\n", <-request.resultChan)
```

On the server side, the handler function is the only thing that changes.

```go
func handle(queue chan *Request) {
    for req := range queue {
        req.resultChan <- req.f(req.args)
    }
}
```

There's clearly a lot more to do to make it realistic, but this code is a framework for a **rate-limited, parallel, non-blocking RPC system**, and there's not a mutex in sight.

5.parallelization

Let's say we have an expensive operation to perform on a vector of items, and that the value of the operation on each item is independent, as in this idealized example.

```go
type Vector []float64

// Apply the operation to v[i], v[i+1] ... up to v[n-1].
func (v Vector) DoSome(i, n int, u Vector, c chan int) {
    for ; i < n; i++ {
        v[i] += u.Op(v[i])
    }
    c <- 1    // signal that this piece is done
}
```

We launch the pieces independently in a loop, one per CPU. They can complete in any order but it doesn't matter; we just count the completion signals by draining the channel after launching all the goroutines.

```go
const numCPU = runtime.NumCPU() // number of CPU cores

func (v Vector) DoAll(u Vector) {
    c := make(chan int, numCPU)  // Buffering optional but sensible.
    for i := 0; i < numCPU; i++ {
        go v.DoSome(i*len(v)/numCPU, (i+1)*len(v)/numCPU, u, c)
    }
    // Drain the channel.
    for i := 0; i < numCPU; i++ {
        <-c    // wait for one task to complete
    }
    // All done.
}
```

Be sure not to confuse the ideas of concurrency—structuring a program as independently executing components—and parallelism—executing calculations in parallel for efficiency on multiple CPUs. Although the concurrency features of Go can make some problems easy to structure as parallel computations, **Go is a concurrent language, not a parallel one, and not all parallelization problems fit Go's model.** For a discussion of the distinction, see the talk cited in [this blog post](https://blog.golang.org/2013/01/concurrency-is-not-parallelism.html).

6.a leaky buffer

The tools of concurrent programming can even make non-concurrent ideas easier to express. Here's an example abstracted from an RPC package. The client goroutine loops receiving data from some source, perhaps a network. To avoid allocating and freeing buffers, it keeps a free list, and uses a buffered channel to represent it. If the channel is empty, a new buffer gets allocated. Once the message buffer is ready, it's sent to the server on `serverChan`.

```go
var freeList = make(chan *Buffer, 100)
var serverChan = make(chan *Buffer)

func client() {
    for {
        var b *Buffer
        // Grab a buffer if available; allocate if not.
        select {
        case b = <-freeList:
            // Got one; nothing more to do.
        default:
            // None free, so allocate a new one.
            b = new(Buffer)
        }
        load(b)              // Read next message from the net.
        serverChan <- b      // Send to server.
    }
}
```

The server loop receives each message from the client, processes it, and returns the buffer to the free list.

```go
func server() {
    for {
        b := <-serverChan    // Wait for work.
        process(b)
        // Reuse buffer if there's room.
        select {
        case freeList <- b:
            // Buffer on free list; nothing more to do.
        default:
            // Free list full, just carry on.
        }
    }
}
```

This implementation builds a **leaky bucket** free list in just a few lines, relying on the buffered channel and the garbage collector for bookkeeping.

# Errors

(1) Library routines must often return some sort of error indication to the caller. As mentioned earlier, Go's multivalue return makes it easy to return a detailed error description alongside the normal return value. It is good style to use this feature to provide detailed error information. 

(2) When feasible, error strings should identify their origin, such as by having a prefix naming the operation or package that generated the error. For example, in package `image`, the string representation for a decoding error due to an unknown format is "image: unknown format".

(3) Callers that care about the precise error details can use a type switch or a type assertion to look for specific errors and extract details.

1.panic

(1) `panic`creates a run-time error that will stop the program. 

(2) Real library functions should avoid `panic`. If the problem can be masked or worked around, it's better to let things continue to run rather than taking down the whole program. 

2.recover

(1) When `panic` is called, including implicitly for run-time errors such as indexing a slice out of bounds or failing a type assertion, it immediately stops execution of the current function and begins unwinding the stack of the goroutine, running any deferred functions along the way. If that unwinding reaches the top of the goroutine's stack, the program dies.  A call to `recover` stops the unwinding and returns the argument passed to `panic`. Because the only code that runs while unwinding is inside deferred functions, `recover` is only useful inside deferred functions.

One application of `recover` is to shut down a failing goroutine inside a server without killing the other executing goroutines.

```go
func server(workChan <-chan *Work) {
    for work := range workChan {
        go safelyDo(work)
    }
}

func safelyDo(work *Work) {
    defer func() {
        if err := recover(); err != nil {
            log.Println("work failed:", err)
        }
    }()
    do(work)
}
```

In this example, if `do(work)` panics, the result will be logged and the goroutine will exit cleanly without disturbing the others. There's no need to do anything else in the deferred closure; calling `recover` handles the condition completely.

(2)If **something unexpected** happens, such as an index out of bounds, the code will fail even though we are using `panic` and `recover` to handle errors.

(3) With error handling in place, the `error` method (because it's a method bound to a type, it's fine, even natural, for it to have the same name as the builtin `error` type) makes it easy to report parse errors without worrying about unwinding the parse stack by hand.

Useful though this pattern is, it should be used only within a package.  it does not expose `panics` to its client. That is a good rule to follow.



