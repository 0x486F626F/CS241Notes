# Lecture 11

More complex example: L={cab}U{w in {a,b,c}^* | w contains an even number of a}
NFA:
![11-01](/pic/11-01.png)
Build DFA using the subset construction:
![11-02](/pic/11-02.png)
Accepting state: any state that includes an accepting state from the original NFA

Obvious: Every DFA can be converted to a DFA for the same language, so NFA and DFA accept tha same class of language (rugular).

### epsilon-NFA
What if we change states without reading a char -- epsilon-transition "free pass to a new state without reading a char". It makes it easier to "glue" smaller machines together.

Ex: L={cab}U{even number of a}
![11-03](/pic/11-03.png)

So every epsilon-NFA has an equivalent DFA, and the conversion can be automated.

If we can find an epsilon-NFA for every regular expression: we have one direction of Kleene's Theorem.

# Regular Expression Type
![11-04](/pic/11-04.png)

Therefor every regular expression has an equivalent (epsilon-NFA, NFA, DFA) and the conversion can be automated.
Thus, we can write a toll to convert regular expression to DFAs.

## Scanning (Lexical Analysis)
Is C regular? Keywords, IDs, literals, operators, comments, punctuations, are all regular, thus sequences of these are also regular.

So we can use Finite Automaton to do kokenization (scanning).

We need:
1. input string w
2. breake w into w1, w2, ..., wn such that wi \n L, else error
3. output each wi

Consider: L={valid C token} is regular. Let M(L) be the DFA that recognize L. Then M(L)^* recognize LL^* (non-empty dequences of tokens). Add an action to each epison-move: epison:output token.

Therefor, question: does this scheme guarantee a unique decomposition?

No, for example, abab could be interpreted as 1, 2, 3, 4 tokens.
