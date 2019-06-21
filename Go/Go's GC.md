Plans for Go 1.4+ garbage collector:

- hybrid stop-the-world/concurrent collector
- stop-the-world part limited by a 10ms deadline
- CPU cores dedicated to running the concurrent collector
- **tri-color mark-and-sweep algorithm**
- non-generational
- non-compacting
- fully precise
- incurs a small cost if the program is moving pointers around
- lower latency, but most likely also lower throughput, than Go 1.3 GC

# How And When Is GC Invoked

Typically, language runtimes trigger GC when they find that the *heap is exhausted* and there's no more memory available to satisfy a memory allocation request by a mutator thread i.e. the running program. 

# Mark-Sweep Collector

Mark-sweep eliminates some of the problems of reference count. It can easily handle cyclic structures and it has lower overhead since it doesn’t need to maintain counts.

It gives up being able to detect garbage immediately. After marking is finished, it sweeps over all of memory and disposes of garbage. Several memories may be swept all at once instead of more spread out over time in the reference counting approach.

It consists of a ***mark phase*** that is followed up by a ***sweep phase***. During ***mark phase***, the collector walks over all the roots (global variables, local variables, stack frames, virtual and hardware registers etc.) and marks every object that it encounters by setting a bit somewhere in/around that object. And during the ***sweep phase***, it walks over the heap and reclaims memory from all the unmarked objects.

All ***mutator*** threads are stopped while the collector runs. This stop-the-world approach seems suboptimal but it greatly simplifies the implementation of ***collector*** because ***mutators*** can't change the state under it.

## ***Dijkstra's Tri-color Marking***

It is one of the most widely used algorithm in ***mark sweep*** category. It is particularly useful for implementing increment and concurrent collectors. 

Initially, all objects are **white**, and as the algorithm proceeds, objects are moved into the **grey** and then **black** sets, in such a way that eventually the orphaned (collectible) objects are left in the white set, which is then cleared. An important property of this algorithm is that it can run concurrently with the “mutator” (program).

The meaning of the three sets is this:

- Black/grey sets: Definitely accessible from the roots (not candidates for collection).
  - Black set: Definitely no pointers to any objects in the white set.
  - Grey set: Might have pointers to the white set.
- White set: Possibly accessible from the roots. Candidates for collection.

The important invariant is the “tricolor” invariant: no pointers go directly from the black set to the white set. It is this invariant which lets us eventually clear the white set.

