# Lecture 12

"abab" could be interpreted to be 1, 2, 3, 4 tokens.

Decide to only take the epsilon-move if no other choice: always look for the longest possible next token.

Consider L={aa, aaa}, w=aaaa. Take longest token first, "aaa", "a" left over, but "aa" and "aa" could have been matched.

## Maximum Munch Algorithm
Run DFA (without epsilon-moves) until no non-error move is possible. 
```
If in accepting state, output token founded. 
else	back up to most recent accepting state
		input to that points is next token
		resume scanning from there
endif
output token: epsilon move back to go
```

## Simplified Maximum Munch Algorithm
As above, but if not in a accepting state when no transition is possible, error (no move back)

In practice, Simplified Maximum Munch usually good enough. Language typically designed to facilitate scanning by Simplified Maximum Munch.

Example in C++: vector <vector<int>> v;

C++ longest match scanner scans this as one token, >> rather than as > >

C++ solution: adapt the language to the scanner, must separate > by space to create two tokens.

What (if any) feature of C(or scheme) programs cannot be verified with DFA.

Consider E={(, )} L={w in E^* | w is a string of balanced parentheses}
![12-01](/pic/12-01.png)

Each new state recognizes one more level of nesting - but no finite number of states recognizes all levels of nesting, and DFAs must have finitely many states.

## Context-Free Languages
Languages that can be described by a context-free grammar, set of "rewrite rules".

#### Intuition-balanced pares
A word in the language is either empty or a word in the language surrounded by (), or the concatenation of two words in the language.

Shorthand: S->E|(S)|SS

Show this system generates (())()
```
S=>SS=>(S)S=>((S))S=>(())S=>(())(S)=>(())()
```

Notation: "=>" = "derives"
"a=>b" means b can be obtained from a by one application of grammar rule.

Def: A context-free grammar consists of
* An alphabet epsilon of terminal symbols.
* A finite non-empty set N of non-terminal symbols, N intersects Sigma = empty (we use V "vocabulary" to denote N intersects Sigma)
* A finite set P of productions. Productions have the form A->B where A in N, B in V^*
* An element S in N (Start Symbol)

Conventions:
a, b, c, ... elements of Sigma (characters)
x, y, z, ... elements of Sigma^* (string)
A, B, C, ... elements of N (non-terminals)
Alpha, Beta, Gamma, ... elements of V^* ((N intersected Sigma)^*)

We write AlphaAB=>AlphaGammaB if there is a production A->Gamma in P (RHS derivable from the LHS in one step). 
```
AlphaAB=>*AlphaGammaB means AlphaAB=>Delta_1=>Delta_2=>...Delta_k=>AlphaGammaB
```
k >= 0 (0 or more derivation steps)

Def: L(G)={w in Sigma^* | S=>*w} language specified by grammar, string of terminals derivable from S.

Def: A language L is context-free if L=L(G) for some context-free grammar G.

Ex: Palindromes over {a, b, c}
```
S->aSa|bSb|cSc|M M->a|b|c|empty
```

Expressions: epsilon = {a, b, c, +, -, *, /} L={arithmetic expressions, using symbol from Sigma}
```
S->a|b|c|S op S| op->+|-|*|/
Sigma`=Sigma U {(, )} L`={expressions with parentheses}
S->a|b|c|S op S|(S) op->+|-|*|/
```

Show: S=>*a+b
```
S=>S op S=>a op S=>a + S=>a + b
```

Leftmost derivation -- always expand leftmost symbol first

Rightmost derivation -- rightmost first.

Derivations: expressed naturally + succinctly as a tree structure.

![12-02](/pic/12-02.png)

For every leftmost (for rightmost) derivation tree is a unique parse tree.
```
