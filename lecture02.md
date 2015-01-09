Lecture 2
===

Machine Language
---
Computer programs operate on data

Computer programs are data

(von Neumann Architecture)

reside in the same memory as the data they operate on

therefore possible to write programs that manipulate other programs. (operating system, compilers, viruses)

How do we know which parts of memory represent code and which represent data? We do not.

What does a machine instruction look like? What instruction are there?

- Many different machine languages - processor specific

For us: MIPS(simplified): 18 different 32-bit instruction types

MIPS Machine
```
           CPU                    Main Memory
|------------------------|       |-----------|
| |--| |--|  |---| |---| |       |           |
| |PC| |IR|  |R00| |ALU| |       |-----------| 
| |--| |--|  |R01| |---| |  BUS  |           |
| ---------  |...| |MAR| |<----->|-----------|
| |Control|  |R30| |---| |       |           |
| |Unit   |  |R31| |MDR| |       |-----------|
| --------|  |---| |---| |       |           |
|------------------------|       |-----------|
```

CPU(Central Processing Unit) "brain of the computer"
---
Contral Unit: 
* decodes instructions
* dispatches to other parts of the computer to carry them out

ALU(Arithmetic/Logic Unit) "does math"

Memory - many kinds

Fast -> Slow

CPU cache Main memory disk memory network memory

On the CPU

- small amount of very fast memory: registar

MIPS

* R00...R31
* 32 general purpose registers
* each holds 32 bits, i.e. one word

CPU can only operate directly on data in registers called $0, ..., $31

* $0 is always 0
* $31 is special (more later)
* $30 is also sort of special 
* example register operation: "add the contents of regsiter s and t and put the result in register d"

How many bits does it take to encode a register number? 5

therefore 15 bits to encode 3 register numbers - leaves 17 bits to encode operation

RAM 
---
* large amount of memory, stored away from the CPU
* data travels between RAM and the CPU on the bus
* big array of n bytes for large n (n ~ 10^9)
* each cell has an address 0,...,n-1
* each 4-byte block is a word
* words have addresses 0, 4, 8, C, 10, 14, 18, 1C, ...
* RAM access much slower than register access
* data in RAM must be transfered to register before it can be used

Communicating with RAM (two commands)
---
load (transfer word from a specified address to a specified register, the disired address goes in to Memory Address Register(MAR))
* goes out on bus
* data at that location comes back on the bus and goes into the Memory Data Register (MDR)

Store (similar)

Do not know which words in memory contain data & which contain code, then how does the machine execute code?

Special register called PC (program conter) holds the address of the next instruction to run.

By convention, we guarantee that a specific address (say 0) contains code to initialize PC to 0.

Computer then runs the fetch-execute cycle:
```
PC <- 0
loop
	IR <- MEM[PC] \\IR = instruction register, holds the current instruction
	PC <- PC + 4
	decode and execute instruction in IR
end loop
```
only program the machine really runs

Again - PC holds the address of the next instrution while the current instruction is running

Q: How does a program get executed?

A: Program called a leader puts your program in memory and sets PC to the address of the first instruction of the program

Q: What happens when a program ends?

A: Need to return control to the loader. 

Set PC to the address of the next instruction in the loader.

$31 will contain the reight address. Need to set PC to $31

Example 1: Add the value in reg 5 to reg 7 store the result in $3 the return.
```
Location|Binary                                 |hex       |meaning
------------------------------------------------------------------------
00000000|0000 0000 1010 0111 0001 1000 0010 0000|0x00a71820|add $3 $5 $7
00000004|0000 0011 1110 0000 0000 0000 0000 1000|0x03e00008|jr $31
```
Example 2: Add 42 to 52, store in $3, return
```
Location|Binary                                 |hex       |meaning
------------------------------------------------------------------------
00000012|0000 0011 1110 0000 0000 0000 0000 1000|0x03e00008|jr $31
```
