# Enhancing xv6 File System

## CS344 Operating Systems Lab — Assignment 5

This project extends the xv6 file system with two features:

1. Support for **large files using doubly-indirect blocks**
2. Support for **symbolic links (soft links)**

### Group 22

- Sunil — 230101099
- Subham — 230101098
- Deepit Gupta — 230101033
- Raghav Goyal — 230101084

---

## 1. Large File Support

The original xv6 file system uses direct and single-indirect block pointers, which limits the maximum file size.

We added a **doubly-indirect block pointer** to allow xv6 to store much larger files.

### Changes

- Reduced `NDIRECT` from 12 to 11.
- Added one doubly-indirect pointer.
- Updated `MAXFILE` to support **65,803 blocks (~64.3 MiB)**.
- Kept the on-disk inode size at **64 bytes**.
- Updated `bmap()` to allocate and access doubly-indirect blocks.
- Updated `itrunc()` to correctly free doubly-indirect blocks.
- Increased `FSSIZE` to 20,000 blocks for testing.

### Modified Files

```text
kernel/fs.h
kernel/file.h
kernel/fs.c
kernel/param.h
user/bigfile.c
````

### How It Works

The block addressing scheme is:

```text
Inode
 ├── Direct blocks
 ├── Single-indirect block
 └── Doubly-indirect block
       └── Indirect blocks
             └── Data blocks
```

The doubly-indirect pointer first points to an indirect block, which then points to the actual data blocks.

### Testing

`bigfile.c` creates a **5 MiB file** by writing 5,120 blocks and then reads the entire file back to verify the data.

Run:

```bash
make qemu
```

Inside xv6:

```text
$ bigfile
```

Expected result:

```text
bigfile: OK
```

The test successfully writes and verifies all 5,120 blocks. 

---

## 2. Symbolic Links

The second part adds support for **symbolic links** in xv6.

A symbolic link stores the path of another file or directory.

For example:

```text
testlink -> testfile
```

Opening `testlink` normally accesses `testfile`.

### Changes

* Added `T_SYMLINK` inode type.
* Added `O_NOFOLLOW` flag.
* Added the `symlink()` system call.
* Modified `open()` to automatically follow symbolic links.
* Added protection against circular links.
* Modified `stat()` to report information about the link itself.

Symbolic-link resolution is limited to **10 levels** to prevent infinite loops. 

### Modified Files

```text
kernel/stat.h
kernel/fcntl.h
kernel/syscall.h
kernel/sysfile.c
user/usys.pl
user/user.h
user/ulib.c
user/symlinktest.c
```

### Example

```c
symlink("testfile", "testlink");
```

This creates:

```text
testlink -> testfile
```

Normal `open()` follows the link:

```c
open("testlink", O_RDONLY);
```

Using `O_NOFOLLOW` opens the link itself:

```c
open("testlink", O_RDONLY | O_NOFOLLOW);
```

---

## 3. Testing

A test program named `symlinktest.c` checks:

* Basic symbolic links
* `O_NOFOLLOW`
* Chained symbolic links
* Symbolic-link loop detection
* Unlinking a symbolic link
* Symbolic links to directories

Run:

```bash
make qemu
```

Then inside xv6:

```text
$ symlinktest
```

Expected result:

```text
symlinktest: all tests passed
```

All six test cases passed successfully in the implementation. 

---

## 4. Summary

This project extends xv6 with:

* Larger file support using doubly-indirect blocks.
* A maximum file size of approximately **64.3 MiB**.
* Correct allocation and deallocation of doubly-indirect blocks.
* Symbolic-link creation and resolution.
* `O_NOFOLLOW` support.
* Chained symbolic links.
* Circular-link detection.
* Correct symbolic-link unlinking.



