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

