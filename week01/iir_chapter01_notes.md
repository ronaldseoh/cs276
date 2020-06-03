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

## 1.2 A first take at building an inverted index

- Steps for building the index in advance consists of:
    1. Collect the documents to be indexed.
    2. *Tokenize* the text, turning each document into a list of tokens.
    3. Make all the tokens *normalized* by performing linguistic preprocessing. These normalized terms will be the indexing terms.
    4. Index the documents that each term occurs in by creating an inverted index, consisting of a dictionary and postings.
- We assume for now that the first 3 are already done.
- We focus on how to build a basic inverted index by *sort-based indexing*.
    - The sequence of terms in each document, tagged by their `documentID` is sorted alphabetically.
    - Instances of the same term are then grouped by word and then by `documentID`.
    - The terms and `documentID` are then separated out. The dictionary stores the terms and has a pointer to the postings list for each term.
    - It commonly also stores other summary information such as the *document frequency* of each term.
- Dictionaries are typically stored in memory while postings lists are usually kept in disks.
- Data structures to use:
    - For an in-memory postings list, two good alternatives: singly linked lists or variable length arrays
    - When stored on disk, contiguous run of postings without explicit pointers.
