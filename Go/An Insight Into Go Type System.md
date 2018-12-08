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

`type T map[S]int`. As `S` underlying type is `string`. Shouldn’t the underlying type of `type T map[S]int` be `map[string]int` instead of `map[S]int`, because here we are talking about **underlying unnamed type** `**map[S]int**` and underlying type stop at first unnamed type ( or as the specs says “If `*T*` is a type literal, the corresponding underlying type is `*T*` itself” ).

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



