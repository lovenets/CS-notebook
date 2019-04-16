# slice

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

# map

file location: src\runtime\map.go

## struct 

A map is just a hash table. The data is arranged into ==an array of buckets==. Each bucket contains up to **8** key/value pairs. The low-order bits of the hash are used to select a bucket. Each bucket contains a few high-order bits of each hash to distinguish the entries within a single bucket. When searching for a key, top byte of each key will be compared at first.

If more than 8 keys hash to a bucket, we chain on extra buckets. When the hash table grows, we allocate a new array of buckets ==twice as big==. Buckets are incrementally copied from the old bucket array to the new bucket array. 

![img](https://user-gold-cdn.xitu.io/2017/11/29/16006583b2d81bcd?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

```go
// A header for a Go map.
type hmap struct {
	// Note: the format of the hmap is also encoded in cmd/compile/internal/gc/reflect.go.
	// Make sure this stays in sync with the compiler's definition.
	count     int // # live cells == size of map.  Must be first (used by len() builtin)
	flags     uint8
	B         uint8  // log_2 of # of buckets (can hold up to loadFactor * 2^B items)
	noverflow uint16 // approximate number of overflow buckets; see incrnoverflow for details
	hash0     uint32 // hash seed

	buckets    unsafe.Pointer // array of 2^B Buckets. may be nil if count==0.
	oldbuckets unsafe.Pointer // previous bucket array of half the size, non-nil only when growing
	nevacuate  uintptr        // progress counter for evacuation (buckets less than this have been evacuated)

	extra *mapextra // optional fields
}

// A bucket for a Go map.
type bmap struct {
	// tophash generally contains the top byte of the hash value
	// for each key in this bucket. If tophash[0] < minTopHash,
	// tophash[0] is a bucket evacuation state instead.
    // bucketCntBits = 3
	// bucketCnt = 1 << bucketCntBits
	tophash [bucketCnt]uint8
	// Followed by bucketCnt keys and then bucketCnt values.
	// NOTE: packing all the keys together and then all the values together makes the
	// code a bit more complicated than alternating key/value/key/value/... but it allows
	// us to eliminate padding which would be needed for, e.g., map[int64]int8.
	// Followed by an overflow pointer.
}
```

Note that "packing all the keys together and then all the values together" means in each bucket key/value pairs are organized like key/key/key....value/value/value... In this way, no padding is needed when the sizes of key and value are different. 

## make

### function signature 

```go
// makemap implements Go map creation for make(map[k]v, hint).
// If the compiler has determined that the map or the first bucket
// can be created on the stack, h and/or bucket may be non-nil.
// If h != nil, the map can be created directly in h.
// If h.buckets != nil, bucket pointed to can be used as the first bucket.
func makemap(t *maptype, hint int, h *hmap) *hmap
```

### workflow 

(1) Initialize struct`hamp`

```go
	mem, overflow := math.MulUintptr(uintptr(hint), t.bucket.size)
	if overflow || mem > maxAlloc {
		hint = 0
	}

	// initialize Hmap
	if h == nil {
		h = new(hmap)
	}
	h.hash0 = fastrand()
```

(2) Calculate how many buckets are needed for storing # of key/value pairs

```go
	// Find the size parameter B which will hold the requested # of elements.
	// For hint < 0 overLoadFactor returns false since hint < bucketCnt.
	B := uint8(0)
	for overLoadFactor(hint, B) {
		B++
	}
	h.B = B
```

```go
// overLoadFactor reports whether count items placed in 1<<B buckets is over loadFactor.
func overLoadFactor(count int, B uint8) bool {
	return count > bucketCnt && uintptr(count) > loadFactorNum*(bucketShift(B)/loadFactorDen)
}
```

(3) Allocate initial hash table

```go
	// allocate initial hash table
	// if B == 0, the buckets field is allocated lazily later (in mapassign)
	// If hint is large zeroing this memory could take a while.
	if h.B != 0 {
		var nextOverflow *bmap
		h.buckets, nextOverflow = makeBucketArray(t, h.B, nil)
		if nextOverflow != nil {
			h.extra = new(mapextra)
			h.extra.nextOverflow = nextOverflow
		}
	}

	return h
```

Note that "if B == 0, the buckets field is allocated lazily later (in `mapassign`)" means buckets will not be allocated until we store data into map for the first time.

```go
// makeBucketArray initializes a backing array for map buckets.
// 1<<b is the minimum number of buckets to allocate.
// dirtyalloc should either be nil or a bucket array previously
// allocated by makeBucketArray with the same t and b parameters.
// If dirtyalloc is nil a new backing array will be alloced and
// otherwise dirtyalloc will be cleared and reused as backing array.
func makeBucketArray(t *maptype, b uint8, dirtyalloc unsafe.Pointer) (buckets unsafe.Pointer, nextOverflow *bmap)
```

## assign 

### function signature 

```go
// Like mapaccess, but allocates a slot for the key if it is not present in the map.
func mapassign(t *maptype, h *hmap, key unsafe.Pointer) unsafe.Pointer
```

### workflow

(1) Determine whether there exist concurrent map writes.

```go
	if h.flags&hashWriting != 0 {
		throw("concurrent map writes")
	}
```

(2) Calculate hash for the key to be inserted into map.

```go
again:
	bucket := hash & bucketMask(h.B)
	if h.growing() {
		growWork(t, h, bucket)
	}
	b := (*bmap)(unsafe.Pointer(uintptr(h.buckets) + bucket*uintptr(t.bucketsize)))
	// hash is generated by calling alg.hash(key, uintptr(h.hash0)), which alg.hash function hashes objects of this type (ptr to object, seed) -> hash
	// tophash generates the top byte of a hash
	top := tophash(hash)
```

(3) Find the place to insert key/value pair.

- Case 1: If the key is present, then update the value associated with it.

```go
	var inserti *uint8 // indicating where the key can be inserted into. Used for case 2
	var insertk unsafe.Pointer
	var val unsafe.Pointer
bucketloop:
	for {
        // Find a bucket.
		for i := uintptr(0); i < bucketCnt; i++ {
			if b.tophash[i] != top {
                // The bucket is empty.
				if isEmpty(b.tophash[i]) && inserti == nil {
					inserti = &b.tophash[i]
                      // Calculate the offset of current bucket 
                      // data offset should be the size of the bmap struct,
                      // but needs to be aligned correctly. 
					insertk = add(unsafe.Pointer(b), dataOffset+i*uintptr(t.keysize))
                      // bucketCnt == 8 i.e. there are at most 8 paris in each bucket
                      // dataOffset+bucketCnt*uintptr(t.keysize) == the position of first value
					val = add(unsafe.Pointer(b), dataOffset+bucketCnt*uintptr(t.keysize)+i*uintptr(t.valuesize))
				}
                 // Since the bucket is empty, we can insert the pair into it.
				if b.tophash[i] == emptyRest {
					break bucketloop
				}
				continue
			}
			k := add(unsafe.Pointer(b), dataOffset+i*uintptr(t.keysize))
             // indirectkey() store ptr to key instead of key itself
			if t.indirectkey() {
				k = *((*unsafe.Pointer)(k))
			}
			if !alg.equal(key, k) {
				continue
			}
			// already have a mapping for key. Update it.
			if t.needkeyupdate() {
				typedmemmove(t.key, k, key)
			}
			val = add(unsafe.Pointer(b), dataOffset+bucketCnt*uintptr(t.keysize)+i*uintptr(t.valuesize))
			goto done
		}
		ovf := b.overflow(t)
		if ovf == nil {
			break
		}
		b = ovf
	}
```

- If the key is absent, insert it into a bucket. If there exists not-full buckets, then insert it into the first one; else, allocate a new overflow bucket. 

```go
	// Did not find mapping for key. Allocate new cell & add entry.

	// If we hit the max load factor or we have too many overflow buckets,
	// and we're not already in the middle of growing, start growing.
	if !h.growing() && (overLoadFactor(h.count+1, h.B) || tooManyOverflowBuckets(h.noverflow, h.B)) {
		hashGrow(t, h)
		goto again // Growing the table invalidates everything, so try again
	}

	if inserti == nil {
		// all current buckets are full, allocate a new one.
		newb := h.newoverflow(t, b)
		inserti = &newb.tophash[0]
		insertk = add(unsafe.Pointer(newb), dataOffset)
		val = add(insertk, bucketCnt*uintptr(t.keysize))
	}

	// store new key/value at insert position
	if t.indirectkey() {
		kmem := newobject(t.key)
		*(*unsafe.Pointer)(insertk) = kmem
		insertk = kmem
	}
	if t.indirectvalue() {
		vmem := newobject(t.elem)
		*(*unsafe.Pointer)(val) = vmem
	}
	typedmemmove(t.key, insertk, key)
	*inserti = top
	h.count++
```

Note that if hash table is grown, find a suitable place again.

(4) Determine whether there exist concurrent map writes again.

```go
	if h.flags&hashWriting == 0 {
		throw("concurrent map writes")
	}
	h.flags &^= hashWriting
	if t.indirectvalue() {
		val = *((*unsafe.Pointer)(val))
	}
	return val
```

## access

### function signature 

```go
// mapaccess1 returns a pointer to h[key].  Never returns nil, instead
// it will return a reference to the zero object for the value type if
// the key is not in the map.
// NOTE: The returned pointer may keep the whole map live, so don't
// hold onto it for very long.
func mapaccess1(t *maptype, h *hmap, key unsafe.Pointer) unsafe.Pointer

func mapaccess2(t *maptype, h *hmap, key unsafe.Pointer) (unsafe.Pointer, bool)

// returns both key and value. Used by map iterator
func mapaccessK(t *maptype, h *hmap, key unsafe.Pointer) (unsafe.Pointer, unsafe.Pointer)

func mapaccess1_fat(t *maptype, h *hmap, key, zero unsafe.Pointer) unsafe.Pointer

func mapaccess2_fat(t *maptype, h *hmap, key, zero unsafe.Pointer) (unsafe.Pointer, bool)
```

### workflow

(1) Determine whether there exist concurrent map reading and writing.

```go
	if h.flags&hashWriting != 0 {
		throw("concurrent map read and map write")
	}
```

(2) Calculate top byte of hash of this key.

```go
	alg := t.key.alg
	hash := alg.hash(key, uintptr(h.hash0))
	m := bucketMask(h.B)
	b := (*bmap)(add(h.buckets, (hash&m)*uintptr(t.bucketsize)))
	if c := h.oldbuckets; c != nil {
		if !h.sameSizeGrow() {
			// There used to be half as many buckets; mask down one more power of two.
			m >>= 1
		}
		oldb := (*bmap)(add(c, (hash&m)*uintptr(t.bucketsize)))
		if !evacuated(oldb) {
			b = oldb
		}
	}
	top := tophash(hash)
```

(3) Find the bucket then find the key. If the key is absent, return zero value of value type.

```go
bucketloop:
    // b is the bucket 
	for ; b != nil; b = b.overflow(t) {
        // every bucket contains at most 8 pairs
		for i := uintptr(0); i < bucketCnt; i++ {
			if b.tophash[i] != top {
				if b.tophash[i] == emptyRest {
					break bucketloop
				}
				continue
			}
            // Get the key
			k := add(unsafe.Pointer(b), dataOffset+i*uintptr(t.keysize))
			if t.indirectkey() {
				k = *((*unsafe.Pointer)(k))
			}
            // Determine if keys are equal.
			if alg.equal(key, k) {
                // Get the value
				v := add(unsafe.Pointer(b), dataOffset+bucketCnt*uintptr(t.keysize)+i*uintptr(t.valuesize))
				if t.indirectvalue() {
					v = *((*unsafe.Pointer)(v))
				}
				return v
			}
		}
	}
	return unsafe.Pointer(&zeroVal[0])
```

## grow

### function signature 

```go
func growWork(t *maptype, h *hmap, bucket uintptr)
```

### workflow 

When the number of items in a map is greater than `6.5*2^B`(6.5 is load factor and B is the number of buckets), it shows that map may overflow and need to grow. New bucket will be allocated and elements in old bucket will be hashed again and copied, which is called `evacuate`.

## delete 

(1) Determine whether there exists concurrent map writes.

```go
	if h.flags&hashWriting != 0 {
		throw("concurrent map writes")
	}
```

(2) Search and delete key and value associated with it.

```go
bucket := hash & bucketMask(h.B)
	if h.growing() {
		growWork(t, h, bucket)
	}
	b := (*bmap)(add(h.buckets, bucket*uintptr(t.bucketsize)))
	bOrig := b
	top := tophash(hash)
search:
	for ; b != nil; b = b.overflow(t) {
		for i := uintptr(0); i < bucketCnt; i++ {
			if b.tophash[i] != top {
				if b.tophash[i] == emptyRest {
					break search
				}
				continue
			}
			k := add(unsafe.Pointer(b), dataOffset+i*uintptr(t.keysize))
			k2 := k
			if t.indirectkey() {
				k2 = *((*unsafe.Pointer)(k2))
			}
			if !alg.equal(key, k2) {
				continue
			}
			// Only clear key if there are pointers in it.
			if t.indirectkey() {
                // clear key
				*(*unsafe.Pointer)(k) = nil
			} else if t.key.kind&kindNoPointers == 0 {
				memclrHasPointers(k, t.key.size)
			}
			v := add(unsafe.Pointer(b), dataOffset+bucketCnt*uintptr(t.keysize)+i*uintptr(t.valuesize))
			if t.indirectvalue() {
                // clear value
				*(*unsafe.Pointer)(v) = nil
			} else if t.elem.kind&kindNoPointers == 0 {
				memclrHasPointers(v, t.elem.size)
			} else {
				memclrNoHeapPointers(v, t.elem.size)
			}
             // emptyOne means this cell is empty
			b.tophash[i] = emptyOne
```

(3) Determine whether the bucket is empty.

```go
			// If the bucket now ends in a bunch of emptyOne states,
			// change those to emptyRest states.
			// It would be nice to make this a separate function, but
			// for loops are not currently inlineable.
			if i == bucketCnt-1 {
				if b.overflow(t) != nil && b.overflow(t).tophash[0] != emptyRest {
					goto notLast
				}
			} else {
				if b.tophash[i+1] != emptyRest {
					goto notLast
				}
			}
			for {
				b.tophash[i] = emptyRest
				if i == 0 {
					if b == bOrig {
						break // beginning of initial bucket, we're done.
					}
					// Find previous bucket, continue at its last entry.
					c := b
					for b = bOrig; b.overflow(t) != c; b = b.overflow(t) {
					}
					i = bucketCnt - 1
				} else {
					i--
				}
				if b.tophash[i] != emptyOne {
					break
				}
			}
		notLast:
			h.count--
			break search
		}
```

Note that the bucket will not be clear even if it is empty.

(4) Determine if there exist concurrent map writes again.

```go
	if h.flags&hashWriting == 0 {
		throw("concurrent map writes")
	}
	h.flags &^= hashWriting
```





