# Lecture 24

### Binary Buddy System

Assume: size of heap is power of 2

Example: Heap is 4k = 1024 words

```
+------------+
|            |
+----1024----+
```

Suppose program requests 20 words, we need 1 word for bookkeeping

Memory allocated in blocks of size $2^k$, so allocate 32 words

1024 is too big, so split into two heaps "buddies"

```
+---------------+
|       |       |
+--512------512-+
```

Allocate from one fo the 512-word buddies. Still too big, split again.

```
+-------+-------+--------+
|       |       |        |
+--256--+--256--+---512--+
again
+-------+-------+-------+--------+
|       |       |       |        |
+--128--+--128--+--256--+---512--+
until
+------+------+------+-------+-------+--------+
|alloc |      |      |       |       |        |
+--32--+--32--+--64--+--128--+--256--+---512--+
```

Suppose request 63 words, so we allocate 64. Heap contains a 64-word block

```
+------+------+------+-------+-------+--------+
|alloc |      |alloc |       |       |        |
+--32--+--32--+--64--+--128--+--256--+---512--+
```

Request 50 words, allocate 51, so allocate 64-word block. Need split 128-word block first.

```
+------+------+------+------+------+-------+--------+
|alloc |      |alloc |alloc |      |       |        |
+--32--+--32--+--64--+--64--+--64--+--256--+---512--+
```

First 64-word block is released

```
+------+------+------+------+------+-------+--------+
|alloc |      |      |alloc |      |       |        |
+--32--+--32--+--64--+--64--+--64--+--256--+---512--+
```

32-word block is released

```
+------+------+------+------+------+-------+--------+
|      |      |      |alloc |      |       |        |
+--32--+--32--+--64--+--64--+--64--+--256--+---512--+
```

Buddy of the 32-word block is also free, so merge

```
+-------+------+------+-------+--------+
|       |alloc |      |       |        |
+--128--+--64--+--64--+--256--+---512--+
```

If 64-word block released

```
+-------+------+------+-------+--------+
|       |      |      |       |        |
+--128--+--64--+--64--+--256--+---512--+
4 merges
+------------+
|            |
+----1024----+
```

#### Deallocation info

``` 
----+
    |
    V
+------------+
|  |         |
+------------+
```

alloc.asm assigns each block a code.

```
entire heap 
+------------+
|      1     |
+----1024----+

+----------------+
|   10   |   11  |
+--512------512--+

+-------+-------+-------+--------+
|  1010 |  1011 |  101  |    11  |
+--128--+--128--+--256--+---512--+
```

To find your buddy's code, flip the last bit.

Merging buddies -- drop the last bit

## Garbage Collection

System reclaims memory once client can no longer access it

### Mark and Seep

* scan the entire stack, looking for pointers
* for each pointer found, mark the heap block it is pointing to
* if the heap block containing pointers, follow those as well, mark, etc.
* scan the heap, reclaim any blocks not marked, and clear all marks.

### Reference Counting

For each heap block, heap track of the number of pointer that point to it (it is reference count)

You must watch every pointer and update reference pointers (decrement old, increment new) each time a pointer is re-assigned)

If a block is reference count reaches 0, reclaim


Problem: circular references

```
 +---------------+
 |               |
 v               |
+----+--+  +----+--+
|    |  |  |    |  |
+----+--+  +----+--+
       |    ^
	   |    |
       +----+
```

But have reference count=1, but collectively inaccessible.

### Copying Garbage Collection

Heap is in two halves: $from$ and $to$

* allocate only from $from$
* when $from$ fills up, all reachable data is copied from $from$ to $to$
* Their role of $from$ and $to$ is reversed

Built-in compaction, guaranteed that after the swap, all reachable data occupies contiguous memory, no fragmentation.

But can only use half the heap at a time

完结撒花ヽ(*≧ω≦)/~
