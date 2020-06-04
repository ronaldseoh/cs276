# Chapter 2: The term vocabulary and postings lists

## 2.1 Document delineation and character sequence decoding

### 2.1.1 Obtaining the character sequence in a document

- We would have to determine the correct encoding.
- We will simply assume that our documents are a list of characters.
- The idea that text is a linear sequence of characters is also called into question by some writing systems, such as Arabic.

### 2.1.2 Choosing a document unit

- Document unit
- The issue of indexing *granularity* for very long documents
    - Make each chapter or paragraph as a mini-document?
    - Go even further down individual sentences?
    - Precision/Recall Tradeoff
    - Can be alleviated by use of explicit or implicit proximity search
- *Simultaneously* index documents at multiple levels of granularity
- An IR system should be designed to offer choice of granularity.
- For now, we will assume that a suitable size document unit has been chosen.

## 2.2 Determining the vocabulary of terms

### 2.2.1 Tokenization

- Token: An instance of a sequence of characters in some particular document that are grouped together as *a useful semantic unit for processing*.
- Type: The class of all tokens containing the same character sequence.
- Term: A (perhaps normalized) type that is included in the IR system's dictionary.
    - Index terms could be entirely distinct from the tokens.
    - Rather than being exactly the tokens that appear in the document, they are usually derived from them by various *normalization* processes.
- What are the correct tokens to use?
    - Just split on all non-alphanumeric characters?
    - For either Boolean or free text queries, you always want to do the exact same tokenization to both document and query words.
    - These issues of tokenization are language-specific: Need to do *language identification*.
    - Unusual specific tokens that we wish to recognize as terms (ex. `C++`, `C#`, `M*A*S*H`)
    - Handling Hyphens: As a classification problem, or some heuristic rules
    - Whitespaces
    - Word segmentation: Heuristics, machine learning sequence models (trained over hand-segmented words)
    - Alternatives: Do all indexing via just short subsequences of characters (character k-grams) - individual Chinese character has some semantic content

### 2.2.2 Dropping common terms: stop words

- Stop words: Some extremely common words that would appear to be of *little value* in helping select documents matching a user need
- Stop list: Take the most frequent terms hand-filtered for their semantic content relative to the domain of the documents being indexed.
- Significantly reduces the number of postings that a system has to store, and a lot of the time not indexing stop words does little harm.
    - However, phrase searches could be affected (e.g. `President of the United States`, `flights to London`)
- The general trend in IR over time has been from standard use of quite large stop lists (200~300 terms) to very small stop lists (7~12 terms) to *no stop list whatsoever*.
    - Exploiting the *statistics* of language so as to be able to cope with common words in better ways.
    - Good compression
    - Standard term weighting leading to very common words having *little impact* on document rankings
    - Impact-sorted indexes to terminate scanning postings list *early*

### 2.2.3 Normalization (equivalence classing of terms)

- 

### 2.2.4 Stemming and lemmatization

- 