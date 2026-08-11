# XV6 Operating System - Scheduling & System Calls (OS Lab Assignment 2)

**Group Members**
- Sunil (230101099)
- Subham (230101098)
- Deepit Gupta (230101033)
- Raghav Goyal (230101084)

---

## 📌 Overview

This project implements customized process scheduling algorithms and essential system calls in the **XV6 RISC-V Operating System**. The project is split into two primary tasks:

1. **Task 1.1: Weighted Round-Robin (WRR) Scheduling & Priority System Calls**
   - Implemented system calls: `get_priority()`, `set_priority()`, and `get_rtime()`.
   - Modified XV6's default Round-Robin scheduler to a **Weighted Round-Robin** scheduler based on process priority time slices.
2. **Task 1.2: Shortest Job First (SJF) Non-Preemptive Scheduling**
   - Implemented non-preemptive **Shortest Job First (SJF)** scheduling based on estimated CPU burst times (`est_burst` / `remaining`).
   - Supports dynamic process arrivals at runtime.

---

## 🛠️ System Calls Implementation

### System Calls Created / Modified

| System Call | Prototype | Description |
| :--- | :--- | :--- |
| `set_priority` | `int set_priority(int n)` | Sets the priority of the calling process to `n` (bounded by `MAX_PRIORITY`). |
| `get_priority` | `int get_priority(void)` | Returns the current priority of the calling process. |
| `get_rtime` | `int get_rtime(void)` | Returns the total running time (`rtime`) of the calling process. |
| `top` | `int top(void)` | Display process status/metrics (optional/utility). |
| `setburst` | `int setburst(int b)` | Sets the estimated CPU burst time for SJF scheduling. |

### File Modifications for System Calls

1. **`user/user.h`**: Declared user-level system call prototypes (`set_priority`, `get_priority`, `get_rtime`, `top`).
2. **`user/usys.pl`**: Added stubs for entry generation (`top`, `set_priority`, `get_priority`, `get_rtime`).
3. **`kernel/syscall.h`**: Defined system call numbers:
   ```c
   #define SYS_top          23
   #define SYS_set_priority 24
   #define SYS_get_priority 25
   #define SYS_get_rtime    26



4. **`kernel/syscall.c`**: Added function pointer handlers and array mappings for `[SYS_set_priority]`, `[SYS_get_priority]`, `[SYS_get_rtime]`, and `[SYS_top]`.
5. **`kernel/sysproc.c`**: Implemented `sys_set_priority()`, `sys_get_priority()`, and `sys_get_rtime()` to read and update `proc` struct attributes under lock protection (`p->lock`).

---

## ⚙️ Task 1.1: Weighted Round-Robin (WRR) Scheduler

### Concept

Processes are assigned a `slice` equal to their `priority`. When scheduled, a process consumes its time slice across timer interrupts. When its `slice` hits `0`, it yields to the next runnable process. Slice refill occurs automatically when all processes consume their assigned quotas. This eliminates process starvation while giving higher-priority processes proportionally more CPU execution time.

### Implementation Details

1. **`kernel/proc.h` (`struct proc`)**:
```c
int priority;  // Process priority
int slice;     // Remaining time slice (ticks)
int rtime;     // Total running time

#define MAX_PRIORITY 1000
#define DEFAULT_PRIORITY 1

```


2. **`kernel/proc.c` (`allocproc`)**:
Initializes default parameters upon creation:
```c
p->priority = DEFAULT_PRIORITY;
p->slice = p->priority;
p->rtime = 0;

```


3. **`kernel/proc.c` (`scheduler`)**:
* Tracks the `last` scheduled process index to ensure proper Round-Robin rotation.
* Refills `p->slice = p->priority` if `p->slice == 0`.
* Dispatches processes with remaining slices (`p->slice > 0`).


4. **`kernel/trap.c` (`usertrap`)**:
Decrements process slice count on timer interrupts:
```c
if (which_dev == 2) {
    p->slice--;
    yield();
}

```



### Makefile Setup

To test scheduling on a single CPU core without multi-core distribution:

```makefile
ifndef CPUS
CPUS := 1
endif

```

---

## ⏱️ Task 1.2: Shortest Job First (SJF) Scheduler

### Concept

A non-preemptive scheduling strategy chosen to minimize average process turnaround time. The scheduler scans available `RUNNABLE` processes and selects the one with the smallest remaining CPU time (`remaining`).

### Implementation Details

1. **`kernel/proc.h` (`struct proc`)**:
```c
int est_burst;  // Estimated total CPU burst (set by user or default)
int remaining;  // Remaining CPU time (initialized to est_burst)

```


2. **`kernel/proc.c` (`scheduler`)**:
* Searches all runnable processes to find the one with the minimum `p->remaining`.
* Executes the selected process through context switching without interrupting it early for another task.


3. **`kernel/trap.c` & `kernel/clockintr**`:
* **Non-Preemption:** Disabled `yield()` calls inside `usertrap()` and `kerneltrap()` for timer interrupts (`which_dev == 2`).
* **Tracking Execution:** Decrements `p->remaining` and increments `p->rtime` inside `clockintr()`:
```c
if (p && p->state == RUNNING) {
    p->rtime++;
    if (p->remaining > 0)
        p->remaining--;
}

```





---

## 🧪 Test Programs & Verification

### Added Test Programs in Makefile

```makefile
$U/_arrival\
$U/_rrtest\
$U/_priotest\

```

### 1. Weighted Round-Robin Test (`rrtest.c`)

Forks multiple child processes with different priorities (`2, 4, 8`) running CPU-intensive loops.

**Observed Output Behavior:**

* Priority 2 process runs for **2 slices** $\rightarrow$ yields.
* Priority 4 process runs for **4 slices** $\rightarrow$ yields.
* Priority 8 process runs for **8 slices** $\rightarrow$ yields.
* Higher priority tasks complete earlier relative to their required compute time.

---

### 2. SJF Arrival Test (`arrival.c`)

Tests dynamic runtime arrivals for non-preemptive SJF execution.

#### **Command:**

```bash
$ arrival 90 0 40 10 5 3

```

*(Runs 3 processes: Burst 90 at delay 0, Burst 40 at delay 10, Burst 5 at delay 3)*

#### **Output Execution Trace:**

```text
xv6 kernel is booting
init: starting sh
$ arrival 90 0 40 10 5 3
pid 4: est=90 delay=0 start=368
pid 4: end=458 run=90
pid 6: est=5  delay=3 start=458
pid 6: end=463 run=5
pid 5: est=40 delay=10 start=463
pid 5: end=503 run=40
$

```

#### **Analysis of Output:**

1. At tick `0`, **PID 4** (`est=90`) starts execution as no other process is present.
2. While **PID 4** is running, **PID 6** (`est=5`) arrives at delay `3` and **PID 5** (`est=40`) arrives at delay `10`.
3. Because SJF is non-preemptive, **PID 4** runs uninterrupted until completion (duration 90 ticks, ending at tick `458`).
4. At tick `458`, the scheduler compares remaining jobs (**PID 6** with burst `5` vs **PID 5** with burst `40`).
5. The scheduler picks **PID 6** (shortest job), which completes in 5 ticks (`458` to `463`).
6. Finally, **PID 5** runs to completion (`463` to `503`).

---

## 🚀 Building & Running in XV6

### Prerequisites

* RISC-V GNU Toolchain (`riscv64-unknown-elf-` or `riscv64-linux-gnu-`)
* QEMU Emulator (`qemu-system-riscv64`)

### Compilation Steps

1. **Clean prior builds:**
```bash
make clean

```


2. **Compile and launch XV6 under QEMU:**
```bash
make qemu

```


3. **Run Weighted Round-Robin Test:**
```bash
$ rrtest

```


4. **Run Shortest Job First Arrival Test:**
```bash
$ arrival 90 0 40 10 5 3

```
