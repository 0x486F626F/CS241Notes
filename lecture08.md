# Lecture 08

The other side:
a.asm
```assembly
lis $3
.word abc
```
b.asm
```assembly
abc: sw $3, -4($30)
...
jr $31
```
c.asm
```assembly
...
abc: add $1, $2, $3
...
beq $2, $0, abc
```
How can the linker know which *abc* to link to? Cannot assume labels will not be duplicated.

How can we make one *abc* available and the other not available?

Solution: another assembler directive and MERL entry.

Directive: .export abc 
* make abc available for linking
* does NOT assemble to a wor of MIPS
* tells the assembler to make an entry in the MERL symbol table

### MERL entry: External Symbol Definition (ESD)
Format:
```
word 1 - 0x05 format code for ESD
word 2 - address the symbol represents
word 3 - length of name (n)
word 4 ~ word (3+n) - Then name in ASCII one word per char
```

MERL file contains:
* the code
* all addresses that need relocating
* addresses and names of every ESR
* addresses and names of every ESD

i.e. everything the linker needs to do its job.

### Linker Algorithm
* Input: merl files m1 and m2
* Output: single merl file with m2 linked after m1.
```
a = m1.codeLen-12
relocate m2.code by a
add a to every address in m2.symtbl
if m1.exports.labels + m2.exports.labels is not empty, ERROR
for each <addr1, label> in m1.imports
	if exists <addr2, label> in m2.exports
		m1.code[addr1] = addr2
		remove <addr1, label> from m1.imports
		add addr1 to m1.relocates
	if exists <addr1, label> in m1.exports
		m2.code[addr2] = addr1
		remove <addr2, label> from m2.imports
		add addr2 to m2.relocates
```
```
imports = m1.imports + m2.imports
exports = m1.exports + m2.exports
relocates = m1.relocates + m2relocates
output merl cookie
output total code length + total (import, export, relocate) + 12
output total code length + 12
output m1.code
output m2.code
output imports, exports, relocates
```

## Formal Languages

Assembly
* simple structure
* easy to recognize (parse)
* straightforward, unambiguous translation to machine language

High-level language
* more complex structure
* harder to recognize
* no single translation to machine language

How can we handle the complexity?

Want a formal theory of string recognition -- general principles that work in the context of any programming language

#### Alphabet
* finite set of symbols ({e.g. a, b, c})
* typically denoted \Sigma, asin \Sigma={a, b, c}

* String (or word) -- finite sequence of symbols (form \Sigma)
* Length of a word |w|=number of chars in w
* Empty string -- an empty sequence of symbols. \epsilon denotes the empty string. \epsilon is NOT a symbol.
* Language -- set of strings (words)

Note
* \epsilon -- empty word
* {} -- empty language contaning no words
* {\epsilon} -- singleton language that contains only \epsilon.

Symbol can be anything we want

Our task: How can we recognize automatically whether a given string belongs to a given language?

Answer: Depends on how complex the language is.

*{valid C programs} -- harder
*{Some languages} -- impossible

Characterize languages by complexity of recognition. (from easy to harder to impossible)
* finite 
* regular
* context-free
* context sensitive
* recursive
* ets.

Want to start at as easy a level as possible, move to a harder level when necessary.
