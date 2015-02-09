# Lecture 10

## Formal Definition of DFA
A DFA is a 5-tuple(Q, Sigma, q_0, A, delta) where
* Sigma is a finite non-empty set (alphabet)
* Q is a finite non-empty set (states)
* q_0 in Q (start state)
* A in Q (accepting state)
* delta: Q * Sigma -> Q (transition function: state to input char->state)

delta consume a single char, extend delta to a function that consumes an entire word.
```
delta^*: Q * Sigma^* -> Q (state word ->state)
delta^*(q, empty) = q
delta^*(q, cw)=delta^*(delta(q, c), w)
```
We say a DFA M=(Q, Sigma, q_0, A, delta) accepts a word w if delta^*(q_0,w) in A.

If M is a DFA, we denote by L(m) (the laaanguage of mM) the set of all strings accepted by M: L(M)={w in Singma^*|M accepts w}

## Kleene Theorem
L is regular iff L=L(M) for some DFA M. (The regular languages are exactly the languages accepted by DFAs)

## Implementing a DFA
```
int state = q_0
char ch
while not EOF do
	read ch
	case state of
	q_0: case ch of
		a: state ...
		b: state ...
	q_1: case ch of
		a: state ...
		b: state ...
	...
	q_n: case ch of
		a: state ...
		b: state ...
end while
return state in A
```
Or: use a lookup table

## DFAs with actions
Can attach computations to the arcs of a DFA

e.g. L={binary numbers with no leading 0s} - compute the value of the number - "1(0|1)*|0"
```
            __
           |  |
   _ 1:N=1 _  |0:N=N*2
->|_|-----|_|-|
   |       |__|1:N=N*2+1
   _0:N=0
  |_|
```

## Non-deterministic Finite Automaton
What if we allowed more than one arc from the same state with the same label? What would this mean?

Machine chooses one direction or the other (i.e. the machine is non-deterministic), accepts if **some** set of choices leads to an accepting state. 

NFAs often simpler that DFA. Formally NFA is a 5-tuple (Q, Sigma, q_0, A, delta)
* Sigma is a finite non-empty set (alphabet)
* Q is a finite non-empty set (states)
* q_0 in Q (start state)
* A in Q (accepting state)
* delta: Q * Sigma -> subset of Q

2^Q Non-deterministic want to accept if some path through the NFA leads to acceptance for the giben word.

delta^* for NFAs: set of states * Simge^* -> set of states
```
delta^*:2^Q * Sigma^* -> 2^Q
delta^*(q's, empty)=q's
delta^*(q's, cw)=delta^*(U_{q in q's}(delta(q, c)), w)
```
Then accept w if delta^*({q_0}, w) intersects A = empty 

#### Example
```
   _ a _ b _ b _
->|_|-|_|-|_|-|_|
   |_|
   a,b
```
Already read input | Unread input | States
---|---|---
 | baabb | {1}
b | aabb | {1}
ba | abb | {1, 2}
baa | bb | {1, 2}
baab | b | {1, 3}
baabb |  | {1, 4}

If you give each set of states a name, and call these the states, every NFA become a DFA.
