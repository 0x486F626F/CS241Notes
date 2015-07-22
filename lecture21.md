# Lecture 21

Add to the prologue

```asm
.import init
.import new
.import delete
```

Function init

* sets up allocator's data structures
* call it once in the prologue.

Takes a parameter in $2.

* If calling with mips.array (i.e. int wain(int*, int)), $2=length of the array.
* else $2 = 0

### New and Delete

Functions in alloc.merl

new: $1 = number of words needed.  

* returns pointer to memory in $3. 
* if allocation fails, returns 0.

```
code(new int [expr]) =  code(expr) ($3=expr)
                        add $1, $3, $0
                        call new
                        bne $3, $0, $1
                        add $3, $11, 0
```

delete: $1 = pointer to memory to be deallocated.

```
code(delete [] expr) =  code(expr)
                        beq $3, $11, skipY
                        add $1, $3, $0
                        call delete
                        skipY:
```

## Procedures

Big Picture

```
int f() {}
int g() {}
...
int wain() {}
```

```
lis $5
.word $5
jr $5
f:
g:
wain:
```

Main prologue/Epilogue

* save $1, $2 on stack
* import print, init, new, delete
* set $4, $11, etc. if desired
* set $29
* call init
* ...
* reset stack and jr $31

Procedure-specific prologues

* Do not need imports, set constants
* set $29
* save registers

#### Saving and Restoring

* procedure should save and restore all registers it will modify.
* How do you know which register
    * if now sure, save and restore all of them.
    * our code generation scheme does not use any register past $6 (and $29-$31)
    * if your scheme uses more -- OK, but need to keep track of register used
* Do not forget to save and restore $29

Two approaches to saving and restoring -- caller-save and calle-save. Suppose $f$ calls $g$:

* callee-save: $g$ saves all registers it modifies. $f$ does not worry about what g does
* caller-save: $f$ saves all registers that contain critical data before calling $g$. $g$ does not worry about its registers.

Our approach has been:

* caller-save for $31
* callee-save for everything else.

Who saves $29? caller or callee?

If callee-save

* save $29 along with other registers. 
* If you have not yet set $29, need to count back in the stack to find the beginning of the frame.
* Maybe easier to set $29 first and then save registers.
    * cannot set $29 before saving it.
    * so at least save $29 first
    * then set $29
    * then save other registers
    
Or let the caller ($f) save $29 before calling $g$

```
f: ...
push($29)
push($31)
lis $5
.word g
jalr $5
pop($31)
pop($29)
```

Procedure names match the names of existing labels. It won't assemble.

Use a naming scheme for labels that prevents duplication.

* For functions, $f,g,h$ use the labels $Ff, Fg, Fh$ i.g. reserve labels starting with Fas denoting functions.
* Then make sure all compiler-generated labels do not start with F.

### Parameters

Could not use registers -- may not be enough

```
factor -> ID(expr1, ..., exprn)
code (factor) = push($29)
                push($30)
                code(expr1)
                push($3)
                ...
                code(exprn)
                push($3)
                lis $5
                .word ____
                jalr $5
                ; pop all args
                pop($31)
                pop($29)
code(procedure) =   sub $29, $30, $4
                    push(registers)
                    code(dcls)
                    code(statements)
                    code(expr)
                    pop(registers)
                    add $30, $30, $4
                    jr $31
```

What does the stack look like?

```
...
|__________|
|local regs|-$30
|__________|
|saved regs|-$29
|__________|____
|args for g|f's frame
|__________|
|   $31    |
|__________|
|   $29    |
|__________|____
...


Suppose $g$ is 

```
int g(int a, int b, int c) {
int d = 0; int e = 0; int f = 0;
...
}
```

g's symbol table:

Name | Type | Offset
---|---|---
a | int | 0
b | int | -4
c | int | -8
d | int | -12
e | int | -16
f | int | -20

Parameters below $29, local variables above $29.

In between are saved registers, so symbol table offsets are wrong.

* Parameters should have positive offsets
* Local variable should have negative offsets

Fix symbol table: 

* add 4*number of arguments to all offsets in symbol table.
* Push local variable first, then save registers

...
|__________|
|saved regs|-$30
|__________|
|local regs|-$29
|__________|____
|args for g|f's frame
|__________|
|   $31    |
|__________|
|   $29    |
|__________|____
...

Name | Type | Offset
---|---|---
a | int | 12
b | int | 8
c | int | 4
d | int | 0
e | int | -4
f | int | -8