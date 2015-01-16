# Lecture 04

## Procedures
Want to write a procedure *f* in MIPS assembly

### Problems
##### Call and Return
* How do we transfer control in and out of *f*? What if *f* calls *g*?
* How do we pass parameters?

##### Registers
* What if *f* overwrites register we were using?

e.g. Some critical data in $3, call procedure *f*, *f* modifies $3, on return, my data gone!

We could reserve some of the registers for *f* and some for the main line
* So they do not interfere with each other
* What if *f* calls *g* ... etc? Soon run out of registers
* Instead guarantee that procedures leave registers unchanged when done

Use RAM
* How do we keep procedures from using the same RAM?

Sensible options
* allocate from either the top or the bottom of RAM
* need to keep track of which RAM is used and which is not.

MIPS machine helps us out, $30 initialized (by the loader) to just past the word of memory

Can use $30 as a "bookmark" to separate used and unused RAM, if we allocate from the bottom.

So allocate from the bottom!

Example f, g, h
Say f calls g, g calls h
```
 ____
|Code|
|____|
|h   |
|____|
|g   |
|____|
|f   |
|____|
```
each procedure stores in RAM the register it uses, and restores the original values on return.

### Stack
$30, the stack pointer, contains address of the top of the stack.

### Template for procedures
```assembly
f:	
	sw		$2, -4($30)
	sw		$3,	-8($30)
	lis		$3
	.word	8
	sub		$30, $30, $3

	; body

	add		$30, $30, $3
	lw		$3, -8($30)
	lw		$2, -4($30)

	jr		$31				; return
```
What about call and return?
##### Call
```assembly
main:
	lis		$5
	.word	f
	jalr	$5	
```
##### Return
* need to set pc to the line after the jr (i.e. to HERE above)
* How do we know which address that is?

##### Solution
jalr instruction "jump and link to register"

* like jr, but sets $31 to the address of the next instruction (pc)

##### Q: jalr overwrites $31 then how can we return to the loader? And what if f calls g?

##### A: Need to save $31 on the stack
```assembly
main:
	lis		$5
	.word	f
	sw		$31, -4($30)
	lis		$31
	.word	4
	sub		$30, $30, $31
	jalr	$5
	lis		$31
	.word	$4
	add		$30, $30, $31
	lw		$31, -4($30)
	;...
	jr		$31	; return to loader
f:
	;...
	jr		$31	; return to caller
```
### Parameter & Result Passing
* generally use register
* if too many parameters, can push parameters on to stack

```assembly
; Sum one to N: adds the numbers 1, ...,  N
; $1 word
; $2 input
; $3 output
SumOneToN:
	sw		$1, -4($30)
	sw		$3, -8($30)
	lis		$1
	.word	8
	sub		$30, $30, $1
	add		$3, $0, $0
top:
	add		$3, $3, $2
	lis		&1
	.word	1
	sub		$2, $2, $1
	bne		$2, $0, top
	lis		$1
	.word	8
	add		$30, $30, $1
	lw		$2, -8($30)
	lw		$1, -4($30)
	jr		$31
```

### Recursion
* no extra machinery needed
* if register, parameters, and stack managed properly, recursion will just work.

### Input & Output
* output
	* use sw to store word in location
* 0xffff000c
	* least significant byte will be printed

e.g. print "A\n"
```assembly
lis		$1
.word	0xffff000c
lis		$2
.word	65
sw		$2, 0($1)
lis		$2
.word	10
sw		$2, 0($1)
```
