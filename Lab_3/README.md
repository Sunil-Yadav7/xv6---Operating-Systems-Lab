# OS Lab Assignment 3

## Shared Memory and Mailbox Implementation in xv6

Implementation of **Inter-Process Communication (IPC)** primitives in the xv6 operating system, including **Shared Memory** and **Mailbox-based Message Passing**, along with a concurrent application demonstrating synchronized communication between processes.

---

## Group Members

| Name | Roll Number |
|------|-------------|
| Sunil | 230101099 |
| Subham | 230101098 |
| Deepit Gupta | 230101033 |
| Raghav Goyal | 230101084 |

---

# Overview

This assignment consists of two major tasks.

## Task 1

Implementation of kernel primitives for Inter-Process Communication (IPC).

The following IPC mechanisms were implemented:

- Shared Memory
- Mailboxes
- New xv6 system calls
- Shared memory cleanup during `exit()` and `exec()`
- Blocking mailbox communication

---

## Task 2

Implementation of a concurrent application that uses the IPC primitives developed in Task 1.

Two cooperating processes navigate an intertwined path stored inside shared memory while communicating exclusively using mailboxes.

---

# Task 1 — Kernel IPC Primitives

## Objective

The default xv6 operating system provides very limited support for inter-process communication.

This task extends xv6 by implementing two important IPC mechanisms:

- Shared Memory
- Mailboxes

These primitives enable efficient data sharing and reliable message passing between processes. :contentReference[oaicite:0]{index=0}

---

# System Call Changes

To integrate the IPC mechanisms into xv6, the following kernel modifications were performed:

- Added helper function declarations in `defs.h`
- Assigned new system call numbers in `syscall.h`
- Registered new entries in `syscall.c`
- Implemented syscall handlers in `sysproc.c`
- Added user-space declarations in `user.h`
- Generated user stubs using `usys.pl` :contentReference[oaicite:1]{index=1}

---

# Shared Memory

## Data Structure

A shared memory region maintains:

```c
struct shm_region {
    int used;
    int key;
    char *pa;
    int refcnt;
};
````

Each field serves the following purpose:

* `used` – Indicates whether the slot is allocated.
* `key` – User-visible identifier.
* `pa` – Physical page backing the shared region.
* `refcnt` – Number of attached processes.

A global shared memory table contains:

* Spinlock for synchronization
* Array of shared memory regions
* Initialization flag 

---

## Shared Memory Workflow

The implementation follows these steps:

1. Initialize shared memory subsystem.
2. Acquire the global lock.
3. Search for an existing key.
4. If found, return its slot.
5. Otherwise:

   * Find a free slot.
   * Allocate one physical page.
   * Initialize memory with zeros.
   * Store metadata.
6. Return the slot index. 

---

## Shared Memory Cleanup

During shared memory detachment:

* Virtual mapping is removed.
* Reference count is decremented.
* If no process is attached:

  * Free physical page.
  * Mark slot as unused. 

---

## Exit and Exec Handling

Cleanup handlers were implemented to safely remove shared memory mappings when a process executes:

* `exit()`
* `exec()`

This prevents xv6 kernel panics caused by stale mappings. 

---

# Mailbox Implementation

A mailbox structure was implemented using locks and a circular buffer.

The implementation includes:

* Mailbox initialization
* Mailbox lookup
* Free slot search
* Mailbox creation
* Blocking send
* Blocking receive

Communication follows FIFO ordering using a ring buffer. 

---

# Test Program

A testing program named:

```
shmmbox_test
```

was developed to validate both IPC mechanisms.

The test verifies:

* Shared memory updates are visible across processes.
* Mailboxes correctly implement FIFO communication.
* Blocking behavior works correctly. 

---

# Files Modified

### Kernel Files

```
kernel/defs.h
kernel/syscall.h
kernel/syscall.c
kernel/sysproc.c
kernel/shm.c
kernel/mbox.c
```

### User Files

```
user/user.h
user/usys.pl
user/shmmbox_test.c
```

---

# Task 2 — The Intertwined Memory Challenge

## Objective

Task 2 demonstrates the use of the IPC primitives by building a concurrent application where two independent processes navigate an intertwined path stored inside shared memory.

Neither process knows its own path.

Instead, each process depends entirely on the information received from the other process through mailboxes. 

---

# Master Program

The master program performs the following operations:

* Creates and maps shared memory.
* Initializes shared arrays:

  * `a2b`
  * `b2a`
  * `startA`
  * `startB`
* Builds intertwined paths.
* Creates two mailboxes.
* Launches two child processes.
* Waits for both children.
* Cleans shared memory.
* Prints completion message. 

---

# Process Program

Each process:

* Attaches to shared memory.
* Connects to both mailboxes.
* Exchanges its current position.
* Computes the next move using the teammate's position.
* Prints progress.
* Uses an END/ACK protocol for clean termination. 

---

# Deadlock Avoidance

## Problem

If both processes attempt to receive before sending, they enter a circular wait, resulting in deadlock. 

---

## Solution

The protocol assigns asymmetric roles.

### Process A

```
Send
↓

Receive
↓

Compute next move
```

### Process B

```
Receive
↓

Compute next move
↓

Send
```

This guarantees that every send operation has a matching receive operation, eliminating circular wait conditions. 

---

# Termination Protocol

Two-way END message exchange guarantees clean termination.

### If Process A finishes first

```
A → END → B

B → END → A

Both Exit
```

### If Process B finishes first

```
Receive END

↓

Reply END

↓

Exit
```

This ensures that neither process remains blocked waiting for a message. 

---

# Features Implemented

## Shared Memory

* Dynamic shared memory creation
* Shared page mapping
* Reference counting
* Cleanup during exit
* Cleanup during exec
* Shared memory attachment and detachment

---

## Mailboxes

* Mailbox creation
* Blocking send
* Blocking receive
* FIFO message ordering
* Circular queue implementation
* Thread-safe synchronization

---

## Concurrent Application

* Shared memory communication
* Mailbox synchronization
* Inter-process coordination
* Deadlock-free communication protocol
* Graceful process termination

---

# Project Structure

```
.
├── kernel
│   ├── defs.h
│   ├── syscall.c
│   ├── syscall.h
│   ├── sysproc.c
│   ├── shm.c
│   └── mbox.c
│
├── user
│   ├── user.h
│   ├── usys.pl
│   ├── shmmbox_test.c
│   ├── master.c
│   └── process.c
│
└── Makefile
```

---

# Expected Output

## Shared Memory Test

```
$ shmmbox_test

Child sees 77
Parent now sees 99
Child got 100
Child got 101
Child got 102
Child got 103
Child got 104
```

---

## Mailbox Test

```
$ mbox_ping

ping 0 -> pong 1
ping 1 -> pong 2
ping 2 -> pong 3
```

---

## Intertwined Memory Challenge

```
$ master

Process B: at 0 -> next 4
Process A: at 0 -> next 3
...

Process B reached END
Process A reached END

Master: both processes finished
```

---

# Learning Outcomes

Through this assignment, the following operating system concepts were implemented and explored:

* Inter-Process Communication (IPC)
* Shared Memory
* Message Passing
* xv6 System Call Development
* Kernel Synchronization
* Blocking Communication
* Circular Buffers
* Reference Counting
* Concurrent Programming
* Deadlock Prevention
* Process Synchronization
* Resource Cleanup

---

# References

1. R. Cox, F. Kaashoek, R. Morris. **xv6: A Simple, Unix-like Teaching Operating System.**
2. Abraham Silberschatz, Peter Galvin, Greg Gagne. **Operating System Concepts.** 

