# Slice

## struct

```go
// Pointer represents a pointer to an arbitrary type.
type slice struct {
	array unsafe.Pointer
	len   int
	cap   int
}
```

## make

### file location:

src/runtime/slice.go

### function signature

```go
func makeslice(et *_type, len, cap int) unsafe.Pointer
func makeslice64(et *_type, len64, cap64 int64) unsafe.Pointer
```

### workflow 

1.Determine whether `panicmakeslicelen`or`panicmakeslicecap`will occur

```go
func makeslice(et *_type, len, cap int) unsafe.Pointer {
	mem, overflow := math.MulUintptr(et.size, uintptr(cap)) // Calculate memory size required
	if overflow || mem > maxAlloc || len < 0 || len > cap {
		// NOTE: Produce a 'len out of range' error instead of a
		// 'cap out of range' error when someone does make([]T, bignumber).
		// 'cap out of range' is true too, but since the cap is only being
		// supplied implicitly, saying len is clearer.
		// See golang.org/issue/4085.
		mem, overflow := math.MulUintptr(et.size, uintptr(len))
		if overflow || mem > maxAlloc || len < 0 {
			panicmakeslicelen()
		}
		panicmakeslicecap()
	}

	// ...
}
```

(1)  `MulUintptr` returns a * b and whether the multiplication overflowed.

(2) `maxAlloc` is the maximum size of an allocation.

```go
maxAlloc = (1 << heapAddrBits) - (1-_64bit)*1 
// heapAddrBits is the number of bits in a heap address.
// On most 64-bit platforms, we limit this to 48 bits based on a combination of hardware and OS limitations.
```

On 64-bit, it's theoretically possible to allocate `1<<heapAddrBits` bytes. On 32-bit, however, this is one less than 1<<32 because the number of bytes in the address space doesn't actually fit in a `uintptr`.

(3) `panicmakeslcelen`is firstly tested because saying len is clear.

2.Allocate necessary memory size of specific type.

```go
return mallocgc(mem, et, true)
```

`mallocogc`signature: 

```go
func mallocgc(size uintptr, typ *_type, needzero bool) unsafe.Pointer
```

3.There is also a function `makeslice64`.

```go
func makeslice64(et *_type, len64, cap64 int64) unsafe.Pointer {
    // Determine whether len64 or cap64 overflow 
	len := int(len64)
	if int64(len) != len64 {
		panicmakeslicelen()
	}

	cap := int(cap64)
	if int64(cap) != cap64 {
		panicmakeslicecap()
	}

	return makeslice(et, len, cap)
}
```

## append

### file location

src/runtime/slice.go

### function signature 

```go
// growslice handles slice growth during append.
// It is passed the slice element type, the old slice, and the desired new minimum capacity,
// and it returns a new slice with at least that capacity, with the old data
// copied into it.
// The new slice's length is set to the old slice's length,
// NOT to the new requested capacity.
// This is for codegen convenience. The old slice's length is used immediately
// to calculate where to write new values during an append.
// TODO: When the old backend is gone, reconsider this decision.
// The SSA backend might prefer the new length or to return only ptr/cap and save stack space.
func growslice(et *_type, old slice, cap int) slice
```

### workflow 

1.Determine if the parameter is legal.

```go
	if raceenabled {
		callerpc := getcallerpc()
		racereadrangepc(old.array, uintptr(old.len*int(et.size)), callerpc, funcPC(growslice))
	}
	if msanenabled {
		msanread(old.array, uintptr(old.len*int(et.size)))
	}
	// If cap is less than old.cap which means slice will shrink
	// it should panic 
	if cap < old.cap {
		panic(errorString("growslice: cap out of range"))
	}

	if et.size == 0 {
		// append should not create a slice with nil pointer but non-zero len.
		// We assume that append doesn't need to preserve old.array in this case.
		return slice{unsafe.Pointer(&zerobase), old.len, cap}
	}
```

2.Calaulate the capacity of new slice based on `old.cap`.

```go
	newcap := old.cap
	doublecap := newcap + newcap
	if cap > doublecap {
		newcap = cap
	} else {
		if old.len < 1024 {
			newcap = doublecap
		} else {
			// Check 0 < newcap to detect overflow
			// and prevent an infinite loop.
			for 0 < newcap && newcap < cap {
				newcap += newcap / 4
			}
			// Set newcap to the requested cap when
			// the newcap calculation overflowed.
			if newcap <= 0 {
				newcap = cap
			}
		}
	}
```

(1) If required capacity is greater than double old capacity, then new capacity is required capacity.

(2) If old length (the number of elements) is less than 1024, then new capacity is double old capacity.

(3) If  old length is not less than 1024, then we do a loop `newcap += newcap / 4` until `newcap`overflows or `newcap`is not less than required capacity. If `newcap`at last overflows, then `newcap`is required capacity.

3.Calculate the memory size of new slice based on type and new capacity.

```go
	var overflow bool
	var lenmem, newlenmem, capmem uintptr
	// Specialize for common values of et.size.
	// For 1 we don't need any division/multiplication.
	// For sys.PtrSize, compiler will optimize division/multiplication into a shift by a constant.
	// For powers of 2, use a variable shift.
	switch {
	case et.size == 1:
		lenmem = uintptr(old.len)
		newlenmem = uintptr(cap)
		capmem = roundupsize(uintptr(newcap))
		overflow = uintptr(newcap) > maxAlloc
		newcap = int(capmem)
	case et.size == sys.PtrSize:
		lenmem = uintptr(old.len) * sys.PtrSize
		newlenmem = uintptr(cap) * sys.PtrSize
		capmem = roundupsize(uintptr(newcap) * sys.PtrSize)
		overflow = uintptr(newcap) > maxAlloc/sys.PtrSize
		newcap = int(capmem / sys.PtrSize)
	case isPowerOfTwo(et.size):
		var shift uintptr
		if sys.PtrSize == 8 {
			// Mask shift for better code generation.
			shift = uintptr(sys.Ctz64(uint64(et.size))) & 63
		} else {
			shift = uintptr(sys.Ctz32(uint32(et.size))) & 31
		}
		lenmem = uintptr(old.len) << shift
		newlenmem = uintptr(cap) << shift
		capmem = roundupsize(uintptr(newcap) << shift)
		overflow = uintptr(newcap) > (maxAlloc >> shift)
		newcap = int(capmem >> shift)
	default:
		lenmem = uintptr(old.len) * et.size
		newlenmem = uintptr(cap) * et.size
		capmem, overflow = math.MulUintptr(et.size, uintptr(newcap))
		capmem = roundupsize(capmem)
		newcap = int(capmem / et.size)
	}

	// The check of overflow in addition to capmem > maxAlloc is needed
	// to prevent an overflow which can be used to trigger a segfault
	// on 32bit architectures with this example program:
	//
	// type T [1<<27 + 1]int64
	//
	// var d T
	// var s []T
	//
	// func main() {
	//   s = append(s, d, d, d, d)
	//   print(len(s), "\n")
	// }
	if overflow || capmem > maxAlloc {
		panic(errorString("growslice: cap out of range"))
	}
```

4.Allocate memory and copy values from old slice.

```go
var p unsafe.Pointer
if et.kind&kindNoPointers != 0 {
   p = mallocgc(capmem, nil, false)
   // The append() that calls growslice is going to overwrite from old.len to cap (which will be the new length).
   // Only clear the part that will not be overwritten.
   memclrNoHeapPointers(add(p, newlenmem), capmem-newlenmem)
} else {
   // Note: can't use rawmem (which avoids zeroing of memory), because then GC can scan uninitialized memory.
   p = mallocgc(capmem, et, true)
   if writeBarrier.enabled {
      // Only shade the pointers in old.array since we know the destination slice p
      // only contains nil pointers because it has been cleared during alloc.
      bulkBarrierPreWriteSrcOnly(uintptr(p), uintptr(old.array), lenmem)
   }
}
memmove(p, old.array, lenmem)

return slice{p, old.len, newcap}
```

## copy 

### file location

src/runtime/slice.go

### function signature 

```go
func slicecopy(to, fm slice, width uintptr) int
func slicestringcopy(to []byte, fm string) int
```

### workflow 

1.Determine if at least one of `to`and`fm`slices is empty. If true, no need to copy anything.

```go
	if len(fm) == 0 || len(to) == 0 {
		return 0
	}
```

2.Find the shorter length n because `copy`function only copy n elements. So maybe not all of elements of `to`slice will be overwritten.

```go
	n := fm.len
	if to.len < n {
		n = to.len
	}
```

3.Copy n elements from `fm`slice and return n.

```go
	size := uintptr(n) * width
	if size == 1 { // common case worth about 2x to do here
		// TODO: is this still worth it with new memmove impl?
		*(*byte)(to.array) = *(*byte)(fm.array) // known to be a byte pointer
	} else {
		memmove(to.array, fm.array, size)
	}
	return n
```

There is another version `copy`function `slicestringcopy`which aims at copying a string into a slice of bytes.



