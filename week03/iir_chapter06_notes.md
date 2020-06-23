# Chapter 6: Scoring, term weighting, and the vector space model

- Rank-order the documents
- Three ideas:
    1. Parametric and zone indexes
        - To index and retrieve documents
        - Simple means of scoring
    2. Weighting importance of a term in a document, using statistics of occurrence
    3. Viewing each document as a vector of weights
        - Vector space scoring: to compute a score between *a query* and *each document*

## 6.1 Parametric and zone indexes

- Digital documents often have *metadata*
- One parametric index for each field
    - Support querying *ranges* on ordered values: Structures like B-tree may be used for the field's dictionary
- Zones: Similar to fields, but the contents can be arbitrary free text
    - Document titles, abstracts, etc.
    - The dictionary for a zone index must structure whatever vocabulary stems from the text of that zone.
- We can directly encode the *zone in which* a term occurs in the *postings*, and reduce the dictionary size
    - Also allows efficient computation of *weighted zone scoring*

### 6.1.1 Weighted zone scoring

- Given a boolean query `q` and a document `d`, weighted zone scoring assigns to the pair `(q, d)` a score in the interval `[0, 1]`
    - By computing a *linear* combination of **zone scores**: each zone of the document contributes a Boolean value.
    - The Boolean score from a zone would be `1` if *all* the query terms occur in that zone.
    - `\sum_{i=1}^{l} g_i * s_i`, where `g_i` are weights given for each zone, and `s_i` is the score from each zone
- Weighted zone scoring is also referred to as a **ranked Boolean retrieval**.

