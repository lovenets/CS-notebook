# Virtualization 

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

$$T_{response}=T_{firstrun}-T_{arrival}$$

While great for turnaround time, STCF is quite bad for response time and interactivity.  

**Round Robin**

Instead of running jobs to completion, RR runs a job for a time slice (sometimes called a **scheduling quantum**) and then switches to the next job in the run queue. It repeatedly does so until the jobs are finished. For this reason, RR is sometimes called **time-slicing**. 

Note that the length of a time slice must be a multiple of the timer-interrupt period.

The shorter a time slice is, the better the performance of RR under the response-time metric. However, making the time slice too short is problematic: suddenly the cost of context switching will dominate overall performance. 

### Overlap 

A scheduler clearly has a decision to make when a job initiates an I/O
request, because the currently-running job won’t be using the CPU during the I/O; it is blocked waiting for I/O completion.  

When taking I/O into account, **overlap** is better. CPU is used by one process while waiting for the I/O of another process to complete. By treating each CPU burst as a job, the scheduler makes sure processes that are “interactive” get run frequently. While those interactive jobs are performing I/O, other CPU-intensive jobs run, thus better utilizing the processor. 