# Lecture 17

## Context-Sensitive Analysis

also called **semantic analysis**

What properties of valid programs cannot be enforced by Context-Free Grammars?

* declaration before use
* scope correctness
* type checking
* etc.

To solve these, need more complex programs.

Next language class is **context-sensitive languages** (specified by context-sensitive grammars)

Ad hoc approach actually more useful.

* traverse the parse tree.

e.g. Parse Tree
```c++
class Tree {
    public:
        string rule;                //grammar rule e.g. "term id"
        vector <Tree*> children;
};
```

Traversal example: print the tree
```c++
void print(Tree &t) {
    //print t.rule
    for (vector <Tree*>::iterator i = t.begin(); i != t.end(); i ++)
        print(**i);
}
```

What semantic analysis is needed for WLP4? What error need to be caught?

* variables and procedures used but not declared
* variables and procedures declared more than once
* type errors

For now, assume **wain** is the only procedure.

### Declaration errors

multiple/missing declaration.

Construct a symbol table.

* traverse the parse tree to collect all declarations. 
    * For each node corresponding to rule "dcl type ID", extract name (e.g. "x") and type ("int" or "int*") and add to symbol table
    * If the name already in symbol table, ERROR.
* multiple declarations now checked.
* Traverse again
    * Check for rules of the form "factor ID" and "lvalue ID"
    * undeclared variable now checked.
    
These two passes could be merged.

Symbol table implementation

global variable

```c++
map<string, string> symbol_table; (name and type)
```

But

* does not take scope into account
* does not take procedure declarations

Issue:

```c++
int f() {
    int x = 0;
    return 1;
}
int wain(int a, int b) {
    int x = 0;
    return 1;
}
```
According to the algorithm, $x$ is declared twice. But this should be OK.

Must forbid duplicated definition in the procedure, but allow them in different procedures.

Also forbid this:

```c++
int f() {}
int f() {}
```

Solution: 
* have a separate symbol table for each procedure
* have one top level symbol table that collects all procedure names.

```c++
map <string, map <string, string> > top_symbol_table; //(procedure name, procedure's symbol table)
```

When traversing:

* when rule is "procedure ..." or "main ..." is a new procedure
    * make sure its name is not already in symbol table
    * if not, create a new entry
* may want a global variable **current_procedure" tells you which procedure you are currently in, and keep it up to date.

For variables, we store the declared type in the symbol table. 

Is there type into for procedures? Yes, signature.

Note: 

* all procedures in WLP4 return int, so the signature is the sequence of parameter types
* store signature in top-level table

```c++
map <string, pair <vector <string>, map <string, string> > > top_symbol_table; //(procedure name, (signature, local symbol table))
```

Top compute the signature:

* traverse nodes of the form "paramalist dcl" and "paramlist dcl COMMA pramalist"
* if "pramalist", then signature is empty

All of this can be done in one pass.

## Types

Why do programming languages have types?

* prevent errors
* allow us to assign an interpretation to the contents of some address and to remember the interpretation.

e.g.
```c++
int *a = NULL;  //denotes a pointer
a = 7;          //error, attempt store an int where a pointer is needed.
```

To check type correctness

* find a type for each variable/expression.
* ensure the "rules" are applied properly.

Tree traversal
```
string typeof(Tree &t) {
    for each c in t.children
        compute typeof(c)
    determine typeof(t) from types of children, based on t.rule
```

e.g. ID - get its type from symbol table

$$\frac{<id.name,\tau>\in declaration}{id:\tau}$$

```c++
string typeof(Tree &t) {
    if (t.rule == "ID name"
        return symbol_table.lookup(name);
    ...
}
```

