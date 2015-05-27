# Lecture 06

## Loaders

#### OS code:
```
repeat:
	p = next program to run
	copy p into memory starting at 0x0c
	jalr $0
	beq $0, $0, repeat
```
Problems: OS is a program, where does it sit in memory? Other programs in memory cannot all be at address 0x0c

* choose different starting addresses for programs at assembly time
	* How will the loader know where to put them?
	* what if two the same?
* let the loader decide where to put the program

Loader

* take a program *p* as input
* find a location a in memory for *p*
* copies *p* to memory starting at *a*
* return *a* to OS

#### OS 2.0:

```
repeat:
	p = next program to run
	$3 = loader(p)
	lalr $3
	beq $0, $0, repeat
```

Loader Pseudocode:

```
Input: words w1,..., wk -- the machine code (p)
n = k + space for stack //how much stack space? pick something!
a = first address of n contiguous words of unused RAM
for i = 1..k
	MEM[a+(i+1)*4] = wi;
$30 = a + 4 * n
return a
```

Problem?

* Labels may be resolved to the wrong addresses
* Loader will have to fix this

What needs to change when we relocate?

```
.word id							//add a to id
.word constant						//do not relocate
anything else (includes beq, bne)	//do not relocate
```

#### OS 3.0

```
repeat:
	p = next program to run
	$3 = load_and_relocate(p)
	jalr $3
	beq $0, $0, repeat
```

Assembled file is a stream of bits. How do we know which ones come from .word (with an id) and which are instructions? 

We do not. We need more information from the assembler.

The output of most assemblers is not pure machine code. It is object code.

### Object file

It contains binary code and auxiliary info needed by the loader (and linker).

Our object code format: MERL (MIPS Executable Relocatable Linkable)

What do we need to put in our object file?

* the code
* which lines of the code (addresses) were originally .word id
* other stuff later

### MREL Format:
```
header		0x10000002	cookie， sanity check
			length of the .merl file
			code length = header+MIPS -- really is MERL
	
MIPS		binary
			assembled to start at address 0x0c

symbol		format code
table		address to relocate
(relocation	Format code
table)		address to relocate
```

### Why 0x10000002?

* It is MIPS for beq $0, $0, 2, command to skip the loader.
* So MERL files can be executed as ordinary MIPS files, if loaded at address 0x00.

Would like the assembler to help us generate relocatable object code - relocatable assembler
