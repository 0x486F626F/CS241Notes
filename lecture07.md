# Lecture 07

#### Relocation tool: cs241.MERL
* input: MERL file and relocation address
* output: non-relocatable mips file, with MERL header and footer removed, ready to load at the given address

mips.twoints and mips.array - optional second argument = address at which to load the mips file.

#### Example
myobj.MERL to be loaded at 0x10000
```bash
java cs241.MERL 0x10000 < myobj.MERL > myobj.mips
java mips.twoints myobj.mips 0x10000
```

### Loader Relocation Algorithm
```
read()									//skip cookie
endMod = read()							//end of MERL file
codeLen = read()						//length of code only
a = FindFreeRAM(codeLen + stack space)
for(i = 0; i < codeLen; i += 4)
	MEM[a + i] = read();
i = codeLen + 12						//start of relocation table
while(i < endMod)
	format = read()
	if(format == 1)
		rel = read()					//address to be relocated, relative to start of header (not code)
		Mem[a + rel - 12] += a - 12		//actual location (header not loaded), adjust forward by a, backwards by header length since header not loaded
	else ERROR
	i += 8
```

## Linkers
Convenient to split large MIPS programs into smaller units. 

* Reusable libraries
* Team development

How can the assembler resolve a reference to a label if it is in a different file?

#### Solution 1: cat all asm files together and then assemble

```bash
cat a.asm b.asm c.asm | java cs241.binasm > result.mips
```
This will work, but why should we have to assemble these files more than once (i.e. every time we cat them into a larger project). 

###Cannot we assemble first and then cat?

####Issues

* output needs to be relocatable
* at most one of the pieces can be loaded at address 0x00
* so need to output MERL, not just MIPS
* But catting two MERL files does not yield MERL:

```
 ______          ______
|header|        |header|
|body  |        |body  |
|footer|\       |footer|
|______| \      |header|
 ______   cat-> |body  |
|header| /      |footer|
|body  |/       |______|
|footer|
|______|
```

#### Solution 2: Need a tool that understands MERL files and puts them together intelligently -- a linker

But still

* What should the assembler do with references to labels that are not there?
* Need to change the assembler
* When the assembler encounters .word id 

Where label id: not found, it fills in 0 temporarily, and it must indicate that the program requires the value of id before it can run.

one.asm
```assembly
lis $3
.word a
```
two.asm
```assembly
a: ...
```
one.asm cannot be executed without being linked to code that contains babel a

#### How does the assembler notify us?

Make an entry in the MERL file, but

* We lose a valuable error check!
* What if we meant adb when we said abc?
* Assembler will ask for abc to be linked in, instead of signalling an error.

```assembly
lis $3
.word abc
abd: ...
```
How can the assembler know what is an error and what is intentional?

New assembler directive:

.import id

* Tells assembler to ask for id to be linked in.
* Dose *NOT* assembler to a word of MIPS

So when the assembler encounters .word abc, iF label abc does not occur in current file and no .import abc directive, then error

Notifying the system: make an entry in MERL table

Formal code 0x11 means External Symbol Reference (ESR)

What info is needed?

* where (i.e. at what address) is the symbol being used? (where is the blank we need to fill in?)
* what is the name of symbol?

Format of an ESR entry:

```
word 1					- 0x11
word 2					- location where the symbol is used
word 3					- length of the name in chars (n)
word 4 ... word (3 + n)	- ASCII chars in the symbol name (each char in its own word)
```

The other side:

a.asm (external reference to *abc*)
```assembly
lis $3
.word abc
```
b.asm (*abc* is the beginning of a procedure)
```assembly
abc: sw $3, -4($30)
...
jr $31
```
c.asm (just an internal label)
```assembly
...
abc: add $1, $2, $3
...
beq $2, $0, abc
```

How can the linker know which *abc* to link to? 

Cannot assume labels will not be duplicated.

How can we make *abc* in b.asm accessible, and *abc* in c.asm inaccessible?

Solution: another assembler directive and MERL entry.

Directive: .export abc 
* make abc available for linking
* does NOT assemble to a wor of MIPS
* tells the assembler to make an entry in the MERL symbol table

### MERL entry: External Symbol Definition (ESD)

Format:

```
word 1				- 0x05 format code for ESD
word 2				- address the symbol represents
word 3				- length of name (n)
word 4 ~ word (3+n)	- Then name in ASCII one word per char
```
