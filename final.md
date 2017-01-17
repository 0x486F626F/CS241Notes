# Review Session

1. Top Down Parsing
2. Bottom Up Parsing
3. Error Checking
4. Code Generation
5. Optimization
6. Memory Management

## Top Down Parsing

Build a prediction Table to parse $\vdash dfeaccb \dashv$

* $S'->\vdash S \dashv$
* $S->aYb$
* $S->XS$
* $Y->ccZY$
* $Y->\epsilon$
* $X->dZe$
* $Z->f$
* $Z->\epsilon$

### First Set

* $First(S')=\{\dashv\}$
* $First(S)=\{a,d\}$
* $First(Y)=\{c\}$
* $First(X)=\{d\}$
* $First(Z)=\{f\}$

### Follow Set

* $Follow(S')=\{\}$
* $First(S)=\{\vdash\}$
* $First(Y)=\{b\}$
* $First(X)=\{a, d\}$ first set of $S$
* $First(Z)=\{e, c, b\}$ $e$ follows $Z$, and first set of $Y$, and follow set of $Y$ because $Y$ is nullable

### Prediction Table

||$\vdash$|$\dashv$|$a$|$b$|$c$|$d$|$e$|$f$
---|---|---|---|---|---|---|---|---
$S'$|1||||||||
$S$|||2|||3|||
$X$||||||6|||
$Y$||||5|4||||
$Z$||||8|8||8|7

### Parsing

Action | Stack | Unread
---|---|---
init| $S'$ | $\vdash dfeaccb \dashv$
expand 1| $\dashv S \vdash$ | $\vdash dfeaccb \dashv$
match $\vdash$ | $\dashv S$ | $dfeaccb \dashv$
expand 3| $\dashv SX$ | $dfeaccb \dashv$
expand 6| $\dashv SeZd$ | $dfeaccb \dashv$
match $d$ | $\dashv SeZ$ | $feaccb \dashv$
expand 7| $\dashv Sef$ | $feaccb \dashv$
match $f$ | $\dashv Se$ | $eaccb \dashv$
match $e$ | $\dashv S$ | $accb \dashv$
expand 2| $\dashv bYa$ | $accb \dashv$
match $a$ | $\dashv bY$ | $ccb \dashv$
expand 4| $\dashv bYZcc$ | $ccb \dashv$
match $c$ | $\dashv bYZc$ | $cb \dashv$
match $c$ | $\dashv bYZ$ | $b \dashv$
expand 8| $\dashv bY$ | $b \dashv$
expand 5| $\dashv b$ | $b \dashv$
match $b$| $\dashv$ | $\dashv$
match $\dashv$|  | |
accept | ||

## Top Down Parsing

Action | Stack | Unread
---|---|---
init| $0$ | $\vdash ababaab \dashv$
S3| $0\vdash 3$ | $ababaab \dashv$
R3S10| $0\vdash 3X10$ | $ababaab \dashv$
S7| $0\vdash 3X10a7$ | $babaab \dashv$
R7S8| $0\vdash 3X10a7B8| $babaab \dashv$
S6| $0\vdash 3X10a7B8b6| $abaab \dashv$
R6S8| $0\vdash 3X10a7B8| $abaab \dashv$

## Error Checking

```c++
int wain(int* a, int *b) { //semantic error
    int c = 5;
    if (c !=== a) { // scanner error !===, c != a semantic error
        *(a + 4) = c;
    } // no else parse error
    return b; // return int* semantic error
}
```

## Code Generation

### Want to add $for$ loops
* Simple Version: 1 assignment, one test and 1 assignment in the increment position

1. Add new token **FOR for**
2. Add new rule 
    * statement ->FOR LPAREN assign COMMA test COMMA assign RPAREN LBRACE statement RBRACE
    * assign -> lvalue BECOMES expr
3. Type Check $\frac{well-typed(assign),well-typed(test),well-typed(assign), well-typed(statements)}{well-typed(for)}$
4. Code generation

```
code(assign)
startX:
code(test)
beq $3, $0, endX
code(statements)
code(assign)
beq $0, $0, startX
endX:
```

### Want to add bit shift operators

1. Add new token $LSHIFT <<$ $RSHIFT >>$
2. Add new rule
    * bitexpr -> expr
    * bitexpr -> bitexpr LSHIFT exprs
    * bitexpr -> bitexpr RSHIFT exprs
3. Type Check
4. Code generation

```
code(bitexpr)
push($3)
code(expr)
pop($5)
lis $2
.word 2
startX:
beq $3, $0, endX
mult $5, $2 ;multu ?
mflo $5
sub $3, $3, $11
beq $0, $0, startX
add $3, $5, $0
```

## Optimization

```c++
int foo(int x) {
    int y = 0;
    if ((y + 5) == 0) {
        y = 0;
    }
    else {
        y = 5;
    }
    return y;
}

int wain(int* a, int b) {
    int x = 5;
    int y = 5;
    int* z = NULL;
    z = &y;
    *z = 6;
    while (x != y) {
        x = x + 1;
        b = foo(x);
    }
    return b;
}
```

## Memory Management

Understand the technic