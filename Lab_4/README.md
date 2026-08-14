# Advanced Virtual Memory Management in xv6


## Team

**Group 22 — CS334 Operating Systems Lab**

* Sunil — `230101099`
* Subham — `230101098`
* Deepit Gupta — `230101033`
* Raghav Goyal — `230101084`

## Overview

This project extends the **xv6-riscv operating system** with two important virtual memory features:

1. **Copy-on-Write (COW) Fork**
2. **MRU Paging and Swapping**

The main goal is to improve memory usage by sharing pages between processes and by moving pages to swap space when physical memory becomes full.

---

## Features

### 1. Copy-on-Write (COW)

Normally, `fork()` creates a complete copy of the parent's memory. This can waste memory when the child does not modify most of the pages.

With COW:

* Parent and child initially share the same physical pages.
* Shared pages are marked **read-only** and given a COW flag.
* A reference counter keeps track of how many processes use each physical page.
* When either process tries to write to a shared page, a **page fault** occurs.
* The kernel creates a new copy of the page for that process.
* The other process continues using the original page.

Example:

```text
             fork()
               |
        +------+------+
        |             |
     Parent         Child
        |             |
        +------v------+
           Shared Page
          refcount = 2
                |
          Child writes
                |
           Page Fault
                |
          Create Copy
          /          \
     Parent          Child
   Original Page    New Page
```

This reduces unnecessary memory copying during `fork()`.

---

### 2. Physical Page Reference Counting

Reference counting is added to safely manage shared physical pages.

The main helper functions are:

```c
incref(pa);
decref(pa);
getref(pa);
```

* `incref()` increases the reference count.
* `decref()` decreases the reference count.
* `getref()` returns the current reference count.

A page is freed only when its reference count becomes zero.

---

### 3. MRU Paging

The project also implements **Most Recently Used (MRU)** page eviction.

The kernel maintains a doubly linked list of recently accessed user pages.

```text
MRU Head
   |
   v
Page A -> Page B -> Page C -> Page D
```

The page at the head represents the most recently used page.

When physical memory is full, the system can select the MRU page for eviction.

The MRU list is updated whenever user pages are accessed.

---

### 4. Swapping

When a physical page needs to be evicted, its contents are written to a per-process swap file:

```text
/swap.<pid>
```

The physical page can then be reused.

If the process accesses the swapped-out page later:

1. A page fault occurs.
2. The kernel detects that the page is in swap.
3. A physical page is allocated.
4. The data is read back from the swap file.
5. The page is mapped again.
6. The MRU information is updated.

This provides basic **demand paging**.

---

## Paging Statistics

The implementation keeps track of:

* Page faults
* Number of swap-ins
* Number of swap-outs

A system call `getpagestat()` allows user programs to read these statistics.

Another system call, `dumpmru()`, can be used to display the current MRU list for debugging.

---

## Main Files

| File        | Description                                    |
| ----------- | ---------------------------------------------- |
| `riscv.h`   | Defines the COW page-table flag                |
| `defs.h`    | Declares COW and reference-count functions     |
| `kalloc.c`  | Implements physical page reference counting    |
| `vm.c`      | Handles COW, paging, eviction, and page faults |
| `trap.c`    | Handles COW write faults                       |
| `paging.h`  | Defines paging and swap structures             |
| `mru.c`     | Implements the MRU tracker                     |
| `swap.c`    | Implements swap-in and swap-out                |
| `sysproc.c` | Provides paging-related system calls           |
| `cowtest.c` | Tests Copy-on-Write                            |
| `mrumem.c`  | Tests MRU paging and swapping                  |

---

## Testing

### COW Test

Run the COW test from the xv6 shell:

```text
$ cowtest
```

The test verifies that:

* Parent and child initially share a physical page.
* The reference count increases after `fork()`.
* A child write causes a page fault.
* A new page is created for the child.
* The parent's original data remains unchanged.

Expected result:

```text
cowtest: ok
```

---

### MRU and Swap Test

Run:

```text
$ mrumem
```

The program allocates many pages to create memory pressure and trigger eviction and swapping.

It verifies:

* MRU page selection.
* Swap-out operations.
* Swap-in operations.
* Page-fault handling.
* Preservation of page contents.

Example output:

```text
[evict] ... OK:MRU
```

and:

```text
VERIFY: OK
```

`OK:MRU` confirms that the selected page matches the MRU page, while `VERIFY: OK` confirms that data remains correct after swap-out and swap-in.

---

## Result

The project successfully implements:

* Copy-on-Write `fork()`
* Physical page reference counting
* COW page-fault handling
* MRU page tracking
* MRU-based page eviction
* Per-process swapping
* Swap-in and swap-out
* Paging statistics
* Data-integrity verification

These features demonstrate how an operating system can efficiently manage physical memory while supporting process sharing and paging.


