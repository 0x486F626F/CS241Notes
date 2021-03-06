# Lecture 08

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
if (m1.exports.labels $\cap$ m2.exports.labels) is not empty, ERROR
for each <addr1, label> in m1.imports
	if exists <addr2, label> in m2.exports
		m1.code[addr1] = addr2
		remove <addr1, label> from m1.imports
		add addr1 to m1.relocates
	if exists <addr1, label> in m1.exports
		m2.code[addr2] = addr1
		remove <addr2, label> from m2.imports
		add addr2 to m2.relocates
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

```
                      ________
High Level Language->|Compiler|->Assembly Language
                     |________|
```

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

### Alphabet

* finite set of symbols (e.g. $\{a, b, c\}$)
* typically denoted $\Sigma$, as in $\Sigma=\{a, b, c\}$

### String (or word)

* Finite sequence of symbols (form $\Sigma$), e.g. $a$, $abc$, $cbca$, ...
* Length of a word $|w|$=number of chars in $w$, e.g. $|aba|=3$

#### Empty String

* Empty string -- an empty sequence of symbols. 
* $\epsilon$ denotes the empty string. 
* $\epsilon$ is NOT a symbol, $|\epsilon|=0$

### Language
* set of strings (words), e.g. $\{a^{2n}b | n\geq 0\}$ ({all words with an even number of a, followed by b})

Note

* $\epsilon$ -- empty word
* $\{\}$ -- empty language contains no words
* $\{\epsilon\}$ -- singleton language that contains only $\epsilon$.

Symbol can be anything we want

#### Our task: How can we recognize automatically whether a given string belongs to a given language?

#### Answer: Depends on how complex the language is.

* $\{a^{2n}b | n\geq 0\}$ -- easy
* $\{$valid MIPS assembly programs$\}$ -- almost easy
* $\{$valid C programs$\}$ -- harder
* $\{$Some languages$\}$ -- impossible

Characterize languages by complexity of recognition. (from easy to harder to impossible)

* finite 
* regular
* context-free
* context sensitive
* recursive
* ets.

Want to start at as easy a level as possible, move to a harder level when necessary.

## Finite Languages

* has finite words
* can recognize a word by comparing with each word in the (finite) set.
* can we do this more efficiently?

#### Excise: $L=\{cat, car, cow\}$
Write code to answer $w\in L$? Such that: $w$ is scanned exactly once with out storing previously seen characters.

```
scan the input left-to-right
if first char is 'c', move on
	if next char is 'a'
		if next char is 't'
			if input is empty, accept, else error
		else if next char is 'r'
			if input is empty, accept, else error
		else error
	else if next char is 'o'
		if next char is 'w'
			if input is empty, accept, else error
		else error
	else error
else error
```

Abstraction of this program:

![08-01](pic/08-01.png)

Bubbles are states -- Configurations of the pregram, based on input seen.

e.g. MIPS operators

![08-02](pic/08-02.png)

Since programming languages do not usually admit only finitely many programs, 
finite languages are not that useful.
