# The term vocabulary and postings lists

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