# Lecture 14

## Top-down Parsing

Start $S\Rightarrow\alpha_1\Rightarrow\alpha_2\Rightarrow ...\Rightarrow w$, 
use the stack to store in intermediate $\alpha_i$ in reverse, match against characters in $w$.

#### Invariant
Consumed input + reverse (stack contents) = $\alpha_i$

#### Example
$S\rightarrow A\enspace y\enspace B$    
$A\rightarrow ab$   
$A\rightarrow cd$   
$B\rightarrow z$   
$B\rightarrow wx$   

For simplicity, we use a **augmented grammars** for parsing, i.e.. invent two new symbols, $\vdash$, $\dashv$, new start symbol $S'$

$S'\rightarrow \vdash S\dashv$    
$S\rightarrow A\enspace y\enspace B$    
$A\rightarrow ab$   
$A\rightarrow cd$   
$B\rightarrow z$   
$B\rightarrow wx$  

say $w=\vdash abywx\dashv$

|Stack				|Read Input				|UnreadInput				|Action|
|---|---|---|---|
|$S'$				|$\epsilon$				|$\vdash abywx\dashv$		|pop $S'$; push $\dashv, S, \vdash$|
|$\dashv S\vdash$	|$\epsilon$				|$\vdash abywx\dashv$		|match $\vdash$|
|$\dashv S$			|$\vdash$				|$abywx\dashv$				|pop $S$; push $B,y,A$|
|$\dashv ByA$		|$\vdash$				|$abywx\dashv$				|pop $A$; push $b,a$|
|$\dashv Byba$		|$\vdash$				|$abywx\dashv$				|match $a$|
|$\dashv Byb$		|$\vdash a$				|$bywx\dashv$				|match $b$|
|$\dashv By$		|$\vdash ab$			|$ywx\dashv$				|match $y$|
|$\dashv B$			|$\vdash aby$			|$wx\dashv$					|pop $B$, push $x,w$|
|$\dashv xw$		|$\vdash aby$			|$wx\dashv$					|match $w$|
|$\dashv x$			|$\vdash abyw$			|$x\dashv$					|match $x$|
|$\dashv$			|$\vdash abywx$			|$\dashv$					|match $\dashv$|
|					|$\vdash abywx\dashv$	|$\epsilon$					|Accept|

* When top of stack is a terminal, pop and match against input.   
* When top of stack is a non-terminal $A$, pop $A$ and push $\alpha ^R$ ($\alpha$ reversed), where $A\rightarrow\alpha$ is a gramma rule.    
* Accept when stack and input both empty.

But what if there is $>1$ production with $A$ on the LHS? How can we know which one to pick?

* Brute force (try all combinations until one works) is inefficient.
* Want a deterministic procedure.

**Our solution**: use the next symbol of input (look ahead) to help decide.

Construct a **predictor table**: given non-terminal on stack and input symbol, tell which production to use.

1. $S'\rightarrow \vdash S\dashv$    
2. $S\rightarrow A\enspace y\enspace B$    
3. $A\rightarrow ab$   
4. $A\rightarrow cd$   
5. $B\rightarrow z$   
6. $B\rightarrow wx$  

|    |$\vdash$|$a$|$b$|$c$|$d$|$w$|$x$|$y$|$z$|$\dashv$|
|---|---|---|---|---|---|---|---|---|---|---|
|$S'$|1||||||||||
|$S$ ||2||2|||||||
|$A$ ||3||4|||||||
|$B$ ||||||6|||5||

Empty cell = parse error.

Descriptive: Parse error at row, column: expecting one of ... (characters for which current top of stack has entries)    
e.g. if top of stack is $A$, "Expecting $a$ or $c$"

What if a cell contains more that one entry? Method breaks down.

A grammar is called $LL(1)$ if each cell of the predictor table has at most one entry.

$LL(1)=$left-to-right scan of input, leftmost derivations produced, 1 symbols lookahead.

### Automatically Computing the Predictor Table
$Predict(A, a)$ - rules that apply when $A$ on the stack, $a$ is next input character.

$Predict(A, a)=\{A\rightarrow\beta | a \in First(\beta)\}$

$First(\beta)$ - $\beta\in V ^*$ - set of characters that can be the first letter of derivation starting from $\beta$.

$First(\beta)=\{a|\beta\Rightarrow ^*a\gamma\}$

So $Predict(A,a)=\{A\rightarrow\beta |\beta\Rightarrow ^*\alpha\gamma\}$

* not quite right
* what if $A\Rightarrow ^*\epsilon$ then $a$ might not come from $A$, but from something after $A$.

So $Predict(A,a)=\{A\rightarrow\beta |\beta\Rightarrow ^*\alpha\gamma\}\cup\{A\rightarrow\beta | Nullable(\beta), a\in Follow(A)\}$

$Nullable(B)=true\enspace iff\enspace \beta\Rightarrow ^*\epsilon$    
$Follow(A)=\{b|S'\Rightarrow ^* \alpha Ab\beta\}$


### Computing $Nullable$

initialize $Nullable[A]$=false $\forall A$    
repeat    
$\quad$ for each rule $B\rightarrow B_1 ... B_k$      
$\quad\quad$ if $k=0$ or $Nullable[B_i]\forall B_i$    
$\quad\quad\quad$ $Nullalbe[B]\leftarrow$ true.    
until nothing changes

### Computing $First$

initialize $First[A]=\{\}$ $\forall A$    
repeat    
$\quad$ for each rule $B\rightarrow B_1 ... B_k$      
$\quad$ for $i=1 ... k$      
$\quad\quad\quad$ if $B_i$ is a terminal $a$    
$\quad\quad\quad\quad$ $First[B]\cup=\{a\}$ break   
$\quad\quad\quad$ else    
$\quad\quad\quad\quad$ $First[B]\cup=First[B_i]$    
$\quad\quad\quad\quad$ if not $Nullable[B_i]$ break    
until nothing changes

### Computing $First ^*(\beta)$

First set for a string of symbols

$First ^*(Y_1, ..., Y_n)$     
$\quad$ result$\leftarrow\emptyset$    
$\quad$ for $i=1...n$    
$\quad\quad$ if $Y_i\not\in\Sigma$ (i.e. non-terminal)   
$\quad\quad\quad$ result $\cup=First[Y_i]$   
$\quad\quad\quad$ if not $Nullable[Y_i]$ break   
$\quad\quad$ else (terminal)    
$\quad\quad\quad$ result $\cup=\{Y_i\}$; break;    
$\quad$ return result    

### Computing $Follow$

initialize $Follow[A]=\{\}$ $\forall A\neq S'$  
repeat    
$\quad$ for each rule $B\rightarrow B_1...B_n$    
$\quad\quad$ for $i=1...n$    
$\quad\quad\quad$ if $B_i\in N$    
$\quad\quad\quad\quad$ $Follow[B_i]\cup=First ^*(B_{i+1}...B_n)$   
$\quad\quad\quad\quad$ if all $B_{i+1}...B_n$ nullable (include i=n)    
$\quad\quad\quad\quad\quad$ $Follow[B_i]\cup =Follow[B]$   
until nothing changes
