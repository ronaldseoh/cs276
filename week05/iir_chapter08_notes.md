# Chapter 8: Evaluation in information retrieval

- How do we determine whether certain IR system is effective or not?
- IR is a highly *empirical discipline*
- User utility? User happiness?

## 8.1 Information retrieval system evaluation

- *Relevant* and *nonrelevant* documents
- Gold standard or ground truth judgment of relevance
- A test suite of information needs to be of a reasonable size: around 50 information needs has usually been found to be a sufficient minimum.
- Relevance is assessed relative to an information need, *not a query*.
- To evaluate a system, we require an *overt expression* of an information need.
- We will assume a binary decision of relevance.
- Wrong to report results on a test collection after tuning the parameters against it: Need to use one or more *development test collections* instead for tuning

## 8.2 Standard test collections

- Focus particularly on test collections for *ad hoc information retrieval system*
- The Cranfield collection
- TREC
- GOV2
- NTCIR: East Asian language and cross-language information retrieval
- CLEF
- Reuters-21578 and Reuters-RCV1
- 20 Newsgroups

## 8.3 Evaluation of unranked retrieval sets

- Most frequent and basic measures: *Precision* and *Recall*
- Accuracy: Not an appropriate measure for IR problems
    - The data is extremely skewed: Normally over 99.9% of the documents are in the nonrelevant category
    - Trying to label some documents as relevant will almost always lead to a high rate of false positives
- But users are assumed to have a certain tolerance for seeing some false positives, as long as they get some useful information.
- The advantage of using precision and recall: one is more important than the other in many circumstances.
- You can always get a recall of `1` by retrieving *all* documents for all queries.
- Precision usually decreases as the number of documents retrieved is increased.
- In general, we want to get some amount of recall while tolerating only a certain percentage of false positives.
- *F measure*: The weighted harmonic mean of precision and recall
    - $F = \frac{1}{\alpha \frac{1}{P} + (1-\alpha) \frac{1}{R}}= \frac{(\beta^2 + 1)PR}{\beta^2 P + R}$ where $\beta^2 = \frac{1-\alpha}{\alpha}$
    - Balanced F measure $F_1$: Equal weights; $\alpha=\frac{1}{2}$ or $\beta=1$.
    - $\beta < 1$ emphasize precision while $\beta > 1$ emphasize recall.
- Why harmonic mean?: We can reach the arithmetic mean of 50% when we get 100% recall by retrieving all documents for all queries
    - However, assuming 1 document in 10,000 is actually relevant, the harmonic mean would be 0.02%
    - The harmonic mean is always less than or equal to the arithmetic mean and geometric mean.
    - *When the values of two numbers differ greatly, the harmonic mean is closer to their minimum than to their arithmetic mean*.
