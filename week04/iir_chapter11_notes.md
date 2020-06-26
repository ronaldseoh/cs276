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

### 11.2.2 The PRP with retrieval costs

- Now consider additional retrieval costs associated with relevant and non-relvant documents.
- `C_1`: The cost of *not* retrieving a relevant document
- `C_0`: The cost of retrieving a *non-relevant* document
- According to the PRP, if `C_0 * P(R=0 | d) - C_1 * P(R=1 | d) <= C_0 * P(R=0 | d') - C_1 * P(R=1 | d')`, for a specific document `d` and for all documents `d'` not yet retrieved,
    - Then `d` should be the next document to be retrieved.
    - This allow us to model differential costs of false positives and false negatives at modelling stage, rather than considering them at *evaluation* stage.

## 11.3 The Binary Independence Model

- Documents and queries are both represented as binary term incidence vectors.
- A document `d` is represented by the vector `\vec{x} = (x_1, ..., x_M)` where `x_t = 1` if term `t` is present in document `d` and `0` if not.
    - Many possible documents could have identical representation under this scheme.
- A query is represented by the *incidence vector* `\vec{q}`
- Independence: Terms are occurring in the documents *independently* - not really correct, but often gives satisfactory results in practice
- Assume that the user has a single step information need
- We need to figure out the contribution of terms to the document's relevance
    - How statistics like term frequency, document frequency, document length *influence judgments about document relevance*
    - And how we could reasonably combine them to estimate the probability of relevance
- Another assumption: The relevance of each document is independent of other documents' relevance probabilities
    - This is especially harmful when we allow returning duplicate or near duplicate documents
- Then using Bayes rule, `P(R=1 | \vec{x}, \vec{q}) = \frac{P(\vec{x} | R=1, \vec{q}) * P(R=1 | \vec{q})}{P(\vec{x} | \vec{q})}`, and accordingly for `R=0`.
- We never know the exact probabilities, so we have to use estimates.
    - If we knew the true percentage of relevant documents in the collection, we could use that as priors.

### 11.3.1 Deriving a ranking function for query terms

- We can rank by just looking at the odds of relevance rather than the full probability
    - `O(R | \vec{x}, \vec{q}) = \frac{P(R = 1 | \vec{x}, \vec{q})}{P(R = 0 | \vec{x}, \vec{q})}`
    - `= \frac{P(R = 1 | \vec{q})}{P(R = 0 | \vec{q})} * \frac{P(\vec{x} | R = 1, \vec{q})}{\vec{x} | R = 0, \vec{q})}`
- The fraction of prior probabilities are constant for a given query; no need to estimate it
- But how can we estimate the probability of an entire term incidence vector occurring?
- *Naive Bayes Conditional Independence Assumption*: The presence or absence of a word in a document is independent of the presence or absence of any other words, given the query.
    - Then `O(R | \vec{x}, \vec{q}) = O(R | \vec{q}) * \prod_{t=1} \frac{P(x_t | R=1, \vec{q})}{P(x_t | R=0, \vec{q})}`
- Additional assumption: Terms *not* occurring in the query are *equally likely* to occur in relevant and nonrelevant documents
    - If `q_t = 0` Then `p_t = u_t`.
    - Then we only need to consider terms that appear in the query
    - And divide the product term into the product over the query terms *found in the document* and the ones *not* found in the document.
- **Retrieval Status Value (RSV)**: The only thing we need to estimate eventually to rank documents
    - `RSV_d = \sum_{t: x_t=q_t=1} log \frac{p_t * (1-u_t)}{u_t * (1-p_t)}`
    - Define `c_t = log \frac{p_t}{1-p_t} + log \frac{1-u_t}{u_t}`
        - This term is log odds ratios for each term in the query.
        - The value will be 0 if a term has equal odds of appearing in relevant and non-relevant documents
        - Positive if it is more likely to appear in *relevant* documents.
        - `RSV_d` is the document score for a query.

### 11.3.2 Probability estimates in theory

- If we say that there are `S` relevant documents in total, and `x_t` is present in `s` documents, then
    - `p_t = s/S` and `u_t = (df_t - s) / (N - S).`
- Relative Frequency: One way to estimate the probability of an event from data is simply to count the number of times an event occurred divided by the total number of trials.
- Maximum Likelihood Estimate (MLE): Estimating the probability as the relative frequency.
    - The probability given to events we *happened to see* is usually too high
    - While probabilities given to unseen events become 0 and breaks our models
- *Smoothing*: Simultaneously decreasing the estimated probability of *seen* events and increasing the probability of *unseen* events
- Maximum a Posteriori (MAP): Choose the most likely point value for probabilities based on the prior and the observed evidence
    - Pseudocounts: Add a number `\alpha` to each of the observed counts.
        - Equivalent to using a uniform distribution over the vocabulary as a *Bayesian Prior*
        - `\alpha` denotes the strength of our belief in uniformity

### 11.3.3 Probability estimates in practice

- Under the assumption that relevant documents are a *very small percentage* of the collection
- Plausible to approximate statistics for non-relevant documents by using statistics for the *whole collection*
    - Then `u_t = df_t / N` and `log [(1-u_t)/u_t] \approx log N / df_t`.
    - This gives a justifcation for idf weighting used in Chapter 6.
- But we can't estimate `p_t` using the idea like this. Instead:
    - Use the frequency of term occurrence in *known* relevant documents
    - Croft and Harper (1979): Using a constant for `p_t`
        - If `p_t=0.5`: Cancels out the first term in RSV. Weak estimate, but doesn't disagree violently with our hopes for the search terms appearing in many but *not all* relevant documents
        - Combined with our `u_t` estimation, the document ranking is determined simply by which query terms occur in documents, scaled by their idf weighting.
            - Works well enough in short documents (titles or abstracts), but we want to do better
    - Greiff (1988): Empirically, `p_t` rises with `df_t`. He estimates it to be `p_t = 1/3 + 2/3 * df_t / N`.

