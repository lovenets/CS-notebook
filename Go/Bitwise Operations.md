# Built-in bitwise operators

| Operation       | Result   | Description                         |
| --------------- | -------- | ----------------------------------- |
| `0011 & 0101`   | 0001     | Bitwise AND                         |
| `0011 | 0101`   | 0111     | Bitwise OR                          |
| `0011 ^ 0101`   | 0110     | Bitwise XOR                         |
| `^0101`         | 1010     | Bitwise NOT (same as `1111 ^ 0101`) |
| `0011 &^ 0101`  | 0010     | Bitclear (AND NOT)                  |
| `00110101<<2`   | 11010100 | Left shift                          |
| `00110101<<100` | 00000000 | No upper limit on shift count       |
| `00110101>>2`   | 00001101 | Right shift                         |

- The binary numbers in the examples are for explanation only. Integer literals in Go must be specified in octal, decimal or hexadecimal.
- The bitwise operators take both signed and unsigned integers as input. The right-hand side of a shift operator, however, must be an unsigned integer.
- Shift operators implement arithmetic shifts if the left operand is a signed integer and logical shifts if it is an unsigned integer.

## The & Operator

```
Given operands a, b
AND(a, b) = 1; only if a = b = 1
               else = 0
```

1.we can use the & operator to clear (set to zero) the last least significant bits (LSB) to all zeros.

```go
func main() {
    var x uint8 = 0xAC    // x = 10101100
    x = x & 0xF0          // x = 10100000
}
```

2.We can test whether a number is odd or even with & operator. We can use the & operator apply a bitwise AND operation to an integer the value **1**. If the result is 1, then the original number is odd.

## The | Operator

```
Given operands a, b
OR(a, b) = 1; when a = 1 or b = 1
              else = 0 
```

1.We can use the nature of the bitwise OR operator to selectively set individual bits for a given integer.

```go
func main() {
    var a uint8 = 0
    a |= 196
    fmt.Printf(“%b”, a)
}
// prints 11000100
          ^^   ^    
```

2.Using OR is quite useful when doing bit masking techniques to set arbitrary bits for a given integer value.

### Bits as Configuration

We can combine the use of the OR and the AND as a way of specifying configuration values and reading them respectively. More specifically, we use **OR for setting up configuration and AND for interpreting configuration**.

```go
const (
    UPPER  = 1 // upper case
    LOWER  = 2 // lower case
    CAP    = 4 // capitalizes
    REV    = 8 // reverses
)
func main() {
    // configure
    fmt.Println(procstr(“HELLO PEOPLE!”, LOWER|REV|CAP))
}
func procstr(str string, conf byte) string {
    // reverse string
    rev := func(s string) string {
        runes := []rune(s)
        n := len(runes)
        for i := 0; i < n/2; i++ {
            runes[i], runes[n-1-i] = runes[n-1-i], runes[i]
        }
        return string(runes)
    }
 
    // query config bits
    if (conf & UPPER) != 0 {
        str = strings.ToUpper(str)
    }
    if (conf & LOWER) != 0 {
        str = strings.ToLower(str)
    }
    if (conf & CAP) != 0 {
        str = strings.Title(str)
    }
    if (conf & REV) != 0 {
        str = rev(str)
    }
    return str
}
```

## The ^ Operator

The XOR, exclusive OR, has the following properties:

```
Given operands a, b
XOR(a, b) = 1; only if a != b
     else = 0
```

1.We can toggle some bits (going from 0 to 1 or from 1 to 0).

```go
func main() {
    var a uint16 = 0xCEFF
    a ^= 0xFF00 // same a = a ^ 0xFF00
}
// a = 0xCEFF   (11001110 11111111)
// a ^=0xFF00   (00110001 11111111)
```

2.Two integers `a`, `b` **have the same signs** when `(a ^ b) ≥ 0` (or `(a ^ b) < 0` for opposite sign) is true.

### ^ as Bitwise Complement (NOT)

Unlike other languages (c/c++, Java, Python, JavaScript, etc), Go does not have a dedicated unary bitwise complement operator. Instead, the XOR operator `^` can also be used as a unary operator to apply one’s complement to a number. Given bit `x`, in Go `^x = 1 ^ x` which reverses the bit.

## The &^ Operator

```
Given operands a, b
AND_NOT(a, b) = AND(a, NOT(b))
```

This has the interesting property of clearing the bits in the first operand if the second operand is 1 as defined here:

```
AND_NOT(a, 1) = 0; clears a
AND_NOT(a, 0) = a; 
```

## The << and >> Operators

```
Given integer operands a and n,
a << n; shifts all bits in a to the left n times
a >> n; shifts all bits in a to the right n times
```

Notice that with each shift, the LSB on the right is zero-filled. Inversely, using the right shift operator each bit in a value can shift to the right with the MSB zero-filled on the left. **While signed numbers has an exception.**  When the value to be shifted (the left operand) is a signed value, Go automatically apply arithmetic shifts. During a right shift operation, the sign bit is copied (or extended) to fill the shifted slots.

Simplest tricks that can be done with the left and right shift operators are the multiplication and division where each shift position represents a power of two.

```go
a >> 1 // a divided by 2
a << 1 // a times 2
```

# Package math/bits

| Operation                        | Result   | Description                           |
| -------------------------------- | -------- | ------------------------------------- |
| `bits.UintSize`                  | 32 or 64 | Size of a `uint` in bits              |
| `bits.OnesCount8(00101110)`      | 4        | Number of one bits (population count) |
| `bits.Len8(00101110)`            | 6        | Bits required to represent number     |
| `bits.Len8(00000000)`            | 0        |                                       |
| `bits.LeadingZeros8(00101110)`   | 2        | Number of leading zero bits           |
| `bits.LeadingZeros8(00000000)`   | 8        |                                       |
| `bits.TrailingZeros8(00101110)`  | 1        | Number of trailing zero bits          |
| `bits.TrailingZeros8(00000000)`  | 8        |                                       |
| `bits.RotateLeft8(00101110, 3)`  | 01110001 | The value rotated left by 3 bits      |
| `bits.RotateLeft8(00101110, -3)` | 11000101 | The value rotated **right** by 3 bits |
| `bits.Reverse8(00101110)`        | 01110100 | Bits in reversed order                |
| `bits.ReverseBytes16(0x00ff)`    | `0xff00` | Bytes in reversed order               |