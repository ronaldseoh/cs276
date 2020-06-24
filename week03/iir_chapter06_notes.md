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

### 6.1.2 Learning weights

- How do we set the weights??
- Used to be set by 'experts', but nowadays we learn them from curated training examples
- *Machine-learned Relevance*

## 6.2 Term frequency and weighting

- A document or zone that mentions a query term *more often* should be given higher scores.
- Free text query: Terms are given without any connecting search operators - we simply view them as a set of words
    - Then we could simply compute the total score by summing up over each term a match score between each query term and the document
- We need to assign *weights* to each term in the document
    - The simplest approach: Use *term frequency* - Weights to be equal to *the number of occurrences* of term `t` in the document `d`.
- **Bag of Words Model**: Having number of occurrences as weights is a *quantitative digest* of the document; ignores the exact ordering of the terms
    - Intuitive that two documents with similar bag of words represnetations are similar *in content*.

### 6.2.1 Inverse document frequency

- Using plain term frequency could be problematic when certain terms have very little or no discriminating power in determining relevance
    - Simple Solution: *Scale down* the term weights of terms with high *collection* frequency (total number of occurrences within the entire collection)
- *Document frequency*: The number of *documents* in the collection that contain the term
    - Document frequency and collection frequency could behave quite differently
- **Inverse document frequency (idf)**: `idf_t = log(N / df_t)`

