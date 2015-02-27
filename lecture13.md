# Lecture 13

```
S = a|b|c|S op S
op -> +|-|*|/
```
is an ambiguous grammar.

#### Ex. Leftmost derivation for a+b*c 
```
S => S op S => a op S => a + S => a + S op S => a + b op S => a + b * S => a + b * c
```
or
```
S => S op S => S op S op S => a + S op S => a + b op S => a + b * S => a + b * c
```
Two different parse trees
![13-01](/pic/13-01.png)

A grammar for which some word has more than one distinct leftmost derivation (equivalent, parse tree) is called **ambiguous**.

If we only care whether w in L(G), ambiguity does not matter. But as compiler writers, we want to know why w in L(G). i.e. the derivation matters.

The shape of the parse tree describes the meaning of the string, so a word with multiple parse tree can have multiple meanings.
![13-02](/pic/13-02.png)

So a+b*c could mean (a+b)*c or a+(b*c). What do we do?

1. Use heuristics (precedence) to guide the derivation process
2. Make the grammar unambiguous.

```
E->E op T|T
T->a|b|c|(E)
op->+|-|*|/
a+b*c: E => E op T => E op T op T => T op T op T => a op T op T => ... => a + b * c
```
![13-03](/pic/13-03.png)

What if we want to give * / precedence over + -?
```
E->E PM T|T
T->T TD F|F
F->a|b|c|(E)
PM->+|-
TD->*|/
a+b*c: E => E PM T => T PM T => F PM T => a PM T => a + T => a + T TD F => a + f TD F => a + b TD F => a + b * F => a + b * c
```

Q: If L is contest-free, is there always an unambiguous grammar G such that L=>L(G)

A: No! There are inherently ambiguous languages that only have ambiguous grammars

Q: Can we construct a toll that tell us if a grammar is ambiguous?

A: No! Undecidable.

## Recognizers
What class of computer programs is needed to recognize a CFL?

* Regular language: DFA, essentially a program with finite memory
* Context-free language: NFA + stack, infinite memory, but use is limited to LIFO order.

But we need more than just a Y/N answer -- need the derivation (parse tree). Problem of finding the derivation is called parsing
```
Given: Grammar G, start symbol S, word W
Find: S => ... => w (or report that there is no derivation)

How can this be done? 2 choices:

1. Forwards (top-down): start at S, expand until you reach w
2. Backwards (bottom-up): start at w, figure out how to get back to S.

### Top-Down Parsing
Start with S, apply grammar rules, reach w
```
S => a1 => a2 => ... => an => w
```
Use the stack to store intermediate ai, reverse, match against char in w.

Invariant: consumed input + reverse (stack contents) = a
```
S->Ayb
A->ab
A->cd
B->z
B->wx
```
For simplicity, we will use augmented grammars for parsing, i.e. invent two new symbols |- -|, new start symbol S'
```
S->|-S-|
S->AyB
A->ab
A->cd
B->z
B->wx
say w=|-abywx-|
```
