# Overview

## CSP

CSP stands for “Communicating Sequential Processes,” which is both a technique and the name of the paper that introduced it. In 1978, Charles Antony Richard Hoare published the paper in the Association for Computing Machinery (more popularly referred to as ACM).

To support his assertion that inputs and outputs needed to be considered language primitives, Hoare’s CSP programming language contained primitives to model input and output, or communication, between processes correctly (this is where the paper’s name comes from). Hoare applied the term processes to any encapsulated portion of logic that required input to run and produced output other processes would consume.

Using these primitives, Hoare walked through several examples and demonstrated how a language with first-class support for modeling communication makes solving problems simpler and easier to comprehend. Similar solutions in Go are a bit longer, but also carry with them this clarity.

Memory access synchronization isn’t inherently bad. Sometimes sharing memory is appropriate in certain situations, even in Go. However, the shared memory model can be difficult to utilize correctly — especially in large or complicated programs.

## Go’s Philosophy on Concurrency

Go does support more traditional means of writing concurrent code through memory access synchronization and the primitives that follow that technique. However, higher-level synchronization is better done via channels and communication. 

**Do not communicate by sharing memory. Instead, share memory by communicating.**

Go’s philosophy on concurrency can be summed up like this: aim for simplicity, use channels when possible, and treat goroutines like a free resource.

*How To Choose Between CSP-style Concurrency and Memory Memory Synchronization*

1.Are you trying to transfer ownership of data?

Channels help us communicate data by encoding that intent into the channel’s type. One large benefit of doing so is you can create buffered channels to implement a cheap in-memory queue and thus decouple your producer from your consumer. Another is that by using channels, you’ve implicitly made your concurrent code **composable** with other concurrent code.

2.Are you trying to guard internal state of a struct?

Remember the key word here is **internal**. If you find yourself exposing locks beyond a type, this should raise a red flag. Try to keep the locks constrained to a small lexical scope like this:

```go
type Counter struct {
    mu sync.Mutex
    value int
}

func (c *Counter) Increment() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.value++
}
```

3.Are you trying to coordinate multiple pieces of logic?

Remember that channels are inherently more composable than memory access synchronization primitives. Having locks scattered throughout your object-graph sounds like a nightmare. 

If you find yourself struggling to understand how your concurrent code works, why a deadlock or race is occurring, and you’re using primitives, this is probably a good indicator that you should switch to channels.

4.Is it a performance-critical section?

This absolutely does not mean, “I want my program to be performant, therefore I will only use mutexes.” Rather, if you have a section of your program that you have profiled, and it turns out to be a major bottleneck that is orders of magnitude slower than the rest of the program, using memory access synchronization primitives may help this critical section perform under load. This is because channels *use* memory access synchronization to operate, therefore they can only be slower.

# Go's Concurrency Building Blocks

## Goroutines

### 1.What Is A Goroutine

Goroutines can be considered a special class of coroutine.

Coroutines are simply concurrent subroutines (functions, closures, or methods in Go) that are nonpreemptive — that is, they cannot be interrupted. Instead, coroutines have multiple points throughout which allow for suspension or reentry

Goroutines don’t define their own suspension or reentry points; Go’s runtime observes the runtime behavior of goroutines and automatically suspends them when they block and then resumes them when they become unblocked.

### 2.Go’s Mechanism For Hosting Goroutines

Go’s mechanism for hosting goroutines is an implementation of what’s called an **M:N scheduler**, which means it maps M green threads, which are managed by a language's runtime, to N OS threads. 

### 3. Go's Model of Concurrency

Go follows a model of concurrency called the **fork-join** model. 

The word fork refers to the fact that at any point in the program, it can split off a child branch of execution to be run concurrently with its parent. The word join refers to the fact that at some point in the future, these concurrent branches of execution will join back together.

![fork join model](img/fork join model.jpg)

The `go` statement is how Go performs a fork, and the forked threads of execution are goroutines.

*Be Careful When Using Goroutines With Closures*

```go
func main() {
	var wg sync.WaitGroup
	for _, salutation := range []string{"hello", "greetings", "good days"} {
		wg.Add(1)
		go func() {
			defer wg.Done()
			fmt.Println(salutation)
		}()
	}
	wg.Wait()
}

// good days
// good days
// good days
```

In this example, the goroutine is running a closure that has closed over the iteration variable `salutation`, which has a type of `string`. As our loop iterates, salutation is being assigned to the next string value in the slice literal. Because the goroutines being scheduled may run at any point in time in the future, it is undetermined what values will be printed from within the goroutine.

This is an interesting side note about how Go manages memory. The Go runtime is observant enough to know that a reference to the `salutation` variable is still being held, and therefore will transfer the memory to the heap so that the goroutines can continue to access it. Usually on my machine, **the loop exits before any goroutines begin running**, so `salutation` is transferred to the heap holding a reference to the last value in my string slice, “good day.”

The proper way to write this loop is to pass a copy of `salutation` into  the closure so that by the time the goroutine is run, it will be operating on the data from its iteration of the loop:

```go
func main() {
	var wg sync.WaitGroup
	for _, salutation := range []string{"hello", "greetings", "good days"} {
		wg.Add(1)
		go func(salutation string) {
			defer wg.Done()
			fmt.Println(salutation)
		}(salutation)
	}
	wg.Wait()
}

// good day
// hello
// greetings
```

### 4. Goroutines Are Lightweight

A newly minted goroutine is given a few kilobytes, which is almost always enough. When it isn’t, the run-time grows (and shrinks) the memory for storing the stack automatically, allowing many goroutines to live in a modest amount of memory.

## The `sync` Package

The `sync` package contains the concurrency primitives that are most  useful for low-level memory access synchronization.

### WaitGroup

`WaitGroup` is a great way to wait for a set of concurrent operations to complete when you either don’t care about the result of the concurrent operation, or you have other means of collecting their results.

You can think of a `WaitGroup` like a concurrent-safe counter: calls to `Add` increment the counter by the integer passed in, and calls to `Done` decrement the counter by one. Calls to `Wait` block until the counter is zero.

```go
func main() {
	var wg sync.WaitGroup
	wg.Add(1)
	go func() {
		defer wg.Done()
	}()
	wg.Wait()
	fmt.Println("All goroutines done.")
}
```

Notice that the calls to `Add` are done outside the goroutines they’re helping to track. If we didn’t do this, we would have introduced a race condition; we could reach the call to `Wait` before either of the goroutines begin. Had the calls to `Add` been placed inside the goroutines’ closures, the call to `Wait` could have returned without blocking at all because the calls to `Add` would not have taken place.

### Mutex and RWMutex

1.Mutex

A Mutex shares memory by creating a convention developers must follow to synchronize access to the memory. You are responsible for coordinating access to this memory by guarding access to it with a mutex.

```go
func main() {
	count := 0
	var lock sync.Mutex
	var wg sync.WaitGroup
	for i := 0; i < 5; i++ {
		wg.Add(1)
		go func() {
			lock.Lock()
			defer lock.Unlock()
			count++
			fmt.Println("Incrementing: ", count)
			wg.Done()
		}()
	}
	for i := 0; i < 5; i++ {
		wg.Add(1)
		go func() {
			lock.Lock()
			defer lock.Unlock()
			count--
			fmt.Println("Decrementing: ", count)
			wg.Done()
		}()
	}
	wg.Wait()
	fmt.Println("All goroutines done.")
}
```

Notice that we always call `Unlock` within a `defer` statement. This is a very common idiom when utilizing a `Mutex` to ensure the call always happens, even when panicing. Failing to do so will probably cause your program to deadlock.

2.RWMutex

Perhaps not all of these processes will read and write to this memory. If this is the case, you can take advantage of a different type of mutex: `sync.RWMutex`.

`RWMutex` gives you a little bit more control over the memory. You can request a lock for reading, in which case you will be granted access unless the lock is being held for writing. This means that an arbitrary number of readers can hold a reader lock so long as nothing else is holding a writer lock.

In fact, this lock can be held by an arbitrary number of readers or a single writer.

It's usually advisable to use `RWMutex`instead of `Mutex`when it logically makes sense. 

### Cond

`Cond`is a meeting point for goroutines waiting for or announcing the occurrence of an event. An “event” is any arbitrary signal between two or more goroutines that carries no information other than the fact that it has occurred. Very often you’ll want to wait for one of these signals before continuing execution on a goroutine.

```go
// The NewCond function takes in a type that satisfies the sync.LOcker interface.
// This allows Cond to facilitate coordination with other goroutines
c := sync.NewCond(&sync.Mutex())
// Calling Lock() is necessary 
// because the call to Wait automatically calls
// Unlock() on the Locker when entered
c.L.Lock()
for !condition(){
	c.Wait()
}
// ... make use of condition ...
// Calling Unlock() is necessary 
// because when Call returns, it calls 
// Lock() on the Locker for the condition
c.L.Unlock()
```

Note that the call to `Wait` doesn’t just block, it suspends the current goroutine, allowing other goroutines to run on the OS thread. A few other things happen when you call `Wait`: upon entering `Wait`, `Unlock` is called on the `Cond` variable’s `Locker`, and upon exiting `Wait`, `Lock` is called on the `Cond` variable’s `Locker`. In my opinion, this takes a little getting used to; it’s effectively a hidden side effect of the method. It looks like we’re holding this lock the entire time while we wait for the condition to occur, but that’s not actually the case.

Like most other things in the `sync` package, usage of `Cond` works best when constrained to a tight scope, or exposed to a broader scope through a type that encapsulates it.

1.Signal

```go
func main() {
	c := sync.NewCond(&sync.Mutex{})
	q := make([]int, 0, 10)
	dequeue := func() {
		time.Sleep(time.Second)
		c.L.Lock()
		q = q[1:]
		fmt.Println("Dequeue")
		c.L.Unlock()
		c.Signal()
	}
	for i := 0; i < 10; i++ {
		c.L.Lock()
		// Block until at least one item is dequeued.
		for len(q) == 2 {
			c.Wait()
		}
		fmt.Println("Enqueue")
		q = append(q, i)
		go dequeue()
		c.L.Unlock()
	}
}
```

`Signal`is one of the two methods that the `Cond` type provides for notifying goroutines blocked on a `Wait` call that the condition has been triggered.

2.Broadcast

`Broadcast` sends a signal to all goroutines that are waiting. This is something channels can’t do easily and thus is one of the main reasons to utilize the `Cond` type.

```go
type Button struct {
	Clicked *sync.Cond
}

func main() {
	button := Button{sync.NewCond(&sync.Mutex{})}
	subscribe := func(c *sync.Cond, handler func()) {
		var wgGoroutine sync.WaitGroup
		wgGoroutine.Add(1)
		go func() {
			wgGoroutine.Done()
			c.L.Lock()
			defer c.L.Unlock()
			// Wait for signals to run handler.
			c.Wait()
			handler()
		}()
		wgGoroutine.Wait()
	}

	var wgClick sync.WaitGroup
	wgClick.Add(3)
	subscribe(button.Clicked, func() {
		fmt.Println("Maximizing window.")
		wgClick.Done()
	})
	subscribe(button.Clicked, func() {
		fmt.Println("Mouse clicked.")
		wgClick.Done()
	})
	subscribe(button.Clicked, func() {
		fmt.Println("Displaying annoying dialog box.")
		wgClick.Done()
	})
	button.Clicked.Broadcast()
	wgClick.Wait()
}

// Displaying annoying dialog box.
// Maximizing window.
// Mouse clicked.
```

### Once

```go
func main() {
	count := 0
	var once sync.Once
	var wg sync.WaitGroup
	wg.Add(100)
	for i := 0; i < 100; i++ {
		go func() {
			defer wg.Done()
			once.Do(func() {
				count++
			})
		}()
	}
	wg.Wait()
	fmt.Println(count) // 1
}
```

`sync.Once` is a type that utilizes some sync primitives internally to ensure that only one call to `Do` ever calls the function passed in — even on different goroutines.

Notice that `sync.Once` only counts the number of times `Do` is called, not how many times unique functions passed into `Do` are called.

```go
var count int
increment := func() { count++ }
decrement := func() { count-- }
var once sync.Once
once.Do(increment)
once.Do(decrement)
fmt.Printf("Count: %d\n", count) // Count: 1
```

### Pool

`Pool` is a concurrent-safe implementation of the object pool pattern.

At a high level, the pool pattern is a way to create and make available a fixed number, or pool, of things for use. It’s commonly used to constrain the creation of things that are expensive (e.g., database connections) so that only a fixed number of them are ever created, but an indeterminate number of operations can still request access to these things. In the case of Go’s `sync.Pool`, this data type can be safely used by multiple goroutines.

`Pool`’s primary interface is its `Get` method. When called, `Get` will first check whether there are any available instances within the pool to return to the caller, and if not, call its `New` member variable to create a new one. When finished, callers call `Put` to place the instance they were working with back in the pool for use by other processes.

When working with a `Pool`, just remember the following points:

- When instantiating `sync.Pool`, give it a New member variable that is thread-safe when called.
- When you receive an instance from `Get`, make no assumptions regarding the state of the object you receive back.
- Make sure to call `Put` when you’re finished with the object you pulled out of the pool. Otherwise, the `Pool` is useless. Usually this is done with `defer`.
- Objects in the pool must be roughly uniform in makeup.