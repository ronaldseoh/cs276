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

## 1.3 Processing Boolean queries

- **Intersection** (Merging) operation: Given the simple conjunctive query, we need to merge two lists with a logical `AND` operation.
- Simple merge algorithm `INTERSECT(p_1, p_2)`: Walk through the two postings list simultaneously, in time linear in the total number of postings entries.
    - At each step, we compare the `docID` in the results list, and advance both pointers. Otherwise, we advance the pointer pointing to the *smaller* `docID`.
    - If the lengths of the postings lists are `x` and `y`, the intersection takes `O(x + y)` operations. (Formally, the complexity of queryin is `Theta(N)`, where `N` is the number of documents in the collection. Our indexing methods gain us just a constant, but that constant would be huge in practice)
    - To use this algorithm, it is crucial that postings be sorted by a single global ordering. Using a numeric sort by `docID` is one simple way to achieve this.
- *Query optimization*: The process of selecting how to organize the work of answering a query, so that the least total amount of work needs to be done.
    - What is the best order in which postings lists are accessed?
    - The standard heuristic is to process terms in order of *increasing document frequency*. If we start by intersecting the two smallest postings lists, then all intermediate results *must be no bigger* than the smallest postings list.
    - When we have `OR`s combined by `AND`: Estimate the size of each `OR` by the sum of the frequencies of its disjuncts.
    - For arbitrary Boolean queries, we would have to evaluate and temporarily store the answers for immediate expressions in a complex expression.
    - However, in many circumstances, a query is *purely conjunctive*.
    - `INTERSECT(<t_1, ..., t_n>)`: In this case, it would be more efficient to intersect each retrieved postings list with the current intermediate results in memory, where we initially set up the intermediate result with the posting list of the least frequent term.
