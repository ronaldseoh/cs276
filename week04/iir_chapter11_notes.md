# Chapter 11: Probabilistic Information Retrieval

- Estimating the probability of a term `t` appearing in a relevant document `P(t | R = 1)`
- Users' *information needs* -> translated into *query representations*
- Documents -> *Document representation*
- In the Boolean or vector space models, matching is done in a formally defined but *semantically imprecise calculus* of index terms: *Uncertain guesses*
- Probability provides a principled foundation for reasoning under uncertainty

## 11.1 Review of basic probability theory

- A little basic probability theory

## 11.2 The Probability Ranking Principle

### 11.2.1 The 1/0 loss case

- `R_{d,q}`: An indicator random variable that says whether `d` is *relevant* with respect to a given query `q`.
- **Probability Ranking Principle (PRP)**: Rank documents by their estimated *probabilities* of relevance: `P(R=1 | d, q)`
- *1/0 Loss*: In the simplest case of PRP, you simply lose a point for returning a non-relevant document or failing to return any relevant document.
- *Bayes Optimal Decision Rule*: `d` is relevant iff `P(R = 1 | d, q) > P(R = 0 | d, q)`
    - Theorem: The PRP is optimal, in the sense that it minimizes the expected loss (aka the *Bayes risk*) under 1/0 loss.
        - Requires that all probabilities are known *correctly*, which is rarely the case in practice

