# What Is A Type System

A language type system specifies which operations are valid for which types.

# Type System In Go

## Essential Concepts

### Named (defined) Type

Types with name: such as `int`, `int64`, `float32`, `string`, `bool`, etc. These are predeclared.

Also any type that we create using the `type declaration` is also named typed.

Keep in mind that a named (defined) type is ALWAYS DIFFERENT FROM ANY OTHER TYPE.

### Unnamed Type

Array, struct, pointer, function, interface, slice, map, and channel types are all of Unnamed Type.

As they have no name, but have got a `type literal` description about how they are to be structured.

### Underlying Type

If `T` is one of the predeclared boolean, numeric, or string types, or a type literal, the corresponding underlying type is `T` itself. Otherwise, `T`'s underlying type is the underlying type of the type to which `T` refers in its [type declaration](https://golang.org/ref/spec#Type_declarations).

```go
type S string
type T map[S]int
```

`type T map[S]int`. As `S` underlying type is `string`. Shouldn’t the underlying type of `type T map[S]int` be `map[string]int` instead of `map[S]int`, because here we are talking about **underlying unnamed type** **map[S]int** and underlying type stop at first unnamed type ( or as the specs says “If *T* is a type literal, the corresponding underlying type is *T* itself” ).

## Assignability

![assignability](img\assignability.png)

The most important and frequent rule is that **both should have the same underlying type, and at least one of then is not a named type**.

```go
type aInt int

func main() {
	var i int = 10
	var ai aInt = 100
	i = ai
	printAiType(i)
}

func printAiType(ai aInt) {
	print(ai)
}
```

The above code will not compile and will give us compile time error **because the** `i` **is of named type** `int` **and** `ai` **is of named type** `aInt` **, even though their underlying type is the same.**

## Type Conversion

![Type Conversion](img\Type Conversion.png)

```go
type Meter int64

type Centimeter int32

func main() {
	var cm Centimeter = 1000
	var m Meter
	m = Meter(cm)
	print(m)
	cm = Centimeter(m)
	print(cm)
}
```

The above code will compile **as both** `Meter` **and** `Centimeter` **are of** `integers`**type,** and their underlying type are convertable between each other.

```go
type Meter struct {
	value int64
}

type Centimeter struct {
	value int32
}

func main() {
	cm := Centimeter{
		value: 1000,
	}

	var m Meter
	m = Meter(cm)
	print(m.value)
	cm = Centimeter(m)
	print(cm.value)
}
```

Notice the term “**Identical Underlying Type**”. Since the underlying type of field `Meter.value` is of `int64` and field `Centimeter.value` is of `int32` they are **not identical** as **A defined type is always different from any other type**. So we will get the compilation error for the above code.

# Reflection 

Reflection in computing is the ability of a program to examine its own structure, particularly through types; it's a form of metaprogramming.

We need to be precise about all this because reflection and interfaces are closely related. **Go's interfaces are statically typed**: a variable of interface type always has the same static type, and even though at run time the value stored in the interface variable may change type, that value will always satisfy the interface.

For example, 

```go
var r io.Reader
r = os.Stdin
r = bufio.NewReader(r)
r = new(bytes.Buffer)
// and so on
```

`io.Reader`is an interface with a method `Read`. Any type that implements a `Read` method with this signature is said to implement `io.Reader` . However, `r`'s type is always `io.Reader`whatever concrete value `r` may hold.

## The representation of an interface 

A variable of interface type stores **a pair**:  1) the value is the underlying concrete data item that implements the interface and 2) the type describes the full type of that item.

```go
var r io.Reader
tty, err := os.OpenFile("/dev/tty", os.O_RDWR, 0)
if err != nil {
    return nil, err
}
r = tty
```

`r` contains the (value, type) pair, (`tty`, `*os.File`).

(1) What methods are implemented?

Notice that the type `*os.File` implements methods other than `Read`**; the value inside carries all the type information about that value**.

```go
var w io.Writer
w = r.(io.Writer)
```

The item inside `r` also implements `io.Writer`, and so we can assign it to `w`. 

(2) What methods can be invoked?

**The static type of the interface determines what methods may be invoked with an interface variable**, even though the concrete value inside may have a larger set of methods. Therefore, `r`can invoke `Read`method only.

## The laws of reflection 

### 1. Reflection goes from interface value to reflection object.

To get started, there are two types we need to know about in [package reflect](https://golang.org/pkg/reflect/): [Type](https://golang.org/pkg/reflect/#Type) and [Value](https://golang.org/pkg/reflect/#Value). Those two types give access to the contents of an interface variable, and two simple functions, called `reflect.TypeOf`and `reflect.ValueOf`, retrieve `reflect.Type` and `reflect.Value` pieces out of an interface value.

(1) `reflect.TypeOf`

The signature of `reflect.TypeOf`is `func TypeOf(i interface{}) Type`.

When we call `reflect.TypeOf(x)`, `x` is first stored in an empty interface, which is then passed as the argument; `reflect.TypeOf` unpacks that empty interface to recover the type information.

```go
reflect.TypeOf(3.4) // float64
```

(2) `reflect.ValueOf`

The `reflect.ValueOf` function, of course, recovers the value.

```go
reflect.ValueOf(3.4)  // 3.4
```

We can also retrieve the type information through `Value`.

```go
reflect.ValueOf(3.4).Type() // float64
```

(3) other important methods 

- Both `Type` and `Value` have a `Kind` method that returns a constant indicating what sort of item is stored: `Uint`, `Float64`, `Slice`, and so on.
-  Also methods on `Value` with names like `Int` and `Float` let us grab values (as `int64` and `float64`) stored inside.

(4) some important properties 

- The "getter" and "setter" methods of `Value` operate on **the largest type** that can hold the value: `int64` for all the signed integers, for instance. That is, the `Int` method of `Value` returns an `int64` and the `SetInt` value takes an `int64`

- The `Kind` of a reflection object describes the **underlying type**, not the static type.  In other words, the `Kind`cannot discriminate an int from a `MyInt` even though the `Type` can.

```go
	type MyInt int
	var x MyInt = 7
	fmt.Println(reflect.TypeOf(x))        // MyInt
	fmt.Println(reflect.TypeOf(x).Kind()) // int
```

### 2. Reflection goes from reflection object to interface value.

Given a `reflect.Value` we can recover an interface value using the `Interface` method; in effect the method packs the type and value information back into an interface representation and returns the result. 

In short, the `Interface` method is the inverse of the `ValueOf` function, except that **its result is always of static type `interface{}`**.

### 3. To modify a reflection object, the value must be settable.

Just keep in mind that **reflection Values need the address of something in order to modify what they represent.**

If we want to modify a reflection object, the settability of it should be true which can be tested by `CanSet`method:

```go
var x float64 = 3.4
v := reflect.ValueOf(x)
fmt.Println("settability of v:", v.CanSet()) // false
```

Settability is a bit like addressability, but stricter. It's the property that a reflection object can modify the actual storage that was used to create the reflection object. Settability is determined by whether the reflection object holds the original item. When we say

```go
var x float64 = 3.4
v := reflect.ValueOf(x)
```

we pass a copy of `x` to `reflect.ValueOf`, so the interface value created as the argument to `reflect.ValueOf` is a copy of `x`, not `x` itself. Thus, we can't change`x`itself through `v`.

If we want to modify `x` by reflection, we must give the reflection library a pointer to the value we want to modify.

```go
var x float64 = 3.4
p := reflect.ValueOf(&x) // Note: take the address of x.
fmt.Println("type of p:", p.Type())
fmt.Println("settability of p:", p.CanSet()) // false
```

Why still false? The reflection object `p` isn't settable because but it's not `p` we want to set, it's (in effect) `*p`. To get to what `p` points to, we call the `Elem` method of `Value`, which indirects through the pointer, and save the result in a reflection `Value`called `v`:

```go
// Elem returns the value that the interface v contains or that the pointer v points to. 
// It panics if v's Kind is not Interface or Ptr. 
// It returns the zero Value if v is nil.
v := p.Elem()
fmt.Println("settability of v:", v.CanSet()) // true
v.SetFloat(7.1)
fmt.Println(v.Interface()) // 7.1
fmt.Println(x) // 7.1
```

*What if we want to modify the fields of a structure using reflection?*

```go
type T struct {
    A int
    B string
}
t := T{23, "skidoo"}
s := reflect.ValueOf(&t).Elem()
typeOfT := s.Type()
for i := 0; i < s.NumField(); i++ {
    f := s.Field(i) // Field returns a Value
    fmt.Printf("%d: %s %s = %v\n", i,
        typeOfT.Field(i).Name, f.Type(), f.Interface())
}

// 0: A int = 23
// 1: B string = skidoo
```

There's one more point about settability introduced in passing here: the field names of `T` are upper case (exported) because only exported fields of a struct are settable.

# `uintptr`vs`unsafe.Pointer`

In short, there are two kinds of representations of an untyped pointer in Go: `uintptr`and`unsafe.Pointer`.

## `uintptr`

`uintptr` is an **integer** type that is large enough to hold the bit pattern of any pointer.

## `unsafe.Pointer`

`Pointer` represents a **pointer** to an arbitrary type.

## The difference

The superficial difference is that you can do arithmetic on an `uintptr` but not on an `unsafe.Pointer` (or any other Go pointer).

> A `uintptr` is an integer, not a reference. Converting a `Pointer` to a `uintptr` creates an integer value with no pointer semantics. Even if a `uintptr` holds the address of some object, the garbage collector will not update that `uintptr`'s value if the object moves, nor will that `uintptr` keep the object from being reclaimed.

*Why is `Pointer` unsafe?*

The GC can and will use them to keep live objects from being reclaimed and to discover further live objects (if the `unsafe.Pointer` points to an object that has pointers of its own). If you create an `unsafe.Pointer` that doesn't point to an object, even for a brief period of time, the Go garbage collector may choose that moment to look at it and then crash because it found an invalid Go pointer. 

## The relation

 There are four special operations available for type Pointer that are not available for other types:

- A pointer value of any type can be converted to a `Pointer`.
- A `Pointer` can be converted to a pointer value of any type.
- A `uintptr` can be converted to a `Pointer`.
- A `Pointer` can be converted to a `uintptr`.

Let's say we want to modify a struct by moving a pointer like C:

```go
func main() {
	u:=new(user)
	fmt.Println(*u)

    // The first field has 0 offset.
	pName:=(*string)(unsafe.Pointer(u))
	*pName="John"

    // unsafe.Offsetof tells us a field's offset
    // Keep in mind that only uintptr can be offset
	pAge:=(*int)(unsafe.Pointer(uintptr(unsafe.Pointer(u))+unsafe.Offsetof(u.age)))
	*pAge = 20

	fmt.Println(*u)
}

type user struct {
	name string
	age int
}
```

Note that we shouldn't change 

```go
pAge:=(*int)(unsafe.Pointer(uintptr(unsafe.Pointer(u))+unsafe.Offsetof(u.age)))
```

into 

```go
temp:=uintptr(unsafe.Pointer(u))+unsafe.Offsetof(u.age)
pAge:=(*int)(unsafe.Pointer(temp))
```

This is dangerous because `uintptr`will not keep the object from being reclaimed so `temp`may be reclaimed before we modify the object it points to.