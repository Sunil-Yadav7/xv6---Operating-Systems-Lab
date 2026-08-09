# OS Lab Assignment 1

## Group Members

- Sunil (230101099)
- Subham (230101098)
- Deepit Gupta (230101033)
- Raghav Goyal (230101084)

---

# Overview

This assignment consists of two tasks implemented in the xv6 operating system.

- **Task 1.1:** Prime PID Allocation
- **Task 1.2:** Custom `top` System Call

---

# Task 1.1 – Prime PID Allocation

## Objective

Modify the xv6 kernel so that newly created user processes are assigned prime-numbered Process IDs (PIDs).

---

## Implementation

### Step 1

Introduced an array named `primes` inside `kernel/proc.c` to keep track of allocated prime PIDs.

```c
int primes[1000];
```

---

### Step 2

Implemented a helper function to check whether a number is prime.

```c
int isprime(int x)
```

The function iterates from `2` to `sqrt(x)` and returns whether the number is prime.

---

### Step 3

Modified the `allocpid()` function in `proc.c`.

Logic:

- Reserve existing kernel PIDs.
- Search for the next available prime number.
- Mark it as allocated.
- Assign it as the PID of the new process.

A small special case was added for the initial kernel processes because they are created before the user test program executes.

---

### Step 4

Created a user program

```
primepidtest.c
```

The program creates multiple child processes using `fork()` and prints their assigned PIDs.

Example:

```
Child 1 : PID = 2
Child 2 : PID = 3
Child 3 : PID = 5
Child 4 : PID = 7
Child 5 : PID = 11
```

---

### Step 5

Modified the xv6 Makefile.

Added the test program to the `UPROGS` section.

Example:

```make
UPROGS=\
    ...
    $U/_primepidtest\
```

---

## Output

Running the test program confirms that child processes receive prime-numbered PIDs.

Example output:

```
Child 1 : PID = 2
Child 2 : PID = 3
Child 3 : PID = 5
Child 4 : PID = 7
Child 5 : PID = 11
```

---

# Task 1.2 – Custom `top` System Call

## Objective

Implement a custom system call similar to the Linux `top` command to display active processes along with their execution time (ticks).

---

## Implementation Steps

### Step 1

Assigned a new system call number inside

```
kernel/syscall.h
```

Example:

```c
#define SYS_top 23
```

---

### Step 2

Updated

```
kernel/syscall.c
```

Added

```c
extern uint64 sys_top(void);
```

and registered the system call inside the syscall table.

---

### Step 3

Declared the user-space prototype.

Modified

```
user/user.h
```

```c
int top(void);
```

---

### Step 4

Added the user stub.

Modified

```
user/usys.pl
```

```perl
entry("top");
```

After generating the assembly wrappers, `usys.S` includes the corresponding syscall stub.

---

### Step 5

Added a runtime counter for every process.

Modified

```
kernel/proc.h
```

Added

```c
int rtime;
```

inside

```c
struct proc
```

to maintain the execution ticks of each process.

---

### Step 6

Initialized the runtime counter.

Modified

```
allocproc()
```

inside

```
kernel/proc.c
```

```c
p->rtime = 0;
```

---

### Step 7

Updated the scheduler.

Each time a process runs, increment

```c
p->rtime++;
```

This keeps track of CPU execution time in ticks.

---

### Step 8

Implemented

```
sys_top()
```

inside

```
kernel/sysproc.c
```

The function iterates over all active processes and prints:

- PID
- Process State
- Process Name
- Runtime Ticks

Example output format:

```
PID     State       Name        Ticks
1       SLEEPING    init        25
2       RUNNING     sh          18
3       RUNNING     top         44
```

---

### Step 9

Updated the Makefile.

Added

```make
$U/_top\
```

to the `UPROGS` section.

---

### Step 10

Created a user-space testing program.

Example:

```c
int main() {
    for(int i = 0; i < 5; i++) {
        top();
        sleep(10);
    }
    exit(0);
}
```

The program periodically invokes the `top` system call and displays updated runtime statistics.

---

# Files Modified

## Task 1.1

```
kernel/proc.c
user/primepidtest.c
Makefile
```

---

## Task 1.2

```
kernel/syscall.h
kernel/syscall.c
kernel/sysproc.c
kernel/proc.c
kernel/proc.h
user/user.h
user/usys.pl
Makefile
user/top.c
```

---

# Features Implemented

## Prime PID Assignment

- Prime-number PID allocation
- Prime number tracking
- Custom PID generation
- User-space testing program

## Custom Top Command

- New system call
- Runtime tick tracking
- Process information display
- User-space command support

---

# Sample Output

## Prime PID Test

```
Child 1 : PID = 2
Child 2 : PID = 3
Child 3 : PID = 5
Child 4 : PID = 7
Child 5 : PID = 11
```

---

## Top Command

```
PID     State       Name        Ticks
1       SLEEPING    init        25
2       SLEEPING    sh          18
3       RUNNING     top         44
```

---

# Conclusion

This assignment demonstrates kernel-level modifications in xv6 by implementing two independent features:

- Prime-number-based process ID allocation.
- A custom `top` system call for monitoring process execution time.

These tasks provide practical experience with process management, kernel data structures, system call implementation, scheduler modification, and user-kernel interaction.