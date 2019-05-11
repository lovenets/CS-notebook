# CPU Virtualization 

## The Process

Time sharing is one of the most basic techniques used by an OS to share a resource. By allowing the resource to be used for a little while by one entity, and then a little while by another, and so forth, the resource in question (e.g., the CPU, or a network link) can be shared by many. 

**Time sharing** of the CPU virtualizes the CPU, which can promote the illusion that many virtual CPUs exist when in fact there is only one physical CPU (or a few).

 ### The Abstraction: A Process 

A process is just a running program. To understand what constitutes a process, we thus have to understand its machine state.

*components of machine state*

1.memory

The memory that the process can address (called its address space) is part of the process. 

2.registers

(1) program counter (PC): which instruction of the program is currently being executed.

(2) stack pointer and associated frame pointer manage the stack for  function parameters, local variables, and return addresses. 

3.I/O information: about persistent storage devices. 

### Process Creation 

![process creation](img/process creation.jpg)

1. OS must load its **code and any static data** (e.g., initialized variables) into memory, into the address space of the process.  
2. Some memory must be allocated for the program’s **run-time stack** (or just **stack**). C programs use the stack for local variables, function parameters, and return addresses 
3. The OS may also allocate some memory for the program’s **heap**. In C programs, the heap is used for explicitly requested dynamically allocated data.
4. The OS will also do some other initialization tasks, particularly as related to I/O. 
5.  OS starts the program running at the entry point, namely `main()`.  The OS transfers control of the CPU to the newly-created process, and thus the program begins its execution. 

### Process States 

- **Running**: In the running state, a process is running on a processor.
  This means it is executing instructions.
- **Ready**: In the ready state, a process is ready to run but for some
  reason the OS has chosen not to run it at this given moment
- **Blocked**: In the blocked state, a process has performed some kind of operation that makes it not ready to run until some other event
  takes place. 

![process states transitions](img/process states transitions.jpg)

### Data Structures 

The OS is a program, and like any program, it has some key data structures that track various relevant pieces of information.  

1.process list

Any OS that has the ability to run multiple programs at once will have something akin to this structure in order to keep track of all the running programs in the system. 

2.Process Control Block (PCB)

It's a C structure that contains information about each process.

## Process API

### The fork() System Call

The process calls the `fork()` system call, which the OS provides as a way to create a new process. 

The odd part: to the OS, it now looks like there are two copies of the program running, and both are about to return from the `fork()` system call. However, the child isn't an *exact* copy. Specifically, although it now has its own copy of the address space (i.e., its own private memory), its own registers, its own PC, and so forth, the value it returns to the caller of `fork()` is different.  

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main(int argc, char *argv[]) {
    printf("hello, world (pid:%d)\n", (int) getpid());
    int rc = fork();
    if (rc < 0) {
        fprintf(stderr, "fork failed\n");
        exit(1);
    } else if (rc == 0) { // child 
        printf("hello, I am child (pid:%d)\n", (int) getpid());
    } else { // parent
        printf("hello, I am parent of %d (pid:%d)\n", rc, (int) getpid());
    }
}

// output (not deterministic):
// hello world (pid:29146)
// hello, I am parent of 29147 (pid:29146)
// hello, I am child (pid:29147)
```

### The wait() System Call

The parent process calls `wait()` to delay its execution until the child finishes executing. When the child is done, `wait()` returns to the parent. So if the parent does happen to run first, it will immediately call `wait()`; this system call won’t return until the child has run and exited.

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main(int argc, char *argv[]) {
    printf("hello, world (pid:%d)\n", (int) getpid());
    int rc = fork();
    if (rc < 0) {
        fprintf(stderr, "fork failed\n");
        exit(1);
    } else if (rc == 0) { // child 
        printf("hello, I am child (pid:%d)\n", (int) getpid());
    } else { // parent
        int wc = wait(NULL);
        printf("hello, I am parent of %d (pid:%d)\n", rc, (int) getpid());
    }
}

// output (deterministic):
// hello world (pid:29146)
// hello, I am child (pid:29147)
// hello, I am parent of 29147 (pid:29146)
```

### The exec() System Call

Calling `fork()`is only useful if you want to keep running "copies" of the same program. However, often you want to run a different program; `exec()` does just that.

 Actually, there are six variants of `exec()` in Unix: `execl()`, `execle()`, `execlp()`, `execv()`, and `execvp()`.

`exec()`is quite odd too. What it does:

1. Given the name of an executable, and some arguments, it loads code (and static data) from that executable and overwrites its current code segment (and current static data) with it.
2. The heap and stack and other parts of the memory space of
   the program are re-initialized. 
3. The OS simply runs that program, passing in any arguments as the `argv` of that process.

Thus, it does **not** create a new process; rather, it just transforms the currently running process into a different process. Note that a successful call to `exec()` never returns. 

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main(int argc, char *argv[]) {
    printf("hello, world (pid:%d)\n", (int) getpid());
    int rc = fork();
    if (rc < 0) {
        fprintf(stderr, "fork failed\n");
        exit(1);
    } else if (rc == 0) { // child 
        printf("hello, I am child (pid:%d)\n", (int) getpid());
        char *myargs[3];
        myargs[0] = strdup("wc"); // program: "wc" (word count)
        myargs[1] = strdup("p3.c"); // argument
        myargs[2] = NULL;
        execvp(myargs[0], myargs);
        printf("this shouldn't print out");
    } else { // parent
        int wc = wait(NULL);
        printf("hello, I am parent of %d (pid:%d)\n", rc, (int) getpid());
    }
}

// output (deterministic):
// hello world (pid:29146)
// hello, I am child (pid:29147)
//  29   107   1030  p3.c
// hello, I am parent of 29147 (pid:29146)
```

The separation of `fork()` and `exec()` is essential in building a UNIX shell, because it lets the shell run code after the call to `fork()` but before the call to `exec()`. 

For example, you then type a command into it; in most cases, the shell then figures out where in the file system the executable resides, calls `fork()` to create a new child process to run the command, calls some variant of `exec()` to run the command, and then waits for the command to complete by calling `wait()`. When the child completes, the shell returns from `wait()` and prints out a prompt again, ready for your next command.

 ## Mechanism: Limited Direct Execution

The OS must virtualize the CPU in an efficient manner while retaining
control over the system. OS developers came up with a technique, which we call **limited direct execution**.

### Restricted Operations 

Direct execution has the obvious advantage of being fast; the program
runs natively on the hardware CPU and thus executes as quickly as one would expect.  But what if the process wishes to performs some kind of restricted operations such as I/O?

The approach we take is to introduce a new processor mode: **user mode.** Code that runs in user mode is restricted in what it can do. In contrast to the user mode is **kernel mode**, which the OS runs in. Code that runs can do what it likes including privileged operations.  

Executing a system call can turn user mode into kernel mode. To execute a system call, a program must execute a special **trap** instruction. This instruction simultaneously jumps into the kernel and raises the privilege level to kernel mode. When finished, the OS calls a
special **return-from-trap** instruction, which returns into the calling user program while simultaneously reducing the privilege level back to user mode. 

To specify the exact system call, a **system-call number** is usually assigned to each system call. The user code is thus responsible for placing the desired system-call number in a register or at a specified location on the stack; the OS, when handling the system call inside the trap handler, examines this number, ensures it is valid, and, if it is, executes the corresponding code. This level of **indirection** serves as a form of protection; user code cannot specify an exact address to jump to, but rather must request a particular service via number.

**LDE Protocol Timeline**

![LDE protocol](img/LDE protocol.jpg)

A trap table is a responsible for "remembering" which code to run inside the OS when calling system call and trapping into OS.

### Switching Between Processes

How can the operating system regain control of the CPU so that it can
switch between processes? 

**A Cooperative Approach: Wait For System Call**

The OS regains control of the CPU by waiting for a system call (like opening a file and subsequently reading it) or an illegal operation of some kind (like accessing memory illegally) to take place.  

**A Non-Cooperative Approach: The OS Takes Control**

The cooperative approach brings in a another problem: how can the OS gain control of the CPU even if processes are not being cooperative, for example, a process is stuck in infinite loop? 

A short answer: a **timer interrupt**.A timer device can be programmed to raise an interrupt every so many milliseconds; when the interrupt is raised, the currently running process is halted, and a pre-configured **interrupt handler** in the OS runs. At this point, the OS has regained control of the CPU, and thus can stop the current process and start another. 

The timer is started by OS during the boot time and can also be turned off later.

**Saving and Restoring Context**

After the OS has regained control, it has to decide whether to continue running the currently-running process or switch to different one. If the decision is to switch, the OS executes a low-level piece of code called **context switch**. All the OS has to do is save a few register values for the currently-executing process (onto its kernel stack, for example) and restore a few for the soon-to-be-executing process (from its kernel stack). 

To save the context of the currently-running process, the OS will execute some low-level assembly code to save the **general purpose registers, PC, and the kernel stack pointer** of the currently-running process, and then restore said registers, PC, and switch to the kernel stack for the soon-to-be-executing process. By switching stacks, the kernel enters the call to the switch code in the context of one process (the one that was interrupted) and returns in the context of another (the soon-to-be-executing one). When the OS then finally executes a return-from-trap instruction, the soon-to-be-executing process becomes the currently-running process. 

![switch context](img/switch context.jpg)

Note that there are two types of register saves/restores that happen during this protocol. The first is when the timer interrupt occurs; in this case, the **user registers** of the running process are implicitly saved by the **hardware**, using the kernel stack of that process. The second is when the OS decides to switch from A to B; in this case, the **kernel registers** are explicitly saved by the **software** (i.e., the OS), but this time into memory in the process structure of the process. 

## Scheduling Policies  Less Realistic 

### Workload Assumptions

To simplify the problem, we will make the following *unrealistic* assumptions about the processes:

- Each process runs for the same amount of time.
- All processes arrive at the same time.
- Once started, each process runs to completion. 
- All processes only use the CPU (i.e., they perform no I/O)
- The run-time of each process is known.

### Scheduling Metrics 

1.turnaround time 

$$T_{turnaround}=T_{completion}-T_{arrival}$$

Since we have assumed that all processes arrive at the same time, for now $$T_{arrival}=0$$, and hence $$T_{turnaround}=T_{completion}$$. 

Note that turnaround time is a performance metric, which will be our primary focus. 

2.fairness 

Performance and fairness are often at odds in scheduling; a scheduler, for example, may optimize performance but at the cost of preventing a few jobs from running, thus decreasing fairness. 

### FIFO

FIFO is scheduling is simple and thus easy to implement.

But if each process runs for different amount of time, a bad situation will occur where a number of relatively-short potential consumers of a resource get queued behind a heavyweight resource consumer. 

### Shortest Job First (SJF)

Run the shortest job first, then the next shortest, and so on.  

But if each process can arrive at any time, short-run process may wait until long-run process has completed. 

### Shortest Time-to-Completion First (STCF)

To address this concern, we need to relax assumption 3 (that jobs must
run to completion). We also need some machinery within the scheduler itself: timer interrupts and context switching. 

Any time a new job enters the system, the STCF scheduler determines which of the remaining jobs (including the new job) has the least time left, and schedules that one.  

### Round Robin (RR)

**Response Time**

The introduction of time-shared machines changed all that. Now users would sit at a terminal and demand interactive performance from the system as well. And thus, a new metric was born: response time. Response time is defined as the time from when the job arrives in a system to the first time it is scheduled.  

$$T_{response}=T_{firstrun}-T_{arrival}​$$

While great for turnaround time, STCF is quite bad for response time and interactivity.  

**Round Robin**

Instead of running jobs to completion, RR runs a job for a time slice (sometimes called a **scheduling quantum**) and then switches to the next job in the run queue. It repeatedly does so until the jobs are finished. For this reason, RR is sometimes called **time-slicing**. 

Note that the length of a time slice must be a multiple of the timer-interrupt period.

The shorter a time slice is, the better the performance of RR under the response-time metric. However, making the time slice too short is problematic: suddenly the cost of context switching will dominate overall performance. 

### Overlap 

A scheduler clearly has a decision to make when a job initiates an I/O
request, because the currently-running job won’t be using the CPU during the I/O; it is blocked waiting for I/O completion.  

When taking I/O into account, **overlap** is better. CPU is used by one process while waiting for the I/O of another process to complete. By treating each CPU burst as a job, the scheduler makes sure processes that are “interactive” get run frequently. While those interactive jobs are performing I/O, other CPU-intensive jobs run, thus better utilizing the processor. 

## Scheduling Policy: The Multi-Level Feedback Queue

The multi-level feedback queue is an excellent example of a system that learns from the past to predict the future. 

### Basic Rules

MLFQ has a number of distinct **queues**, each assigned a different **priority level**. At any given time, a job that is ready to run is on a single queue. A job with higher priority (i.e., a job on a higher queue) is chosen to run. Of course, more than one job may be on a given queue, and thus have the same priority. In this case, we will just use round-robin scheduling among those jobs. 

- Rule 1: If Priority(A) > Priority(B), A runs.
- Rule 2: If Priority(A) = Priority(B), A & B runs in RR.

Rather than giving a fixed priority to each job, MLFQ varies the priority of a job based on its **observed behavior**. 

### How to Change Priority 

- Rule 3: When a job enters the system, it is placed at the highest priority.
- Rule 4a: If a job uses up an entire time slice while running, its priority is **reduced** (such as CPU-intensive jobs).
- Rule 4b: If a job gives up the CPU before the time slice is up, it stays at the **same** priority level (such as jobs issuing I/O). 

These rules may arise some problems. 

1. Starvation: if there are too many interactive jobs in the system, they will combine to consume **all** CPU time and thus long-running jobs will never receive any CPU time.
2. A smart user could rewrite their program to **game the scheduler**. Gaming the scheduler generally refers to the idea of doing something sneaky to trick the scheduler into giving you more than your fair share of the resource. 

### The Priority Boost 

- Rule 5: After some time period $$S$$, change all the jobs in the system to the highest priority. 

1.Advantage 

(1) Processes are guaranteed not to starve. A job with the highest priority will share the CPU with other high-priority jobs in a RR fashion.

(2) If a CPU-bound job becomes interactive, the scheduler treats it properly once it has received the priority boost. 

2.Disadvantage 

If $$S$$ is set too high, long-running jobs could starve; too low, and interactive jobs may not get a proper share of the CPU.

### Better Accounting 

If we want to prevent gaming of our scheduler, the scheduler should keep track how much a time slice a process used at a given level instead of just forgetting it. 

We thus rewrite Rules 4a and 4b to the following single rule:

- Rule 4: Once a job uses up its time allotment at a given level (regardless of how many times it has given up the CPU), its priority is reduced.

Without any protection from gaming, a process can issue an I/O just before a time slice ends and thus dominate CPU time. With such protections in place, regardless of the I/O behavior of the process, it slowly moves down the queues, and thus cannot gain an unfair share of the CPU.

### Tuning MLFQ And Other Issues 

One big question is how to parameterize such a scheduler. For example, how many queues should there be? How big should the time slice be per queue? How often should priority be boosted in order to avoid starvation and account for changes in behavior?

Unfortunately, there are no easy answers to these questions, and thus only some experience with workloads and subsequent tuning of the scheduler will lead to a satisfactory balance.

### MLFQ: Summary 

MLFQ approach has **multiple levels** of queues, and uses **feedback** to determine the priority of a given job. History is its guide: pay attention to how jobs behave over time and treat them accordingly.

- Rule 1: If Priority(A) > Priority(B), A runs (B doesn’t).
- Rule 2: If Priority(A) = Priority(B), A & B run in RR.
- Rule 3: When a job enters the system, it is placed at the highest priority (the topmost queue).
- Rule 4: Once a job uses up its time allotment at a given level (regardless of how many times it has given up the CPU), its priority is reduced (i.e., it moves down one queue).
- Rule 5: After some time period S, move all the jobs in the system to the topmost queue 

MLFQ can deliver excellent overall performance (similar to SJF/STCF) for short-running interactive jobs and is fair and makes progress for long-running CPU-intensive workloads. 

## Scheduling Policy: Proportional Share

Proportional-share is based around a simple concept: instead of optimizing for turnaround or response time, a scheduler might instead try to guarantee that each job obtain a certain percentage of CPU time.

An excellent modern example of proportional-share scheduling is **lottery scheduling**. The basic idea is quite simple: every so often, hold a lottery to determine which process should get to run next; processes that should run more often should be given more chances to win the lottery.

### Basic Concept: Tickets

Underlying lottery scheduling is one very basic concept: **tickets**, which are used to represent the share of a resource that a process (or user or whatever) should receive. The percent of tickets that a process has represents its share of the system resource in question. 

The scheduler must know how many total tickets there are (in our example, there are 100). The scheduler then picks a winning ticket, which is just a number. Assuming A holds tickets 0 through 74 and B 75 through 99, the winning ticket simply determines whether A or B runs. The scheduler then loads the state of that winning process and runs it.

### Ticket Mechanism 

1.ticket currency 

Currency allows a user with a set of tickets to allocate tickets among their own jobs in whatever currency they would like; the system then automatically converts said currency into the correct global value.

For example, assume users A and B have each been given 100 tickets.
User A is running two jobs, A1 and A2, and gives them each 500 tickets
(out of 1000 total) in User A’s own currency. User B is running only 1 job
and gives it 10 tickets (out of 10 total). The system will convert A1’s and
A2’s allocation from 500 each in A’s currency to 50 each in the global currency; similarly, B1’s 10 tickets will be converted to 100 tickets. The lottery will then be held over the global ticket currency (200 total) to determine which job runs. 

2.ticket transfer

This is useful especially in a C/S setting, where a client process sends a message to a server asking it to do some work on the client's behalf. 

With transfers, a process can temporarily hand off its tickets to another process. When finished, the process which received tickets before transfers the tickets back to the client and all is as before. 

3.ticket inflation 

With inflation, a process can temporarily raise or lower the number of tickets it owns. Inflation can be applied in an environment where a group of processes trust one another; in such a case, if any one process knows it needs more CPU time, it can boost its ticket value as a way to reflect that need to the system, all without communicating with any other processes.

### Implementation of Lottery Scheduling

Let’s assume we keep the processes in a list. Here is an example comprised of three processes, A, B, and C, each with some number of tickets. 

![implementation of lottery scheduling](img/implementation of lottery scheduling.jpg)

To make a scheduling decision, we first have to pick a random number (the winner) from the total number of tickets (400). Let’s say we pick the number 300. Then, we simply traverse the list, with a simple counter used to help us find the winner.

```c
// counter: used to track if we've found the winner yet
int counter = 0;

// winner: use some call to a random number generator 
// to get a value, between 0 and the total # of tickets
int winner = getrandom(0, totaltickets);

// current: use this to walk through the list of jobs
node_t *current = head;

// loop until the sum of ticket values is > the winner
while (current) {
    counter = counter + current->tickets;
    if (counter > winner) {
        break; // found the winner
    }
    current = current->next;
}
// 'current' is the winner: schedule it
```

To make this process most efficient, it might generally be best to organize the list in descending order. The ordering does not affect the correctness of the algorithm; however, it does ensure in general that the fewest number of list iterations are taken, especially if there are a few processes that possess most of the tickets.

### Stride Scheduling 

Lottery scheduling is random; while random gets us a simple (and approximately correct) scheduler, it occasionally will not deliver the exact right proportions, especially over short time scales. That's why **stride scheduling**, a deterministic fair-share scheduler, comes. 

Each job in the system has a stride, which is inverse in proportion to the number of tickets it has. In our example, with jobs A, B, and C, with 100, 50, and 250 tickets, respectively, we can compute the stride of each by dividing some large number by the number of tickets each process has been assigned. For example, if we divide 10,000 by each of those ticket values, we obtain the following stride values for A, B, and C: 100, 200, and 40. We call this value the **stride** of each process; every time a process runs, we will increment a counter for it (called its pass value) by its stride to track its global progress.

The scheduler then uses the stride and pass to determine which process should run next. The basic idea is simple: at any given time, pick the process to run that has the lowest pass value so far; when you run a process, increment its pass counter by its stride.

```pseudocode
current = remove_min(queue); // pick client with minimum pass
schedule(current); // use resource for quantum
current->pass += current->stride; // compute next pass using stride
insert(queue, current); // put back into the queue
```

Lottery scheduling achieves the proportions probabilistically over time; stride scheduling gets them exactly right at the end of each scheduling cycle.

*So why use lottery scheduling at all instead of stride scheduling?*

Lottery scheduling has one nice property that stride scheduling does not: **no global state**. Imagine a new job enters in the middle of our stride scheduling example above; what should its pass value be? Should it be set to 0? If so, it will monopolize the CPU. With lottery scheduling, there is no global state per process; we simply add a new process with whatever tickets it has, update the single global variable to track how many total tickets we have, and go from there. In this way, lottery  makes it much easier to incorporate new processes in a sensible manner.

### Summary 

Lottery and stride scheduling are wo implementations of proportional-share scheduling. 

Although both are conceptually interesting, they have not achieved wide-spread adoption as CPU schedulers for a variety of reasons. One is that such approaches do not particularly mesh well with I/O; another is that they leave open the hard problem of ticket assignment, i.e., how do you know how many tickets your browser should be allocated?

## Multiprocessor Scheduling

### Background: Multiprocessor Architecture

To understand the new issues surrounding multiprocessor scheduling, we have to understand a new and fundamental difference between single-CPU hardware and multi-CPU hardware. This difference centers around the use of hardware **caches**, and exactly how data is shared across multiple processors.

Caches are thus based on the notion of **locality**, of which there are two kinds: temporal locality and spatial locality. The idea behind temporal locality is that when a piece of data is accessed, it is likely to be accessed again in the near future. The idea behind spatial locality is that if a program accesses a data item at address x, it is likely to access data items near x as well.

### Synchronization

The solution, of course, is to make such routines correct via locking. In this case, allocating a simple mutex and then adding a `lock(&m)` at the beginning of the routine and an `unlock(&m)` at the end will solve the problem, ensuring that the code will execute as desired. Unfortunately, as we will see, such an approach  is not without problems, in particular with regards to performance. Specifically, as the number of CPUs grows, access to a synchronized shared data structure becomes quite slow.

### Cache Affinity

A process, when run on a particular CPU, builds up a fair bit of state in the caches (and TLBs) of the CPU. The next time the process runs, it is often advantageous to run it on the same CPU, as it will run faster if some of its state is already present in the caches on that CPU. If, instead, one runs a process on a different CPU each time, the performance of the process will be worse, as it will have to reload the state each time it runs (note it will run correctly on a different CPU thanks to the cache coherence protocols of the hardware). Thus, a multiprocessor scheduler should consider cache affinity when making its scheduling decisions, perhaps preferring to keep a process on the  same CPU if at all possible.

### Single-Queue Scheduling

The most basic approach is to simply reuse the basic framework for single processor scheduling, by putting all jobs that need to be scheduled into a single queue; we call this single-queue multiprocessor scheduling or **SQMS** for short.

1.advantages

This approach has the advantage of simplicity.

2.disadvantages 

(1) a lack of **scalability**

To ensure the scheduler works correctly on multiple CPUs, the developers will have inserted some form of **locking** into the code. Locks, unfortunately, can greatly reduce performance, particularly as the number of CPUs in the systems grows

(2) cache affinity 

Because each CPU simply picks the next job to run from the globally shared queue, each job ends up bouncing around from CPU to CPU, thus doing exactly the opposite of what would make sense from the standpoint of cache affinity.

### Multi-Queue Scheduling

Some systems opt for multiple queues, e.g., one per CPU. We call this approach multi-queue multiprocessor scheduling (or **MQMS**).

Each queue will likely follow a particular scheduling discipline, such as round robin, though of course any algorithm can be used. When a job enters the system, it is placed on exactly one scheduling queue, according to some heuristic (e.g., random, or picking one with fewer jobs than others). Then it is scheduled essentially **independently**, thus avoiding the problems of information sharing and synchronization found in the single-queue approach.

1.advantages 

(1) As the number of CPUs grows, so too does the number of queues, and thus lock and cache contention should not become a central problem.

(2) Jobs stay on the same CPU and thus reap the advantage of cache affinity.

2.disadavantages

Workload imbalance is the biggest problem. Let's say there is only one job running on CPU 0 and there are two jobs running on CPU 1, both CPU using round-robin policy. The resulting schedule:

![workload imbalance](img/workload imbalance.jpg)

How should a multi-queue multiprocessor scheduler handle load imbalance, so as to better achieve its desired scheduling goals? The obvious answer to this query is to move jobs around, a technique which we (once again) refer to as **migration**. By migrating a job from one CPU to another, true load balance can be achieved.

One basic approach is to use a technique known as **work stealing**. With a work-stealing approach, a (source) queue that is low on jobs will occasionally peek at another (target) queue, to see how full it is. If the target queue is (notably) more full than the source queue, the source will “steal” one or more jobs from the target to help balance load.

But if you look around at other queues too often, you will suffer from high overhead and have trouble scaling; if you don't look at other queues very often, you are in dangerous of suffering from severe load imbalance. Finding the right threshold remains, as is common in system policy design, a black art.

# Memory Virtualization

## Address Space 

Address space is the running program’s view of memory in the system. The address space of a process contains all of the memory state of the
running program, such as the **code**, **stack** and **heap**. Heap and stack may grow (and shrink) while the program runs. We put them at opposite ends of the address space so we can allow such growth: they just have to grow in opposite directions.

When the OS virtualizes memory, the running program thinks it is loaded into memory at a particular address (say 0) and has a potentially very large address space. When, for example, process A tries to perform a load at address 0 (which we will call a virtual address), somehow the OS, with some hardware support, will have to make sure the load doesn’t actually go to physical address 0 but rather to physical address 320KB (where A is loaded into memory).

So never forget: if you print out an address in a program, it’s a virtual one, an illusion of how things are laid out in memory; only the OS (and the hardware) knows the real truth.

### VM Goals

1.transparency

The OS should implement virtual memory in a way that is invisible to
the running program. Thus, the program shouldn’t be aware of the fact
that memory is virtualized; rather, the program behaves as if it has its
own private physical memory.

2.efficiency

The OS should strive to make the virtualization as efficient as possible, both in terms of time (i.e., not making programs run much more slowly) and space (i.e., not using too much memory for structures needed to support virtualization).

3.protection

Protection thus enables us to deliver the property of isolation among processes; each process should be running in its own isolated cocoon, safe from the ravages of other faulty or even malicious processes.

## Memory API

### Types of Memory

1.stack memory 

Allocations and deallocations of it are managed *implicitly* by the compiler for you, the programmer.

2.heap memory

All allocations and deallocations are *explicitly* handled by you, programmer.

### The `malloc()`Call

### The `free()`Call

### Memory Leak

No matter what the state of your heap in your address space, the OS takes back all of those pages when the process dies, thus ensuring that no memory is lost despite the fact that you didn’t free it.

Thus, for short-lived programs, leaking memory often does not cause any operational problems. When you write a long-running server (such as a web server or database management system, which never exit), leaked memory is a much bigger issue, and will eventually lead to a crash when the application runs out of memory.

### Underlying OS Support

`malloc`and`free`are not system calls, but rather library calls. Thus the malloc library manages space within your virtual address space, but itself is built on top of some system calls which call into the OS to ask for more memory or release some back to the system.

One such system call is called `brk`, which is used to change the location of the program’s break: the location of the end of the heap. It takes one argument (the address of the new break), and thus either increases or decreases the size of the heap based on whether the new break is larger or smaller than the current break. An additional call `sbrk` is passed an increment but otherwise serves a similar purpose.

## Address Translation

With address translation, the hardware transforms each memory access (e.g., an instruction fetch, load, or store), changing the **virtual** address provided by the instruction to a **physical** address where the desired information is actually located. Thus, on each and every memory reference, an address translation is performed by the hardware to redirect application memory references to their actual locations in memory.

### Dynamic (Hardware-based) Relocation

Specifically, we’ll need two hardware registers within each CPU: one is called the **base** register, and the other the **bounds** (sometimes called a **limit** register). This base-and-bounds pair is going to allow us to place the address space anywhere we’d like in physical memory, and do so  while ensuring that the process can only access its own address space.

1.base register

In this setup, each program is written and compiled as if it is loaded at address zero. However, when a program starts running, the OS decides where in physical memory it should be loaded and sets the base register to that value. When any memory reference is generated by the process, it is translated by the processor in the following manner:

`physical address = virtual address + base`

Because this relocation of the address happens at runtime, and because we can move address spaces even after the process has started running, the technique is often referred to as **dynamic relocation**.

2.bound register

The bounds register is there to help protection. If a process generates a virtual address that is greater than the value stored in the bounds register, or one that is negative, the CPU will raise an exception, and the process will likely be terminated.

More specifically, the bounds register can hold the *size* of the address space, and thus the hardware checks the virtual address against it first before adding the base; it can either hold the *physical address* of the end of the address space, and thus the hardware first adds the base and then makes sure the address is within bounds. Both methods are logically equivalent; for simplicity, we’ll usually assume the former method.

3.MMU

The base and bounds registers are hardware structures kept on the chip (one pair per CPU). Sometimes people call the part of the processor that helps with address translation the **memory management unit (MMU)**.

### Hardware Support

| Hardware Requirements                                        | Notes                                                        |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Privileged mode                                              | Needed to prevent user-mode processes from executing privileged operations |
| Base/bounds registers                                        | Need pair of registers pre CPU to support address translation and bounds checks |
| Ability to translate virtual addresses and check if within bounds | Circuitry to do translations and check limits; in this case, quite simple |
| Privileged instruction(s) to update base/bounds              | OS must be able to set these values before letting a user program run |
| Privileged instruction(s) to register exception handlers     | OS must be able to tell hardware what code to run if exception occurs |
| Ability to raise exceptions                                  | When processes try to access privileged instructions or out-of-bounds memory |

### OS Issues

| OS Requirements        | Notes                                                        |
| ---------------------- | ------------------------------------------------------------ |
| Memory management      | Need to allocate memory for new processes; Reclaim memory from terminated processes; Generally manage memory via free list. |
| Base/bounds management | Must set base/bounds properly upon context switch            |
| Exception handling     | Code to run when exceptions arise; likely action is to terminate offending process |

![dynamic relocation timeline](img/dynamic relocation timeline.jpg)

### Inefficiencies

Because the process stack and heap are not too big, all of the space between the two is simply wasted. This type of waste is usually called **internal fragmentation**, as the space inside the allocated unit is not all used (i.e., is fragmented) and thus wasted.

## Segmentation

How do we support a large address space with (potentially) a lot of
free space between the stack and the heap? To solve this problem, an idea was born, and it is called **segmentation**.

### Generalized Base/Bounds

Why not have a base and bounds pair per logical **segment** of the address space? A segment is just a *contiguous* portion of the address space of a particular length, and in our canonical address space, we have three logically-different segments: code, stack, and heap. What segmentation allows the OS to do is to place each one of those segments in different parts of physical memory, and thus avoid filling physical memory with unused virtual address space.

For example, as we can see in the diagram, only used memory is allocated space in physical memory, and thus large address spaces with large amounts of unused address space (which we sometimes call **sparse address spaces**) can be accommodated.

![placing segments in physical memory](img/placing segments in physical memory.jpg)

So there is a set of three base and bounds register pairs. 

| Segment | Base | Size |
| ------- | ---- | ---- |
| Code    | 32K  | 2K   |
| Heap    | 34K  | 2K   |
| Stack   | 28K  | 2K   |

### Which Segment?

How does the hardware know the offset into a segment, and to which  segment an address refers? 

1.explicit approach

The address space is chopped up into segments based on the top few bits of the virtual address. In our example above, we have three segments; thus we need two bits to accomplish our task. 

In our example, then, if the top two bits are 00, the hardware knows the virtual address is in the code segment, and thus uses the code base and bounds pair to relocate the address to the correct physical location. If the top two bits are 01, the hardware knows the address is in the heap, and thus uses the heap base and bounds. 

```pseudocode
// get top bits
Segment = (VitualAddress & SEG_MASK) >> SEG_SHIFT
// now get offset
Offset = VirtualAddress & OFFSET_MASK
if (Offset >= Bounds[Segment]) 
	RaiseException(PROTeCTION_FAULT)
else
	PhysAddr = Base[Segment] + Offset
	Register = AccessMemory(PhysAddr)
```

2.implicit approach

The hardware determines the segment by noticing how the address was formed. If, for example, the address was generated from the program counter (i.e., it was an instruction fetch), then the address is within the code segment; if the address is based off of the stack or base pointer, it must be in the stack segment; any other address must be in the heap.

**What About The Stack?**

The stack has one critical difference: it *grows backwards*. Instead of just base and bounds values, the hardware also needs to know which way the segment grows (a bit, for example, that is set to 1 when the segment grows in the positive direction, and 0 for negative).

| Segment | Base | Size | Grows Positive? |
| ------- | ---- | ---- | --------------- |
| Code    | 32K  | 2K   | 1               |
| Heap    | 34K  | 2K   | 1               |
| Stack   | 28K  | 2K   | 0               |

### Support Sharing

To save memory, sometimes it is useful to share certain memory segments between address spaces. In particular, **code sharing** is common and still in use in systems today.

To support sharing, we need a little extra support from the hardware, in the form of **protection bits**. Basic support adds a few bits per segment, indicating whether or not a program can read or write a segment, or perhaps execute code that lies within the segment. By setting a code segment to read-only, the same code can be shared across multiple processes, without worry of harming isolation;

In addition to checking whether a virtual address is within bounds, the hardware also has to check whether a particular access is permissible.

### OS Support 

1.What should the OS do on a context switch? 

The segment registers must be saved and restored. Clearly, each process has its own virtual address space, and the OS must make sure to set up these registers correctly before letting the process run again.

2.external fragmentation

Now, we have a number of segments per process, and each segment might be a different size. The general problem that arises is that physical memory quickly becomes full of little holes of free space, making it difficult to allocate new segments, or to grow existing ones.

A simpler approach is to use a free-list management algorithm that tries to keep large extents of memory available for allocation. Unfortunately, though, no matter how smart the algorithm, external fragmentation will still exist; thus, a good algorithm simply attempts to minimize it.

## Free-Space Management

How should free space be managed, when satisfying variable-sized requests?

### Low-Level Mechanisms

1.Splitting and Coalescing

A free list contains a set of elements that describe the free space still remaining in the heap.

![a free list](img/a free list.jpg)

(1) splitting

Assume we have a request for just a single byte of memory. In this case, the allocator will perform an action known as **splitting**: it will find a free chunk of memory that can satisfy the request and split it into two. The first chunk it will return to the caller; the second chunk will remain on the list.

(2) coalescing

A corollary mechanism found in many allocators is known as **coalescing** of free space. 

We don't  want this:

![memory shatters](img/memory shatters.jpg)

but this:

![complete memory](img/complete memory.jpg)

What allocators do in order to avoid this problem is coalesce free space when a chunk of memory is freed. The idea is simple: when returning a free chunk in memory, look carefully at the addresses of the chunk you are returning as well as the nearby chunks of free space; if the newly freed space sits right next to one (or two, as in this example) existing free chunks, merge them into a single larger free chunk.

2.Tracking The Size Of Allocated Regions

`free(void *ptr)` does not take a size parameter; thus it is assumed that given a pointer, the malloc library can quickly determine the size of the region of memory being freed and thus incorporate the space back into the free list.

To accomplish this task, most allocators store a little bit of extra information in a **header** block which is kept in memory, usually just before the handed-out chunk of memory.

![header of memory chunk](img/header of memory chunk.jpg)

When the user calls `free(ptr)`, the library then uses simple pointer arithmetic to figure out where the header begins:  

```c
void free(void *ptr) {
    header_t *hptr = (void *)ptr - sizeof(header_t);
    // ...
}
```

After obtaining such a pointer to the header, the library can easily determine whether the magic number matches the expected value as a sanity check `(assert(hptr->magic == 1234567))` and calculate the total size of the newly-freed region via simple math.

Note the small but critical detail in the last sentence: the size of the free region is the size of the header plus the size of the space allocated to the user. Thus, when a user requests N bytes of memory, the library does not search for a free chunk of size N; rather, it searches for a free chunk of size N plus the size of the header.

3.Embedding A Free List

Assume we have a 4096-byte chunk of memory to manage (i.e., the heap is 4KB). To manage this as a free list, we first have to initialize said list; initially, the list should have one entry, of size 4096 (minus the header size).

![a free chunk](img/a free chunk.jpg)

When some memory is requested, it may look like:

![a heap after an allocation](img/a heap after an allocation.jpg)

4.Growing The Heap

Most traditional allocators start with a small-sized heap and then request more memory from the OS when they run out. To service the `sbrk` request, the OS finds free physical pages, maps them into the address space of the requesting process, and then returns the value of the end of  the new heap; at that point, a larger heap is available, and the request can be successfully serviced.

### Basic Strategies

1.Best Fit

First, search through the free list and find chunks of free memory that are as big or bigger than the requested size. Then, return the one that is the smallest in that group of candidates; this is the so called best-fit chunk (it could be called smallest fit too).

(1) Simple.

(2) Naive implementations pay a heavy performance penalty when performing an exhaustive search for the correct free block.

2.Worst Fit

Find the largest chunk and return the requested amount; keep the remaining (large) chunk on the free list.

(1) Worst fit tries to thus leave big chunks free instead of lots of small chunks that can arise from a best-fit approach.

(2) A full search of free space is required, and thus this approach can be costly. Worse, most studies show that it performs badly, leading to excess fragmentation while still having high overheads.

3.First Fit 

The first fit method simply finds the first block that is big enough and returns the requested amount to the user.

(1) No exhaustive search of all the free spaces are necessary.

(2) It sometimes pollutes the beginning of the free list with small objects. Thus, how the allocator manages the free list’s order becomes an issue. One approach is to use **address-based** ordering; by keeping the list ordered by the address of the free space, coalescing becomes easier, and fragmentation tends to be reduced.

4.Next Fit

Instead of always beginning the first-fit search at the beginning of the list, the next fit algorithm keeps an extra pointer to the location within the list where one was looking last.

(1) It searches for free space throughout the list more uniformly.

(2) An exhaustive search is once again avoided.

## Paging

Instead of splitting up a process’s address space into some number of variable-sized logical segments (e.g., code, heap, stack), we divide it into fixed-sized units, each of which we call a **page**. Correspondingly,we view physical memory as an array of fixed-sized slots called **page frames**; each of these frames can contain a single virtual-memory page.

### Overview

To record where each virtual page of the address space is placed in physical memory, the operating system usually keeps a **per-process** data structure known as a **page table**. The major role of the page table is to store address translations for each of the virtual pages of the address space, thus letting us know where in physical memory each  page resides. For example, the page table may have some entries (PTE): (Virtual Page 0 → Physical Frame 3), (VP 1→PF 7), (VP 2→PF 5), and (VP 3→PF 2).

![pages](img/pages.jpg)

In the example above, there are four pages and each of them is 16 bytes. because the virtual address space of the process is 64 bytes, we need 6 bits total for our virtual address (2^6^ = 64). The page size is 16 bytes in a 64-byte address space; thus we need to be able to select 4 pages, and the top 2 bits of the address do just that. Thus, we have a 2 bit **virtual page number (VPN)**. The remaining bits tell us which byte of the page we are interested in, 4 bits in this case; we call this the **offset**. In this way, we can translate a virtual address into physical address. 

### Where Are Page Table Stored?

Because page tables are too big,we don’t keep any special on-chip hardware in the MMU to store the page table of the currently-running process. Instead, we store the page table for each process in memory somewhere.

### What's Actually In The Page Table?

The simplest form of a page table is called a **linear page table**, which is just an array. The OS *indexes* the array by the virtual page number (VPN), and looks up the page-table entry (PTE) at that index in order to find the desired physical frame number (PFN).

As for the contents of each PTE, different bits may have different meanings. 

- A **valid bit** is common to indicate whether the particular translation is valid.
- **Protection bits** indicate whether the page could be read from, written to, or executed from.
- A **present bit** indicates whether this page is in physical memory or on disk (i.e., it has been swapped out).
- A **dirty bit** indicates whether the page has been modified since it was brought into memory.
- A **reference bit** (a.k.a. accessed bit) is sometimes used to track whether a page has been accessed, and is useful in determining which pages are popular and thus should be kept in memory

### Paging Is Also Too Slow

```pseudocode
// Extract the VPN from the virtual address
VPN = (VirtualAddress & VPN_MASK) >> SHIFt

// Form the address of the page-table entry (PTE)
PTEAddr = PTBR + (VPN + sizeof(PTE))

// Fecth the PTE
PTE = AccessMemory(PTEAddr)

// Check if process can access the page
if (PTE.Valid == FALSE)
	RaiseException(SEGMENTATION_FAULT)
else if (CanAccess(PTE.ProtectBits) == FALSE)
	RaiseException(PROTECTION_FAULT)
else 
	// Access if OK: form physsical address and fetch it
	offset = VirtualAddress & OFFSET_MASK
	PhysAddr = (PTE.PFN << PFN_SHIFt) | offset
	Register = AccessMemory(PhysAddr)
```

To summarize, we now describe the initial protocol for what happens on each memory reference. For every memory reference (whether an instruction fetch or an explicit load or store), paging requires us to perform one extra memory reference in order to first fetch the translation from the page table. That is a lot of work! Extra memory references are costly, and in this case will likely slow down the process by a factor of two or more.

## Translation-lookaside Buffer (TLB)

How can we speed up address translation, and generally avoid the extra memory reference that paging seems to require?

A **TLB** is part of the chip’s memory-management unit (MMU), and is simply a hardware cache of popular virtual-to-physical address translations; thus, a better name would be an address-translation **cache**.  Upon each virtual memory reference, the hardware first checks the TLB to see if the desired translation is held therein; if so, the translation is performed (quickly) without having to consult the page table.

### TLB Basic Algorithm

The algorithm the hardware follows works like this: first, extract the virtual page number (VPN) from the virtual address and check if the TLB holds the translation for this VPN (Line 2). If it does, we have a **TLB hit**, which means the TLB holds the translation.

If the CPU does not find the translation in the TLB (a **TLB miss**), we have some more work to do. The hardware accesses the page table to find the translation, and, assuming that the virtual memory reference generated by the process is valid and accessible , updates the TLB with the translation. Finally, once the TLB is updated, the hardware retries the instruction; this time, the translation is found in the TLB, and the memory reference is processed quickly.

It's out hope to have more TLB hits. Like any cache, TLBs rely upon both **spatial** and **temporal locality** for success, which are program properties. If the program of interest exhibits such locality (and many programs do), the TLB hit rate will likely be high.

### Who Handles The TLB Miss?

Some "older" architectures have hardware-managed TLBs. More modern architectures have what is known as a software-managed TLB. On a TLB miss, the hardware simply raises an exception, which pauses the current instruction stream, raises the privilege level to kernel mode, and jumps to a **trap handler**. This trap handler is code within the OS that is written with the express purpose of handling TLB misses. When run, the code will lookup the translation in the page table, use special “privileged” instructions to update the TLB, and return from the trap; at this point, the hardware retries the instruction (resulting in a TLB hit).

The primary advantage of the software-managed approach is *flexibility*: the OS can use any data structure it wants to implement the page table, without necessitating hardware change. Another advantage is *simplicity*; as you can see in the TLB control flow, the hardware doesn’t have to do much on a miss; it raises an exception, and the OS TLB miss handler does the rest.

*Important Details*

First, the return-from-trap instruction needs to be a little different than the return-from-trap which services a system call. In the latter case, the return-from-trap should resume execution at the instruction after the trap into the OS, just as a return from a procedure call returns to the instruction immediately following the call into the procedure. In the former case, when returning from a TLB miss-handling trap, the hardware must resume execution at the instruction that caused the trap; this retry thus lets the instruction run again, this time resulting in a TLB hit.

Second, when running the TLBmiss-handling code, the OS needs to be
extra careful not to cause an infinite chain of TLB misses to occur. Many
solutions exist; for example, you could keep TLB miss handlers in physical memory (where they are **unmapped** and not subject to address translation), or reserve some entries in the TLB for permanently-valid translations and use some of those permanent translation slots for

### TLB Contents

A typical TLB might have 32, 64, or 128 entries and be what is called fully associative. Basically, this just means that any given translation can be anywhere in the TLB, and that the hardware will search the entire TLB in parallel to find the desired translation. A TLB entry might look like this:

VPN | PFN | other bits

Note that both the VPN and PFN are present in each entry, as a translation could end up in any of these locations.

### TLB Issus: Context Switches 

When context-switching between processes, the translations in the TLB for the last process are not meaningful to the about-to-be-run process. What should the hardware or OS do in order to solve this problem?

One approach is to simply **flush** the TLB on context switches, thus emptying it before running the next process. However, there is a cost: each time a process runs, it must incur TLB misses as it touches its data and code pages. If the OS switches between processes frequently, this cost may be high.

To reduce this overhead, some systems add hardware support to enable sharing of the TLB across context switches. In particular, some hardware systems provide an **address space identifier** (**ASID**) field in the TLB. You can think of the ASID as a process identifier (PID), but usually it has fewer bits. Here is a depiction of a TLB with the added ASID field:

| VPN  | PFN  | valid | prot | ASID |
| ---- | ---- | ----- | ---- | ---- |
| 10   | 100  | 1     | rwx  | 1    |
| -    | -    | 0     | -    | -    |
| 10   | 170  | 1     | rwx  | 2    |
| -    | -    | 0     | -    | -    |

### Issue: Replacement Policy

Which TLB entry should be replaced when we add a new TLB entry? The goal, of course, being to minimize the miss rate (or increase hit rate) and thus improve performance.

1.LRU

One common approach is to evict the **least-recently-used** or LRU entry. LRU tries to take advantage of locality in the memory-reference stream, assuming it is likely that an entry that has not recently been used is a good candidate for eviction.

2.random

Such a policy is useful due to its simplicity and ability to avoid corner case behaviors; for example, a “reasonable” policy such as LRU behaves quite unreasonably when a program loops over n + 1 pages with a TLB of size n; in this case, LRU misses upon every access, whereas random does much better.

## Paging: Smaller Tables

Simple array-based page tables (usually called linear page tables) are too big, taking up far too much memory on typical systems. How can we make page tables smaller?

A quite simple approach is to use bigger pages. However, the major problem with this approach, however, is that big pages lead to waste within each page, a problem known as **internal fragmentation**. Our problem will not be solved so simply, alas.

### Hybrid Approach: Paging and Segments

Instead of having a single page table for the entire address space of the process, why not have one **per logical segment**? In this example, we might thus have three page tables, one for the code, heap, and stack parts of the address space. Now, remember with segmentation, we had a base register that told us where each segment lived in physical memory, and a bound or limit register that told us the size of said segment. In our hybrid, we still have those structures in the MMU; here, we use the base not to point to the segment itself but rather to hold the **physical address of the page table** of that segment. The bounds register is used to indicate the end of the page table (i.e., how many valid pages it has).

When a process is running, the base register for each of these segments  contains the physical address of a linear page table for that segment; thus, each process in the system now has **three** page tables associated with it.

*Disadvantages*

First, it still requires us to use segmentation. Second, this hybrid causes external fragmentation to arise again. 

### Multi-Level Page Tables

The basic idea behind a multi-level page table is simple. First, chop up the page table into page-sized units; then, if an entire page of page-table entries (PTEs) is invalid, don’t allocate that page of the page table at all. To track whether a page of the page table is valid (and if valid, where it is in memory), use a new structure, called the **page directory**. The page directory thus either can be used to tell you where a page of the page table is, or that the entire page of the page table contains no valid pages.

![linear and multi-level page tables](img/linear and multi-level page tables.jpg)

The page directory, in a simple two-level table, contains one entry per page of the page table. It consists of a number of **page directory entries** (PDE). A PDE (minimally) has a valid bit and a **page frame number** (PFN), similar to a PTE. However, as hinted at above, the meaning of this valid bit is slightly different: if the PDE entry is valid, it means that at least one of the pages of the **page table** that the entry points to (via the PFN) is valid, i.e., in at least one PTE on that page pointed to by this PDE, the valid bit in that PTE is set to one. If the PDE entry is not valid (i.e., equal to zero), the rest of the PDE is not defined.

*Advantages*

First, and perhaps most obviously, the multi-level table only allocates page-table space in proportion to the amount of address space you are using; thus it is generally compact and supports sparse address spaces.

Second, if carefully constructed, each portion of the page table fits neatly within a page, making it easier to manage memory; the OS can simply grab the next free page when it needs to allocate or grow a page table.

*Disadvantages*

1. On a TLB miss, two loads from memory will be required to get the right translation information from the page table (one for the page directory, and one for the PTE itself).
2. Whether it is the hardware or OS handling the page-table lookup (on a TLB miss), doing so is undoubtedly more involved than a simple linear page-table lookup.

### Inverted Page Tables

Instead of having many page tables (one per process of the system), we keep a single page table that has an entry for each physical page of the system. The entry tells us which process is using this page, and which virtual page of that process maps to this physical page.

Finding the correct entry is now a matter of searching through this data structure. A linear scan would be expensive, and thus a hash table is often built over the base structure to speed lookups.

### Swapping the Page Tables To Disk

Some systems place page tables in kernel virtual memory, thereby allowing the system to swap some of these page tables to disk when memory pressure gets a little tight.





