# Lecture 18

### Singleton Productions

#### Type of LHS = type of RHS

$expr\rightarrow term$    
$term\rightarrow factor$   
$factor\rightarrow id$   

$$\frac{}{NUM:int}\quad\frac{}{NULL:int*}$$

If $E$ has type $\tau$, then $(E)$ has type $\tau$.

$$\frac{E:\tau}{(E):\tau}$$

#### Address of

$$\frac{E:int}{\& E:int*}$$

#### Dereferece

$$\frac{E:int*}{*E:int}$$

#### Allocation

$$\frac{E:int}{new\enspace int\enspace [E]:int*}$$

#### Multiplication

$$\frac{E_1:int\enspace E_2:int}{E_1*E_2:int}$$

Similar for $/,\%$

#### Addition

$$\frac{E_1:int\enspace E_2:int}{E_1+E_2:int}$$
$$\frac{E_1:int*\enspace E_2:int}{E_1+E_2:int*}$$
$$\frac{E_1:int\enspace E_2:int*}{E_1+E_2:int*}$$

#### Subtraction

$$\frac{E_1:int\enspace E_2:int}{E_1-E_2:int}$$
$$\frac{E_1:int*\enspace E_2:int}{E_1-E_2:int*}$$
$$\frac{E_1:int*\enspace E_2:int*}{E_1-E_2:int}$$

#### Function Call

$$\frac{<f,(\tau_1,...,\tau_n)>\in procdure\enspace E_1:\tau_1...E_n:\tau_n}{f(E_1,...,E_n):int}$$

### Additinal Type Constraints

loops, if statements

```
while (T) {S}
if (T) {S1} else {S2}
```

Test $T$ should be boolean, not $int$ or $int*$.
But there is no boolean type in WLP4.

The grammar ensures that $T$ will be a boolean test (expr comparison expr), so as long as the comparison is well-typed, the if/while will be correct.

Any expression that has a type is well-typed.

$$\frac{E:\tau}{well-typed(E)}$$

##### Test

$$\frac{E_1:\tau\enspace E_2:\tau}{well-typed(E_1==E_2)}$$

Similarly $!=$

$$\frac{E_1:\tau\enspace E_2:\tau}{well-typed(E_1<E_2)}$$

Similarly $<=,>,>=$

#### Assignment

$$\frac{E_1:\tau\enspace E_2:\tau}{well-typed(E_1=E_2)}$$

#### Print

$$\frac{E:int}{well-typed(println(E))}$$

#### Deallocation

$$\frac{E:int*}{well-typed(delete [] E)}$$

#### If Statement

$$\frac{well-typed(T)\enspace well-typed(S_1)\enspace well-typed(S_2)}{well-typed(if(T)\{S_1\}else\{S_2\})}$$


#### While Loop

$$\frac{well-typed(T)\enspace well-typed(S)}{well-typed(while(T)\{S\})}$$

#### Sequences

empty sequence

$$\frac{}{well-typed(E)}$$

$$\frac{well-typed(S_1)\enspace well-typed(S_2)}{well-typed(S_1S_2)}$$

#### Declaration

$$\frac{}{well-typed(int\enspace id=NUM)}$$

$$\frac{}{well-typed(int*\enspace id=NULL)}$$

#### Procedures

$$\frac{well-typed(dcls)\enspace well-typed(statement)\enspace E:int}{well-typed(INT\enspace ID(params)\{dcls\enspace statements\enspace return\enspace E;\})}$$

#### wain

$$\frac{dcl2=INT\enspace ID\enspace well-typed(dcls)\enspace well-typed(statement)\enspace E:int}{well-typed(INT\enspace wain(dcl1,dcl2)\{dcls\enspace statements\enspace return\enspace E;\})}$$

END OF SEMANTIC ANALYSIS

## Code Generations

```
         ________________        _______             _________________ Parse Tree    _______________ 
source->|lexical Analysis|----->|Parsing|---------->|Semantic Analysis|------------>|Code Generation|->Assembly
        |________________|Token |_______|Parse Tree |_________________|Symbol Table |_______________|
```

* By now, the source program is guaranteed to be free of compile-time errors
* Must output equivalent MIPS assembly code.

There are infinity many equivalent MIPS programs. Which do we choose?

* Correct (essential)
* Easiest (for CS241)
* Shortest
* Fastest to run
* Fastest to compile

### Convention
* parameters of wain are in \$1 and \$2
* output will be in \$3

E.g.
```c++
int wain (int a, int b) { return a; }
```

```asm
add $3, $1, $0
jr  $31
```

```c++
int wain (int a, int b) { return b; }
```

```asm
add $3, $2, $0
jr  $31
```

Parse trees for these programs are the same. 
How do you tell one from the other?   
Need to determine which of the two declared IDs the returned ID matches.

How do we do that? Use the symbol table.

Add a field to the symbol table to indicate where each symbol is stored

Name|Type|Location
---|---|---
a|int|\$1
b|int|\$2

Then, where traversing for code generations, look up ID in the symbol table.

### Local Declarations

cannot all be in registers (probably not enough)

For simplicity:

* put all local variables on the stack, including parameters of wain.

```c++
int wain(int a, int b) { return a; }
```
```asm
sw    $1, -4($30)
sw    $2, -8($30)
lis   $4
.word 8
sub   $30, $30, $4
lw    $3, 4($30)
add   $30, $30, $4
jr    $31
```

Name|Type|Offset from $30
---|---|---
a|int|4
b|int|0
