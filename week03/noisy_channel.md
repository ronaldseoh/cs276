# More on Noisy Channel (From Lecture 5 Slides)

## Independent Word Spelling Correction

- The goal: Find the intended word, given a word where the letters have been scrambled in some manner (quoted from Wikipedia)
- Noisy model = Bayes' Rule
    - $\hat{w} = arg\,max_{w \in V} P(w \mid x) = arg\,max_{w \in V} \frac{P(x \mid w) P(w)}{P(x)} = arg\,max_{w \in V} P(x \mid w) P(w)$
- Candidate generation
    - Words with similar spelling: short edit distance
        - 80% of errors are within edit distance 1
        - Almost all errors are with edit distance 2
        - Allow inserting space or hyphen
        - Allow merging words
    - Words with similar pronunciation
- What's $P(w)$?
    - Language Model: With a big supply of words (A document collection with $T$ tokens), let $P(w)=\frac{C(w)}{T}$, where $C(w)$ is the total occurrence of $w$ in the collection.
    - In other collections, we can simply consider the supply to be the query typed in.
- $P(x \mid w)$: Probability of the *edit* (deletion, insertion, substitution, transposition)
- Channel Model:
    - Confusion matrix for substitution
        - $\text{del}\lbrack x, y \rbrack$: Count $xy$ typed as $x$
        - $\text{ins}\lbrack x, y \rbrack$: Count $x$ typed as $x$
    - If deletion, $P(x \mid w) = \frac{\text{del}\lbrack w_{i-1}, w_i \rbrack}{\text{count}\lbrack w_{i-1} w_i \rbrack}