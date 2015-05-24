# Lecture 03

### Example 2:
Add 42+52, store result in $3, return
```
Location|Binary                                 |hex       |meaning
--------|---------------------------------------|----------|------------
00000000|0000 0000 0000 0000 0010 1000 0001 0100|0x00002814|lis $5
00000004|0000 0000 0000 0000 0000 0000 0010 1010|0x0000002a|.word 42
00000008|0000 0000 0000 0000 0011 1000 0001 0100|0x00003814|lis $7
0000000c|0000 0000 0000 0000 0000 0000 0011 0100|0x00000034|.word 52
00000014|0000 0000 1010 0111 0001 1000 0010 0000|0x00a71820|add $3, $5, $7
00000018|0000 0011 1110 0000 0000 0000 0000 1000|0x03e00008|jr $31
```

#### lis $d load immediate and skip
Treat next word as an immediate value and load it into $d, then skip to the **following** instruction

## Assembly Language
Replace tedious binary/hex encodings with easier-to-use shorthad.
* less chance of error
* translation to binary can be automated (assembler)

One line of assembly = one machine instruction (one word)

For example 2
```assembly
lis		$5				; load immediate and skip
.word	42				; not an instruction (directive), directly translate the word into binary
lis		$7
.word	52
add		$3,	$5,	$7		; desalination first ($3 <- $5 + $7)
jr		$31
```

### Some Instruction Modify *pc* - "branches & jumps"
#### beq 
- branch if two register have equal contents
- increment *pc* by the given number of words
- can branch backwards
#### bne
#### slt "set less than"

#### Example 3:
Compute the absolute value of $1, store in $1 and return
```assembly
slt	$2,	$1,	$0	; compare $1 < 0
beq $2,	$0,	1	; if false, skip
sub $1,	$0,	$1	; $1 = 0 - $1
jr	$31
```

#### Example 4:
sum the integers 1 to 13, store in $3 return
```assembly
lis		$2			;
.word	13			;
add		$3,	$0,	$0	;
lis		$1			;  
.word	1			;  
add		$3,	$3,	$2	;<-|
sub		$2,	$2,	$1	;  |
bne		$3,	$0,	-3	;--|
jr		$31			;
```

### RAM
#### lw 
* "load word" from RAM into register
* lw $a, i($b) loads MEM[b + i] into $a
#### sw
* store word, from register to RAM
* sw $a, i($b) stores $a at MEM[$b + i]

#### Example 5:
* $1 address of an array
* $2 number of elements in the array
place element 5 in $3
#### Solution 1:
```assembly
lw	$3,	20($1)
jr	$31
```
#### Solution 2:
What if the item number you want is not known before runtime

#### mult $a, $b
Product of 2 32-bit numbers is 64-bits (too long for a register), so two special register, hi & lo store the result of mult. 

mult $a, $b = hi:lo

#### mflo "move from lo" mfhi "move from hi"

For division
* lo store quotient
* hi store remainder

```assembly
lis		$5
.word	5
lis		$4
.word	4
mult	$5,	$4
mflo	$5
add		$5,	$5,	$1
lw		$3,	0($5)
jr		$31
```
Moving instructions into/out of branches -- must update branch offsets
Instead, assembler allows labelled instructions
```assembly
label:	instruction
foo:	add $1, $2, $3
```
Assembler associates the name foo with address of add
```assembly
lis			$2			;
.word		13			;
add			$3,	$0,	$0	;
lis			$1			;  
.word		1			;  
loo:add		$3,	$3,	$2	;<-|
sub			$2,	$2,	$1	;  |
bne			$3,	$0,	loo	;--|
jr			$31			;
```
