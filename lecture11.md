# Lecture 11

More complex example: L={cab}U{w in {a,b,c}^* | w contains an even number of a}
NFA:
![11-01](/pic/11-01.png)
Build DFA using the subset construction:
![11-02](/pic/11-02.png)
Accepting state: any state that includes an accepting state from the original NFA

Obvious: Every DFA can be converted to a DFA for the same language, so NFA and DFA accept tha same class of language (rugular).

### empty-NFA
What if we change states without reading a char -- empty-transition "free pass to a new state without reading a char". It makes it easier to "glue" smaller machines together.

Ex: L={cab}U{even number of a}

