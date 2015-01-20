# Lecture 5

## The Assembler
```
Assembly Code -> Assembler -> Machine Code
```
Any translation process involves two phases.
1. Analysis - understand what is meant by the source string
2. Synthesis - output the equivalent target string

Assembly file - stream of characters
* First step generally to group characters into meaningful **tokens** e.g. label, hex number, register number, .word, etc.
* This is done for you in asm.ss, asm.cc

Your job: 
* group tokens into instructions (if possible) (Analysis)
* output equivalent machine code (Synthesis)

If the tokens are not arranged into sensible instructions, output ERROR to stderr.

#### Advice:
There are many more wrong token configurations than right ones.
* Focus on finding all the right ones, anything else is error.
* Later, more structured ways of grouping tokens (parsing)

### Biggest problem with writing Assemblers
#### How do we assemble:
```assembly
beq $0, $1, abc
...
abc:
...
```
We cannot assemble the "beq" because we do not yet know what "abc" is.
##### Standard Solution: assemble in two passes
1. group tokens into instructions, record the addresses of all labelled instructions ("symbol table" - list of (label, address) pairs)
2. translate each instruction into machine code if an instruction refers to a label, look up the associated address in the symbol table.

Note: 
* a line of assembly may have more than one label. (e.g. "f:g: mult $1 $2")
* can label the address just past the end of the program

Your assembler:
* output assembled MIPS to stdout
* output the symbol table to stderr

##### Example:
```assembly
main:
	lis		$2
	.word	13
	add		$3, $0, $0
top:
	add		$3, $3, $2
	lis		$1
	.word	$1
	sub		$2, $2, $1
	bne		$2, $0, top
	jr		$31
beyond:
```
Pass 1: Group tokens into instructions, build symbol table.

lable|address
---|---
main|0x00
top|0x0c
beyond|0x24

Pass 2: Translate each instruction
```assembly
lis		$2				; -> 0x00001014
.word	13				; -> 0x0000000d
...
bne		$2, $0, top		; lookup top in symbol table (0x0c), calculate "(top-pc)/4=-5" -> 0x1440fffb
```
Note: to negate a two's complement number, flip the bits and add 1.
```
5	00000000 00000101
-5	11111111 11111010 + 1
	11111111 11111011
	fffb
```

### Bit-level operations
To assemble "bne $2, $0, -5"
* opcode = 000101 = 5
* 1st register = $2 = 2
* 2nd register = $0 = 0
* offset = -5
6 bits + 5 bits + 5 bits + 16 bits = 32 bits

1. to put 000101 in the first 6 bits, need to append 26 0, i.g. left shift by 26 bits
2. move $2 21 bits to the left
3. move $0 16 bits to the left
4. -5 is 32 bits, want only the last 16 bits.

Bitwise "and" and "or"
* bitwise & with 1: original value stays the same
* bitwise & with 0: result is 0

* bitwise | with 1: result is 1
* bitwise | with 0: original value stays the same

So do a bitwise "and" with 0xffff.

Then bitwise "or" the four prices together (5<<26|2<<21|0<<16|-5&0xffff)

### Print
```c++
unsigned int instr;
unsigned char c;
c = instr >> 24;
cout << c;
c = instr >> 16;
cout << c;
c = instr >> 8;
cout << c;
c = instr >> 0;
cout << c;
```
