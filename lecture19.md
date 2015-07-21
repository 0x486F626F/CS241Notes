# Lecture 19

#### Problem:

You cannot know the offsets until all declarations have been processed, because \$30 changes with each new declaration.

```c++
int wain(int a, int b) { 
    int c = 0;
    return a; 
}
```

Name|Type|Offset from \$30
---|---|---
a|int|8
b|int|4
c|int|0

Introdue two conventions:

1. \$4 always contains 4
2. \$29 points to the **bottom** of the stack frame
    * If offsets calculated with respect to \$29, they will be constant
    
```c++
int wain(int a, int b) {
    int c = 0;
    return a;
}
```
```asm
lis $4
.word 4
sub $29, $30, $4
sw  $1, -4($30)
sw  $2, -8($30)
sw  $0, -12($30)
lis $5
.word 12
sub $30, $30, $12
lw  $3, 0($29)
add $30, $30, $5
jr  $31
```

Name|Type|Offset from \$29
---|---|---
a|int|0
b|int|-4
c|int|-8

$29 is called the frame pointer.

What about more complicated programs?

```c++
int wain(int a, int b) {
    return a + b;
}
```

In general: for every grammar rule $A\rightarrow\gamma$ build code($A$) from code($gamma$).

Extend previous convention: use \$3 as "output" for all expressions.

```
a + b
$3<-eval(a)
$3<-eval(a) -- value of a is lost!
$3<-$3 + $3
```

Need a place to store pending computation could use a reg:
```
code(a+b)=code(a)           ($3<-a)
          add $5, $3, $0    ($5<-a)
          code(b)           ($3<-b)
          add $3, $5, $3    ($3<-a+b)
```

What about $a+(b+c)$?

```
code(a+(b+c)) =	code(a)           ($3<-a)
				add $5, $3, $0    ($5<-a)
				code(b)           ($3<-b)
				add $6, $3, $0    ($3<-b)
			    code(c)           ($3<-c)
		        add $3, $6, $3    ($3<-(b+c))
		        add $3, $5, $3    ($3<-a+(b+c))
```

Similarly $a+(b+(c+d))$ would need $3$ extra registers. Eventually run out of registers

More general solution: use the stack

```
code(a) ($3<-a)
push($3)
code(b) ($3<-b)
push($3)
code(c) ($3<-c)
push($3)
code(d) ($3<-d)
pop($5) ($5<-c)
add $3, $5, $3  ($3<-c+d)
pop($5) ($5<-b)
add $3, $5, $3  ($3<-b+(c+d))
pop($5) ($5<-a)
add $3, $5, $3  ($3<-a+(b+(c+d)))
```

Only one extra register!

In general: $expr_1->expr_2+term$

```
code(expr_1) = code(expr_2)
             + push($3)
             + code($term$)
             + pop($5)
             + add $3, $5, $3
```

Singleton Rules: easy -- $expr->term$ 

code($expr$)=code($term$)

### println(expr)

print expr, followed by newline

You did this already! A2P6, A2P7a

Runtime environment: set of procedures supplied by the complier (or the OS) to assist programs in their execution

Procedure print will be part of the runtime environment. We will provide print.merl

```bash
./wlp4gen < prog.wlp4i > prog.asm
java cs241.linkasm < prog.asm > prog.merl
linker prog.merl print.merl > exec.mips
```

So assume "print" is available -- takes input int \$1

```
code(println(expr)) = code(expr)
                    + add $1, $3, $0
                    + call print
```

remember to save and restore \$31

### Assignment

```
statement->lvalue=expr
```

For now: assume lvalue is ID (no pointers yet)

```
statement->lvalue=expr
code(statement)=code(expr) ($3<-expr)
sw $3, offset in symbol table($29)
```

### If and While

* need boolean testing
    * Suggestion: store $1$ in \$11
    * store print in $10
    
code so far

Prologue

```asm
.import print
lis     $4
.word   4
lis     $10
.word   print
lis     $11
.word   1
sub     $29, $30, $4
```

Epilogue
```asm
add $30, $29, $4
jr  $31
```

```
test->expr1<expr2
code(test) = code(expr1)    ($3<-expr1)
           + add $6, $3, $0 ($6<-expr2)
           + code(expr2)    ($3<-expr2)
           + slt $3, $6, $0 ($3<-expr1<expr2)
test->expr1>expr2
    treat as expr2<expr1, as above        
```
