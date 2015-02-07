# Lecture 09

## Finite Language
* has finite words
* can recognize a word by comparing with each word in the (finite) set.

##### Excise: L={cat, car, cow}
write code to answer (w in L)? such that: w is scanned exactly once with out storing previously seen chars.
```
scan the input left-to-right
if first char is c, move on, else error
	if next char is a
		if next char is t
			if input is empty, accept, else error
		else if next char is r
			if input is empty, accept, else error
		else error
	else if next char is o
		if next char is w
			if input is empty, accept, else error
		else error
	else error
else error
```
Abstraction of this program is Trie. Nodes are states (configurations) of the program, based on input seen.

Since programming languages do not usually accept only finitely many programs. Cannot do much with finite languages.

## Regular Languages
Built from
* finite language
* union
* concatenation
* repetition

### Union 
```
L1 U L2 = {x|x in L1 or x in L2}
```

### Concatenation 
```
L1 L2={xy|x in L1, y in L2}
```

### Repetition
```
L* = {empty} U {xy|x in L*, y in L}
   = {empty} U L U LL U LLL U ...
   = 0 or more occurrences of a word L
```

## Regular Expressions
Short hand for regular languages
* empty languages: empty
* language consisting of the empty word: {empty}
* singleton language: {aaaaa}

####Is C regular
A C program is a sequence of token, each of which comes from a regular language
```
ID = [A-Za-z][A-Za-z0-9]
```

How can we recognize an arbitrary regular expression automatically.

## Deterministic Finite Automaton (DFA)
* always start at start state
* for each char in the input, follow the corresponding arc to the next state
* if on an accepting state when input is exhausted, accept, else reject.

Fall of the machine = reject. More formally, an implicit "error" state all transitions go there.

