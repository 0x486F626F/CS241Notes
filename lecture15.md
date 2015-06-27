# Lecture 15

### Example
1. $S'\rightarrow\vdash S\dashv$
2. $S\rightarrow b S d$
3. $S\rightarrow p S q$
4. $S\rightarrow C$
5. $C\rightarrow lC$
6. $C\rightarrow\epsilon$


####$Nullable$

|Iteration|0|1|2|3|
|---|---|---|---|---|
|$S'$|false|false|false|false|
|$S$ |false|false|true |true |
|$C$ |false|true |true |true |


####$First$

|Iteration|0|1|2|3|
|---|---|---|---|---|
|$S'$|$\{\}$|$\{\vdash\}$|$\{\vdash\}$|$\{\vdash\}$|
|$S$ |$\{\}$|$\{b,p\}$   |$\{b,p,l\}$ |$\{b,p,l\}$ |
|$C$ |$\{\}$|$\{l\}$     |$\{l\}$     |$\{l\}$     |


####$Follow$

|Iteration|0|1|2|
|---|---|---|---|
|$S$|$\{\}$|$\{\dashv, d, q\}$|$\{\dashv, d, q\}$|
|$C$|$\{\}$|$\{\dashv, d, q\}$|$\{\dashv, d, q\}$|



####Predict:

$Predict(A,a)=\{A\rightarrow\beta |\beta\Rightarrow ^*\alpha\gamma\}\cup\{A\rightarrow\beta | Nullable(\beta), a\in Follow(A)\}$

| |$\vdash$|$\dashv$|$b$|$d$|$p$|$q$|$l$|
|---|---|---|---|---|---|---|---|
|$S'$|1| | | | | | |
|$S$ | |4|2|4|3|4|4|
|$C$ | |6| |6| |6|5|

#### A grammar is $LL(1)$ if
* no two distinct productions with the same LHS can generate the same first terminal symbol.
* no nullable symbol $A$ has the same terminal symbol $a$ in both its first set and its follow set.
* there is only one way to send a nullable symbol to $\epsilon$.


#### Example
1. $E\rightarrow E+T|T$    
2. $T\rightarrow T*F|F$    
3. $F\rightarrow id$    
This grammar is not $LL(1)$    
Why? - left recursion

$E\Rightarrow E+T\Rightarrow T+T\Rightarrow F+T\Rightarrow id+T$    
$E\Rightarrow T\Rightarrow F\Rightarrow id$   
id:same first symbol

Thus, left-recursion is always not $LL(1)$

Make it right-recursive

1. $E\rightarrow E+T|T$    
2. $T\rightarrow F*T|F$    
3. $F\rightarrow id$

This grammar is still not $LL(1)$   
Why? $T+E$ and $T$ generate the same first symbols.


#### Factor
1. $E\rightarrow TE'$
2. $E'\rightarrow \epsilon|+E$
3. $T\rightarrow FT'$
4. $T'\rightarrow \epsilon|*T$
5. $F\rightarrow id$

This grammar is $LL(1)$

$LL(1)$ conflicts with left-associativity.

## Bottom-Up Parsing

go from $w$ to $S$

Stack stores partially-reduced input read so far.

$w\Leftarrow\alpha_k\Leftarrow\alpha_{k-1}\Leftarrow ... \Leftarrow\alpha_1\Leftarrow\ S$

**Invariant**: stack and unread input = $alpha_i$ (or $w$ or $S$)

#### Example
1. $S'\rightarrow\vdash S\dashv$
2. $S\rightarrow AyB$
3. $A\rightarrow ab$
4. $A\rightarrow cd$
5. $B\rightarrow Z$
6. $B\rightarrow wx$

|Stack|         Read Input|           Unread Input|					Actions|
|---|---|---|---|
||					$\epsilon$|				$\vdash abywx\dashv$|  Shift $\vdash$|
|$\vdash$|			$\vdash$  |				$abywx\dashv$|  Shift $a$|
|$\vdash a$|		$\vdash a$  |			$bywx\dashv$|  Shift $b$|
|$\vdash ab$|		$\vdash ab$  |			$ywx\dashv$|  Reduce $A\rightarrow ab$; pop $b,a$, push $A$|
|$\vdash A$|		$\vdash ab$|			$\vdash ywx\dashv$|  Shift $y$|
|$\vdash Ay$|		$\vdash aby$  |			$wx\dashv$|  Shift $w$|
|$\vdash Ayw$|		$\vdash abw$  |			$x\dashv$|  Shift $x$|
|$\vdash Aywx$|		$\vdash abywx$  |		$\dashv$|  Reduce $B\rightarrow wx$; pop $x,w$, push $B$|
|$\vdash AyB$|		$\vdash abywx$  |		$\dashv$|  Reduce $S\rightarrow AyB$; pop $B,y,A$, push $S$|
|$\vdash S$|		$\vdash abywx$  |		$\dashv$| Shift $\dashv$|
|$\vdash S \dashv$| $\vdash abywx\dashv$|	$\epsilon$| Reduce $S'\rightarrow\vdash S\dashv$, pop $S$, push $S'$|
|$\vdash S \dashv$| $\vdash abywx\dashv$|	$\epsilon$|	Accept|

Chice at eacht step:
1. Shift a character from input to stack
2. Reduce the top of stack is the RHS of a grammar rule - replace with LHS.

Accept if stack contains $S'$ when input is $\epsilon$    
(Equivalent $\vdash S\dashv$ on empty input)     
(Equivalent accept when shift $\dashv$)

How do we know whether to shift or reduce?

* use next character to help decide
* problem still hard
