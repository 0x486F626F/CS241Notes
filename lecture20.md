# Lecture 20

```
test->expr1 != expr2
code(test) = code(expr1)    ($3<-expr1)
           + add $6, $3, $0 ($6<-expr2)
           + code(expr2)    ($3<-expr2)
           slt $5, $3, $6   ($5<-$3<$6)
           slt $6, $6, $3   ($6<-$6>$3)
           add $3, $5, $6   ($3<-$6+$7)
test->expr1 == expr2
treat as NOT(expr1 != expr2)
           sub $3, $11, $3
```

### If
```
code(test) =    code(test)     ($3<-test)
                + beq $3, $0, else
                + code(statement1)
                + beq $0, $0, endif
                else:
                + code(statement2)
                endif:
```

Need to generate unique labels.

* keep a counter X
* use elseX, endifX, trueX for label names
* increment X for each new if statement

### While
```
code(test) =    loopY:
                code(test)     ($3<-test)
                + beq $3, $0, doneY
                + code(statement)
                + beq $0, $0, loopY
                doneY:
```

Advice: Generate comments along with MIPS instructions. (aids debugging)

## Pointers

Need to support the following:

* NULL
* dereference
* pointer comparison
* pointer arithmetics
* address of
* allocation/deallocation
* assignment through pointers

### NULL

* could use 0
* use 1 - if this is dereferenced, machine will crash.

```
factor->NULL
    code(factor) = $3, $11, $0  ($3<-1)
```

### Dereference
```
factor->*expr
    code(factor) =  code(expr)  ($3<-expr)
                    lw $3, 0($3)
```

### Comparison

Same as int comparisons, except

* no negative pointers
* pointer comparison should be unsigned i.e. sltu instead of slt

so when generating code for 

```
test->expr1 != expr2
        use slt  if expr1, expr2: int
        use sltu if expr1, expr2: int*
```

re-run "typeof" on expr to recompute the types.

Better:

* add a "type" field to each tree node.
* make "typeof" procedure record each node's type if it has one in the node itself.
* then the type will be available if needed.

### Arithmetics

```
expr1->expr2+term
expr1->expr2-term
```

exact meaning depends on types involved.

#### Addition $expr_1->expr_2+term$

```
expr2:int  term:int  - as before
expr2:int* term:int  - expr2+4*term
    code(expr1) =   code(expr2)
                    push($3)
                    code(term)
                    pop($3)
                    mult $3, $4
                    mflo $3
                    add $3, $5, $3
expr2:int  term:int* - 4*expr2+term
    code(expr1) =   code(expr2)
                    push($3)
                    code(term)
                    pop($3)
                    mult $5, $4
                    mflo $5
                    add $3, $5, $3
```

#### Subtraction $expr_1->expr_2-term$

```
expr2:int  term:int  - as before
expr2:int* term:int  - expr2-4*term
    code(expr1) =   code(expr2)
                    push($3)
                    code(term)
                    pop($3)
                    mult $3, $4
                    mflo $3
                    sub $3, $5, $3
expr2:int* term:int* - (expr2-term)/4
    code(expr1) =   code(expr2)
                    push($3)
                    code(term)
                    pop($3)
                    sub $3, $5, $3
                    div $3, $4
                    mflo $3                    
```

#### Assignment through pointer dereference

LHS = address at which to store the value   
RHS = value

```
statement-> ID = expr    - already done
            *factor = expr

code(statement) =   code(expr)
                    push($3)
                    code(expr)
                    pop($5)
                    sw $3, 0($5)
```

#### Address of $&lvalue$

2 cases ID, STAR factor

if expr=ID

```
code(&ID) = lis $3
            .word ____;look up ID in symbol table
            add $3, $3, $29
```

if expr=*factor
```
code(&*factor) = code(factor)
```

#### New and Delete

* runtime environment
* allocation module alloc.merl
* link to your assembled output
* must be linked last


