CS 241 Foundations of Sequential Programs
===

What is a sequential program?
---

Single threaded, only one thing going on at a time.

Goal: Foundations
---

Understand how sequential programs "work" from the ground up

Starting point: bare hardware (so to speak)

for CS 241, bare hardware is simulated MIPS machine which only understand 1 and 0

At the end - get programs written in a C-like language to run on MIPS

The beginning: Binary and Hexadecimal Numbers
---

Bit: binary digit, i.e. 0 or 1 (high/low voltage, contigurations on magnetic tape)

- all the computer understands
- convenient to group bits together

byte: 8 bits, e.g. 11001010, 256 possible bytes

word: machine specific grouping of bytes

- we will assume a 32-bit computer architecture
- 1 word = 32 bits = 4 bytes

4 bits (1/2 a byte) sometime called a nibble

Q: Given a byte (or a word) in the computer's memory, what does it mean?

Ex: 11001011 means what?

A: It could mean many things. 

1. A number. Binary number system. 11001001 = 201

How can we represent negative numbers?

Simple way: resever the first bit to represent the sine: o for +, 1 for -
The 11001001 = -73

But: two representation for 0: 00000000, 100000000
- wasteful
- arithmetic on numbers is tricky (e.g. add a positive number and a negative number)

Better way: 2's compleneat notation

1. Interpret the n-bit numbers on unsigned integer
2. If the first bit is 0, done.
3. else subtract 2^n

E.g. for n = 3

* 000: 0
* 001: 1
* 010: 2
* 011: 3
* 100: -4
* 101: -3
* 110: -2
* 111: -1

n bits represent numbers -2^(n-1)~2^(n-1)-1

- only one 0
- left bit gibes sign
- additions is clean - just module 2^n
- same addition circuitry works for signed and unsigned

Then 11001001 = 201-256 = -55

Convenice: Hexadecimal number
---

- base 16, 0...F
- more compacition binary
- each hex digit = 4 bit (1 nibble)
- 11001001 = 0xC9

Q: Given a byte 11001001, how can we tell if it is unsigned, sign-magnititu, or 2's comp?
A: We cannot, Need to remember what our intent was when we stored the byte!

We do not know for sure that 11001001 represents a number!

A character- which charactor?
---

Need conventional mappin between bit patterns or charactors

ASCII: uses 7 bits
- 8 bit can be used for additional chars
- non-standard compatibility issues

11001001 is not 7-bit ASCII
100101 decimal 73 is ASCII for I

An instruction (or part for one)
---

- For us, instructions are 32 bits=4bytes
- 11001001 could be part of an instruction

Garbage (unused memory)
---

EBCDIC
---

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

        CPU                 Main Memory
|------------------|       |-----------|
| -----      ----- |       |           |
| |ALU|      |R00| |       |           | 
| -----      |R01| |       |           |
| ---------  |...| |       |           |
| |Control|  |R31| |  BUS  |           |
| |Unit   |  ----- |<----->|           |
| ---------  ----  |       |           |
| -----      |PC|  |       |           |
| |MAR|      ----  |       |           |
| -----      |IR|  |       |           |
| |MDR|      ----  |       |           |
|------------------|       |-----------|

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
