# Boolean Retrieval

## What's IR?

- Information Retrieval: Finding materials of an *unstructured* nature
- Unstructured data: Data which does not have clear, semantically overt, easy-for-a-computer structure
- Almost no data are truly 'unstructured': IR can also be used to facilitate "semi-structured" search
- Filtering / Clustering documents
- Web search, personal information retrieval, and enterprise, institutional, and domain-specific search

## 1.1 An example information retrieval problem

- Determine which plays of Shakespeare contains the words `Brutus`, `Ceasar`, but no `Calpurnia`.
- We can do *linear scan* or grepping through all the texts but we would like to
    - Process *large document collections* quickly
    - Allow more *flexible* matching operations
    - Allow *ranked* retrieval
- **Term-Document Incidence Matrix**: Instead of linear search, we could *index* the documents in advance
    - *Terms*: usually words, but not always
    - To answer the query `Brutus AND Caesar AND (NOT Calpurnia)`, we take binary vectors for each of `Brutus`, `Caesar`, and `Calpurnia`, complement the last, and do bitwise `AND` operations.
- **Boolean Retrieval Model**: A model for information retrieval in which we can pose any query which is in the form of *a Boolean expression* of terms - terms combined with boolean operators `AND`, `OR` and `NOT`.
- 'Documents': Whatever units we have decided to build a retrieval system over
- 'Collection' ('Corpus')
- **Ad-hoc Retrieval Task**: Provide documents from within the collection that are relevant to an arbitrary user *information need*, communicated to the system by means of a *one-off, user-initiated query*.
- Key statistics for assessing the *effectiveness*:
    - Precision: What fraction of the returned results are relevant to the information need?
    - Recall: What fraction of the relevant documents in the collection were *returned* by the system?
- **Inverted Index**:
    - Term-Document Matrix is extremely *sparse*: few non-zero entries. It would be better to record the things that do occur.
    - The basic idea: We keep a *dictionary* (vocabulary) of terms. For each term, we maintain a list of records ('posting') of the term appearing in certain documents.