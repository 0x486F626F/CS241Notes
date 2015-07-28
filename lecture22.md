# Lecture 22

Or save all registers in caller

```
|__________|
|local regs|-$30
|__________|____
|args for g|f's frame
|__________|
|   $31    |
|__________|
|   $29    |
|__________|
|saved regs|
|__________|____
```

But
```
f() { g(); g(); g(); }
```

* callee-save: save registers once per function
* caller-save: save registers once per call

Summary
```
factor->ID(expr1, ..., exprn)
code(factor) =  push($29)
                push($31)
                code(expr1)
                push($3)
                ...
                code(exprn)
                push($3)
                lis $5
                .word Function ID
                jalr $5
code(procedure) =   sub $29, $30, $4
                    push(decls)
                    push(registers)
                    code(statements)
                    code(expr)
                    pop registers
                    add $30, $29, $4
                    jr $31
```

## Optimization

Computationally unsolvable.

### Constant Folding

E.g. $1+2$
```asm
lis     $3
.word   1
sw      $3, -4($30)
sub     $30, $30, $4
lis     $3
.word   2
lw      $5, 0($30)
add     $30, $30, $4
add     $3, $5, $3
```

```asm
lis     $3
.word   3
```

### Constant Propagation

```c++
int x=1
x+x
```

```
lis     $3
.word   1
sw      $3, -4($30)
sub     $30, $30, $4
lw      $3, offset($29)
sw      $3, -4($30)
sub     $30, $30, $4
lw      $3, offset($29)
lw      $5, 0($30)
add     $30, $30, $4
add     $3, $5, $3
```

Could recognize that $x=1$ and do

```
lis     $3
.word   1
sw      $3, -4($30)
sub     $30, $30, $4
lis     $3
.word   2
```

If that's the only place $x$ is sued, it does not need a stack entry.

Watch out for **aliasing**

```c++
int x=1;
int y=NULL;
y=&x;
*y=2;
x+x;
```

Even if x's value is not known, could recognize that $3 already contains x:

```
lw      $3, -12($29)
add     $3, $3, $3
```

Common subexpression elimination $(a+b)*(a+b)$

use a register to hold $a+b$, then multiply it by itself, rather than compute $a+b$ twice.

### Dead Code Elimination

If you are certain that some branch of a program will never run, do not output code for it.

### Register Allocation

* cheaper to use registers for variable instead of stack.
* register $12-$29 unused by our code generations, could use these to hold 17 variable, use stack for the rest.

Which 17 variables?

* probably the most used ones.
* one registers may be used for more than one variable. (out of scope)

Issue: address of

* If you take the address of a variable, it cannot be in a register.

### Strength Reduction

Add usually runs faster than mult, so prefer add over mult

```
lis     $2
.word   2
mult    $3, $2
mflo    $3

add     $3, $3, $3
```

### Procedure-specific Optimizations

#### Inlining

```c++
int f(int x) { return x + x; }
int wain(int a, int b) { return f(a); }

int wain(int a, int b) { return a + a; }
```

* Replace the function call with its body, right in the caller.
* Saves the overhead of a function call
* Is it always a win?
    * If $f$ is called many times, get many copies of its body.
    * Which is bigger -- body of $f$ or code to call $f$
* Some functions are hard to inline -- recursive functions.
* If all calls to f are inlined, do not need code generation for $f$.

#### Tail Recursion

```c++
int fact(int n, int a) {
    if (n == 0) return a;
    else return fib(n - 1, n * a);
}
```

Recursive call is the last thing the function does. No pending work to do when recursive call returns

Contents of current frame (locals, etc.) will not be used again, so reuse the current frame.

Can this be done in WLP4?

* No, because return only at end, no return inside if.
* But we can transform the program

Basic Transformation:

* When return immediately follows an if statement, push inside both branches.

```c++
if () {
    if () {
    }
    else {
    }
else {
}
return x;

if () {
    if () {
        return x;
    }
    else {
        return x;
    }
else {
    return x;
}
```

* Whenever return $x$ follows an assign to $x$, merge:

```
x=f();
return x;

return f();
```

may create some tail recursinve calls.

Generalization: tail call optimization

When a function's last action is any function call (recursive or not), can reuse the stack frame.

# WLP4?

* SL (small language)
* WL (water language)
* WLPP (WL plus pointers)
* WLPPPP (WL plus pointers plus procedures)
