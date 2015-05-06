# Lecture 01

#### What is a sequential program?
Single threaded, only one thing going on at a time.

#### Goal: Foundations
Understand how sequential programs "work" from the ground up

#### Starting Point
Bare hardware (so to speak)

For CS 241, bare hardware is simulated **MIPS** machine which only understand 1 and 0

#### At the end
Get programs written in a C-like language to run on MIPS

## The beginning: Binary and Hexadecimal Numbers

### Bit
Binary digit, i.e. 0 or 1 (high/low voltage, configurations on magnetic tape)

- all the computer understands
- convenient to group bits together

### Byte 
8 bits, e.g. 11001010, 256 possible bytes

### Word
Machine specific grouping of bytes

- we will assume a 32-bit computer architecture
- 1 word = 32 bits = 4 bytes

4 bits (1/2 a byte) sometime called a nibble

#### Q: Given a byte (or a word) in the computer's memory, what does it mean? E.g. 11001011 means what?
#### A: It could mean many things. 

### 1. A number
Binary number system. 11001001 = 201

#### How can we represent negative numbers?

Simple way: reserve the first bit to represent the sine: o for +, 1 for -. e.g. the 11001001 = -73

**But**: two representation for 0: 00000000, 100000000
- wasteful
- arithmetic on numbers is tricky (e.g. add a positive number and a negative number)

Better way: **2's Complement Notation**

1. Interpret the n-bit numbers on unsigned integer
2. If the first bit is 0, done.
3. else subtract $2^n$

E.g. for n = 3
```
|000|001|010|011|100|101|110|111|
|  0|  1|  2|  3| -4| -3| -2| -1|
```

n bits represent numbers $-2^{n-1}$~$2^{n-1}-1$

- only one 0
- left bit gibes sign
- additions is clean - just module $2^n$
- same addition circuitry works for signed and unsigned

Then 11001001 = 201-256 = -55

#### Convince: Hexadecimal number

- base 16, 0...F
- more complication binary
- each hex digit = 4 bit (1 nibble)
- 11001001 = 0xC9

#### Q: Given a byte 11001001, how can we tell if it is unsigned, sign-magnitude, or 2's comp?
#### A: We cannot, Need to remember what our intent was when we stored the byte!

We do not know for sure that 11001001 represents a number!

### 2. A character- which character?

Need conventional mapping between bit patterns or characters

#### ASCII 
- uses 7 bits
- 8 bit can be used for additional chars
- non-standard compatibility issues

11001001 is not 7-bit ASCII. 100101 decimal 73 is ASCII for I

### 3. An instruction (or part for one)

- For us, instructions are 32 bits=4bytes
- 11001001 could be part of an instruction

### 4. Garbage (unused memory)
