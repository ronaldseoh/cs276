# Chapter 2: The term vocabulary and postings lists

## 2.1 Document delineation and character sequence decoding

### 2.1.1 Obtaining the character sequence in a document

- We would have to determine the correct encoding.
- We will simply assume that our documents are a list of characters.
- The idea that text is a linear sequence of characters is also called into question by some writing systems, such as Arabic.

### 2.1.2 Choosing a document unit

- **Document unit**
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

- **Token**: An instance of a sequence of characters in some particular document that are grouped together as *a useful semantic unit for processing*.
- Type: The class of all tokens containing the same character sequence.
- **Term**: A (perhaps normalized) type that is included in the IR system's dictionary.
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

- **Stop words**: Some extremely common words that would appear to be of *little value* in helping select documents matching a user need
- **Stop list**: Take the most frequent terms hand-filtered for their semantic content relative to the domain of the documents being indexed.
- Significantly reduces the number of postings that a system has to store, and a lot of the time not indexing stop words does little harm.
    - However, phrase searches could be affected (e.g. `President of the United States`, `flights to London`)
- The general trend in IR over time has been from standard use of quite large stop lists (200-300 terms) to very small stop lists (7-12 terms) to *no stop list whatsoever*.
    - Exploiting the *statistics* of language so as to be able to cope with common words in better ways.
    - Good compression
    - Standard term weighting leading to very common words having *little impact* on document rankings
    - Impact-sorted indexes to terminate scanning postings list *early*

### 2.2.3 Normalization (equivalence classing of terms)

- We often would want tokens in queries to match the ones in the document even if they are not quite the same.
- **Token normalization**: The process of *canonicalizing tokens* so that matches occur despite superficial differences in the character sequences of the tokens.
- Most standard way: Implicitly creating **equivalence classes**, normally named after one member of the set
    - It is not obvious when you might want to add characters.
- Alternative: Maintain relations between unnormalized tokens.
    - Can be extended to hand-constructed list of synonyms (e.g. `car` <-> `automobile`)
    - Index *unnormalized* tokens and maintain a *query expansion list* (requires more processing at query time)
        - Then a query term is effectively a *disjunction* of several postings lists.
    - Or perform the expansion *during index construction* (require more space for storing postings)
- The best amount of equivalence classing or query expansion to do is a fairly open question.
- Common normalization forms:
    - Accent and diacritics
        - Occasionally words are distinguished only by their accents.
    - Capitalization/case-folding: all letters to lower case, or just make some tokens lower case?
    - Other issues in English
    - Other languages
        - Japanese: An intermingling of multiple alphabets - requires complex equivalence classing across the writing systems
        - Document collections being indexed can include documents from many different languages.

### 2.2.4 Stemming and lemmatization

- Different forms of a word for grammatical reasons: (e.g. `organize`, `organizes`, `organizing`), and families of derivationally related words (e.g. `democracy`, `democratic`, `democratization`)
- Reduce inflectional forms and derivationally related forms to *a common base form*.
- **Stemming**: A crude heuristic process that chops off the ends of words in the hope of achieving the goal correctly most of the time
- **Lemmatization**: Using a vocabulary and morphological analysis of words to remove inflectional endings only and return the base or dictionary form of a word (*lemma*)
- Stemming most commonly collapses derivationally related words, whereas lemmatization commonly only collapses the different inflectional forms of a lemma.
- *Porter Stemmer*: 5 phases of word reductions, applied sequentially.
- Doing full morphological analysis produces at most very modest benefits for retrieval. Either form of normalization tends *not* to improve English information retrieval performance in aggregate - at least not by very much.
- Stemming increases recall while harming precision.
    - Moving to a lemmatizer wouldn't completely fix the problem because particular inflection forms are used in particular locations.
    - Getting better value from term normalization depends more on *pragmatic issues of word use* than on formal issues of linguistic morphology.
    - The situation is different for languages with much more morphology: quite large gains from the use of stemmers

## 2.3 Faster postings list intersection via skip pointers

- Basic intersection operation of lists with the lengths of `m` and `n` takes `O(m+n)` time. Can we do better?
- **Skip list**: Augment postings lists with *skip pointers* at indexing time - skip pointers are effectively shortcuts that allow us to avoid processing parts of the postings list *that will not figure in the search results*.
- A number of variant versions of postings list intersection with skip pointers is possible, depending on *when exactly you check the skip pointer*.
- Skip pointers will only be available for the original postings lists.
- The presence of skip pointers only helps for `AND` queries, not for `OR` queries.
- More skips means shorter skip spans, and that we are more likely to skip.
    - But it also means lots of comparisons to skip pointers, and lots of space storing skip pointers.
- A simple heuristic: For a postings list of length `P`, use `sqrt(P)` evenly-spaced skip pointers.
    - Ignores any details of the distribution of query terms.
- Choosing the *optimal encoding* for an inverted index
    - Nowadays, CPUs are fast and disk is slow, so reducing disk postings list size dominates. However, if you're running a search engine with everything in memory then the equation changes again.

# 2.4 Positional postings and phrase queries

- Phrase queries (uses a double quotes syntax) and *implicit* phrase queries (like peoples' names; entered without use of double quotes).
- It is no longer sufficient for postings lists to be simply lists of documents that contain individual terms.

## 2.4.1 Biword indexes

- Consider *every* pair of consecutive terms in a document as a phrase, or a *biword*
- Phrases with more than two words can be processed by breaking them down.
    - Could work well in practice, but there can and will be occasional false positives: We cannot verify whether the documents actually contain the *original* phrase.
- Noun and noun phrases: can be divided by various *function words* (e.g. `the abolition of slavery`, `renegotiation of the constitution`)
    1. Tokenize the text and perform part-of-speech tagging.
    2. Group terms into nouns, including proper nouns (`N`), and function words, including articles and prepositions (`X`), among other classes.
    3. Deem any strings of terms of the form `NX*N` to be an extended biword. *Each such extended biword is made a term in the vocabulary.*

