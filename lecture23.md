# Lecture 23

## Overloading

What would happen if we wanted to compile

```c++
int f(int a) { ... }
int f(int a, int* b) { ... }
```

Get duplicated labels for $f$

How do we fix this?

### Name mangling

Encode the types of parameters as part of the label.

#### Example Convention:

```
F+type info + _ + name
```

```c++
int f() { ... } //F_f:
int f(int a) { ... } //Fi_f:
int f(int a, int b) { ... } //Fip_f:
```

C++ compilers will do this because C++ has overloading.

No standard mangling convention, all compilers different

* makes it hard or impossible to link code from different compilers
* this is by design because compilers differ in other aspects of their calling conventions, do not want to link successfully if conventions do not truly match.

C does not have overloading (no mangling)

* C and C++ code call each other routinely.
* How is this done?
* Need to suppress manging in C++

Call C from C++

```c++
extern "C" int f(int n); // Tells C++ f will not be mangled.
```

Call C++ from C, need to tell C++ not to mangle the functions

```c++
extern "C" int g(int x) { ... } //Do not mangle g
```

or

```
extern "C" {
    ...
    ...
    ...
}
```

## Memory Management and the Heap

WLP4, C, C++ 

* explicit memory management
* user must free own data via free/delete.

Java, Scheme

* implicit management
* garbage collection.

How do new/delete or malloc/free work? -- variety of implementation

1) List of free blocks:
    * maintain a linked list of pointers to blocks of free RAM
    * Initially entire heap is free, the list contains one entry.
    
* Suppose heap is 1K

```
      +-----------------------+
free->|1024|entire heap (1020)|
      +---------1024----------+
```

* Suppose 16 bytes allocated.
    * actually allocate 20 bytes: 16 bytes + 1 int(4 bytes)
    * return pointer to 2nd word.
    * store size just before returned pointer
    
```
+---+----------------+
| 20|16 bytes memory|
+-4-+-------16-------+
     ^
pointer returned
```

* free list contains the rest of the heap

```
      +-----------------------+
free->|1004|entire heap (1000)|
      +---------1004----------+
```

* Now suppose 28 bytes allocated - realy allocated 32:

```
+---+----------------+  +---+----------------+        +-----------------------+
| 20|16 bytes memmory|  | 32|28 bytes memmory|  free->|972|entire heap (968)  |
+-4-+-------16-------+  +-4-+-------16-------+        +----------972----------+
```

* Suppose first block is freed (20 bytes) (delete p)
    * free/delete checks p[-1] to see how much memory com back, adds to free list
    
```
            __________________________
           |                          |
           |                          V
      +---+----------------+        +-----------------------+
free->| 20|16 bytes memory|        |972|entire heap (968)  |
      +-4-+-------16-------+        +----------972----------+
```

* Suppose the 32-byte block is freed:

```
            ____________________    ____________________
           |                    |  |                    |
           |                    V  |                    V       
      +---+----------------+  +---+----------------+  +-----------------------+
free->| 20|16 bytes memory|  | 32|28 bytes memory|  |972|entire heap (968)  |
      +-4-+-------16-------+  +-4-+-------16-------+  +----------972----------+
```

notice: $p+*p == q$ thus contiguous, so merge
    
```
            __________________________
           |                          |
           |                          V
      +---+----------------+        +-----------------------+
free->| 52|48 bytes memory|        |972|entire heap (968)  |
      +-4-+-------48-------+        +----------972----------+
      
      +-----------------------+
free->|1024|entire heap (1020)|
      +---------1024----------+      
```

Note: repeated allocation and deallocations creates "holes" in heap

```
alloc 20
+--+---------------------+
|20|                     |
+--+---------------------+
alloc 40
+--+---------------------+
|20|40|                  |
+--+---------------------+
free 20
+--+---------------------+
|  |40|                  |
+20+---------------------+
alloca 5
+-+--+-------------------+
|5|  |40|                |
+-+15+-------------------+
etc...
```

Call fragmentation means even if $n$ bytes are free, you may not be able to allocate $n$ bytes.

To reduce fragmentation:

* do not always pick the first block of RAM big enough to satisfy the request.

```
+-+--+-+--+-+---+
| |20| |15| |100|
+-+--+-+--+-+---+
```

alloc 10 - where do you put it?

* first fit: put in 20-byte block
* best fit: put in 15-byte block better fit, hopefully less waste

Searching the heap

* takes time
* allocations is not free

