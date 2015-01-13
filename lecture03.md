# Lecture 03

#### Example 2:
Add 42 + 52, store result in $3, return
```
Location|Binary                                 |hex       |meaning
--------|---------------------------------------|----------|------------
00000000|0000 0000 0000 0000 0010 1000 0001 0100|0x00002814|lis $5
00000004|0000 0000 0000 0000 0000 0000 0001 1010|0x0000002a|.word 42
00000008|0000 0000 0000 0000 0011 1000 0001 0100|0x00003814|lis $7
0000000a|0000 0000 0000 0000 0000 0000 0011 0100|0x00000034|.word 52
00000014|0000 0000 1010 0111 0001 1000 0010 0000|0x00a71820|add $3, $5, $7
00000018|0000 0011 1110 0000 0000 0000 0000 1000|0x03e00008|jr $31
```

#### lis $d load immediate and skip
Treat next word as an immediate value and load it into $d, then skip to the **following** instruction

## Assembly Language
Replace tedious binary/hex encodings with easier-to-use mnemonics.
* less chance of error
* translation to binary can be automated (assembler)
* one line of assembly = one machine instruction (one word)
For example 2
```
lis $5				- load immediate and skip
.word 42			- not an instruction, directly translate the word into binary
lis $7
.word 52
add $3, $5, $7		- deslination first ($3 <- $5 + $7)
jr $31
```

#### Example 3:
Compute the absolute value of $1, store in $1 and return
* some instruction modify *pc* - "braches & jumps"
* *beq* 
	- branch if two register have equal contets
	- incrament pc by the given number of words
	- can branch backwords
* also *bne*
* *slt* "set less than"

