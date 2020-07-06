# Chapter 7: Computing scores in a complete search system

- Speedups for cosine scoring
- How to build a complete search engine?
- Vector space model and query operators

## 7.1 Efficient scoring and ranking

- For the purpose of ranking, we are interested in *relative* scores of the documents in the collection.
- Hence it suffices to compute the cosine similarity from each document unit vector $\vec{v}(d)$ to $\vec{V}(q)$, where all non-zero components of the query vector are set to 1, rather than to the unit vector $\vec{v}(q)$.
    - For any two documents $d_1$, $d_2$, $\vec{V}(q) \cdot \vec{v}(d_1) > \vec{V}(q) \cdot \vec{v}(d_2) \Leftrightarrow \vec{v}(q) \cdot \vec{v}(d_1) > \vec{v}(q) \cdot \vec{v}(d_2)$.

### 7.1.1 Inexact top $K$ document retrieval

- Instead of retrieving precisely the top $K$, let's come up with $K$ documents that are *likely* to be among the $K$ highest scoring documents.
    - Dramatically lowering the computing costs, without materially altering the user's *perceived* relevance of the top $K$ results.
    - Cosine similarity is a proxy anyway.
- The principal computing cost comes from calculating similarities between the query and *a large number of documents*.
- So we need to get many documents out of consideration without calculating their scores, using the heuristics with the two-step scheme:
    1. Find a set $A$ of documents that are contenders, where $K < \lvert A \rvert \ll N$. $A$ does not necessarily contain the $K$ top-scoring documents for the query, but is likely to have many documents with scores near those of the top $K$.
    2. Return the $K$ top-scoring documents in $A$.
- Many of these heuristics will require many parameter tunings.
- These are for free text queries and not for Boolean or phrase queries.

### 7.1.2 Index elimination

- For a multi-term query $q$, we already consider only the documents containing at least one of the query terms. We could use more heuristics.
1. Only consider documents containing terms with *high enough idf*: The postings lists of low idf terms are generally long. Basically we now consider them as stop words, they end up not contributing anything to the scoring.
    - Cutoff threshold can be adapted in a *query-dependent manner*.
2. Only consider documents containing *many* (sometimes all) of the query terms.
    - We might end up with fewer than $K$ candidates.

### 7.1.3 Champion lists

- *Champion list*: For each term $t$ in the dictionary, precompute the set of $r$ documents with the highest weights for $t$.
    - $r$ should be chosen in advance.
- Then make the set $A$ the *union* of the champion lists for each of the terms comprising $q$, and restrict cosine computation to only the documents in $A$.
    - Hence $r$ should be fairly larger than $K$.
    - One issue is that $r$ would be set during the index construction, while $K$ is application dependent.
- No need to set the same value of $r$ for all terms: we might set it higher for rarer terms.

### 7.1.4 Static quality scores and ordering

- **Static quality score**: A measure of quality $g(d)$ for each document $d$ that is query-*independent* and thus *static*.
    - A number between 0 and 1
- Then the *net score* for a document $d$ is some combination of $g(d)$ together with the query-dependent score.
- Using these static quality scores, we could create postings lists by *decreasing* value of $g(d)$ and perform the postings intersection.
    - Note that what we needed for postings intersection was a *single common ordering* between all postings.
- *Global champion list*: Extension of a regular champion list. Maintain the list with the highest values for $g(d) + \text{tf-idf}_{t,d}$.
- Maintaining *two* postings lists consisting of *disjoint* sets of documents
    - *High*: the list containing the $m$ documents with the highest tf values for $t$.
    - *Low*: the list containing *all other* documents containing $t$.
    - Then we can first try scanning only through high lists of all query terms. We go through low lists only if we don't get $K$ documents.

