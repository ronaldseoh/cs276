# Chapter 3: Dictionaries and tolerant retrieval

- Techniques that are robust to typographical errors in the query, and alternative spellings

## 3.1 Search structures for dictionaries

- How should we implement vocabulary lookup operation using dictionaries? hashing or search trees
- Key considerations:
    1. How many keys are we likely to have?
    2. Is the number likely to remain static, or change a lot?
        - And in the case of changes, are we likely to only have new keys inserted, or to also have some keys in the dictionary be deleted?
    3. What are the relative frequencies with which various keys will be accessed?

### Hashing

- Key is hashed into an integer over quite large space, making collsion unlikely
    - Collisions get resolved by auxiliary structures that would require some care to maintain
- No easy way to find *minor variants* of a query term, as they could be hashed into very different integers
- A given hash function might not be enough in the long run

### Search trees

- Allow us to do various things like enumerating through all vocab terms starting with `automat*`
- **Binary tree**: With two children at each internal node
    - Efficient search would require $O(\log M)$ comparisons, assuming *balanced tree*
    - The numbers of terms under the two subtrees of any node are either equal or differ by one
    - Rebalancing: We need to rebalance the tree when new terms gets added, to maintain the balance property
- **B-tree**: Allow the number of subtrees under an internal node to *vary in a fixed interval* (i.e. we could allow those trees to have from 2 to 4 child nodes)
    - Each branch under an internal node can be considered as a test for a range of character sequences: Collapsing multiple levels of the binary tree into one
        - Beneficial especially when some of the dictionary is in disk - we can prefetch imminent binary tests at the current level
        - The interval would be decided based on the sizes of disk blocks
- Unlike hashing, search trees require the characters used in the collection to have a *prescribed ordering*

## 3.2 Wildcard queries

- Used in the following situations:
    1. The user is uncertain of the spelling
    2. Aware of variants of the term and looking for documents containing any of them
    3. Looking for documents likely to be found after stemming, but unsure whether search engine performs them
    4. Uncertain of the correct rendition of a foreign word or phrase
- Trailing wildcard query: `*` symbol occurs only *once* at the end - use search tree to go through all documents containing terms starting with the prefix
- Leading wildcard query: `*` symbol once in the beginning; Use **_reverse_** B-tree - each root-to-leaf path of the tree corresponds to a term in the dictionary written *backwards*
- Wildcard in the middle of a search query?
    1. Use a regular B-tree first
    2. Use a reverse B-tree next
    3. Then **intersect the two sets**
    - But this is too expensive!
    - The solution: Transform wild-card queries so that the `*`'s occur at the end

### 3.2.1 General wildcard queries

- A common strategy: Express the given wildcard query $q_w$ as a *Boolean* query $Q$ on a specially constructed index
    - Such that the *answer* to $Q$ is a *superset* of the set of vocabulary terms matching $q_w$.
    - Then we check each term in the answer to $Q$ against $q_w$, discarding those vocab terms that do not match $q_w$.
- **Permuterm Indexes**
    - `$`: a special symbol to mark the end of a term. First attach this to the original term.
    - Construct a permuterm index: the various *rotations* of each term (augmented with `$`) all link to the original vocabulary term (ex. `hello$, ello$h, llo$he, lo$hel, ...` $\rightarrow$ `hello`)
        - We will refer to these rotations as *permuterm vocabularies.*
- For wildcard queries, we would want to rotate the query in a way that the `*` symbol appears at the *end* of the string. (ex. `m*n -> n$m*`)
    - Once if there are permuterm vocabularies that match the rotation, then the original terms linked to the permuterm vocabularies must match the original query.
- Multiple wildcards
    - We first rotate as if the letters between the two wildcards don't exist: (ex. `fi*mo*er -> er$fi*`)
    - Then we go through all the matched vocabularies to see if they contain the crossed out letter sequences.
- Dictionaries for permuterm indexes would be really large, in order to store all different rotations of each terms

### 3.2.2 $k$-gram indexes for wildcard queries

- A **$k$-gram** is a sequence of $k$ characters. 
    - Example: `$ca, cas, ast, stl, tle, le$` are all 3-grams occuring in `castle`. Note that `$` is still a character!
- We maintain a second inverted index from $k$-grams to dictionary terms that match each $k$-gram.
- `re*ve` turns into Boolean queries of `$re` and `ve$`. We look these up in the 3-gram index and get a list of matching terms. This list is a *conjunction* of the two 3-grams.
- We then do a *post-filtering* step to find the terms that actually match the original query.
- Wildcard queries are essentially a disjunction of single-term queries
    - Then combination of wildcard queries using Boolean operators are *conjunctions* of disjunctions
    - Increases the processing load on search engines due to added lookups and filtering

## 3.3 Spelling correction

- Edit distance
- $k$-gram overlap
- Non-word errors
    - Any word not in a dictionary is an error
    - Generate candidates: real words that are similar to error
    - Choose the best one: shorted weighted edit distance or highest noisy channel proability
- Real-word errors
    - Typographical errors
    - Cognitive errors
    - Find candidate words with similar pronunications or spellings
    - Particularly more context sensitive: have to consider whether the surronding words *makes sense*

### 3.3.1 Implementing spelling correction

- Principle 1: Choose the *nearest* one among all possible *correct* spellings.
- Principle 2: Given two tied correct spellings, choose more *common* one.
    - Consider the number of occurrences *within the collection?*

### 3.3.2 Forms of spelling correction

- *Isolated-term correction*: Correct each query term individually, without considering other terms in the query
    - Edit distance
    - $k$-gram overlap
- *Context-sensitive correction*

### 3.3.3 Edit distance

- **Edit distance** (Levenshtein distance): Minimum number of edit operations required to transform $s_1$ to $s_2$.
    - Insert a character into a string
    - Delete a character from a string
    - Replace a character of a string by another character
- We can generalize edit distance by allowing varying weights for different edit operations
- How to compute edit distance:
    - Use the dynamic programming algorithm: $O(\lvert s_1 \rvert \cdot \lvert s_2 \rvert)$ time
        - Fills in the entries in a matrix $m$ where two dimensions equal the lengths of the two strings whose edit distances is being computed
        - $m\lbrack i, j \rbrack$ is the edit distance between the string of the first $i$ characters of $s_1$ and the first $j$ characters of $s_2$.
- But simplying calculating edit distance for all terms is really expensive
- Heuristics:
    1. Restrict the search to dictionary terms beginning with the same letter as the query string
    2. Use a version of permuterm index: the set of all rotations of the query string - for each rotation $r$, retrieve all terms with rotations beginning with $r$, that *do not have* a small edit distance from $q$
        - Could miss more pertinent terms
        - Refinement: For each rotation, discard a *suffix* of $l$ characters before going through the permuterm index
            - Each retrieved term would be long enough to have a some substring in common with $q$.

### 3.3.4 $k$-gram indexes for spelling correction

- Further limit the set of vocabulary terms to compute edit distances
- Find terms that have many $k$-grams that share "most" $k$-grams with the query.
    - For "reasonable definitions" of "many $k$-grams in common", we can simply do a *single* scan through the $k$-grams index for the *original* query $q$.
    - Simply finding some terms with common $k$-grams may lead to an *implausible* (unlikely?) correction of the original query
    - Need more *nuanced* measures of the overlap in $k$-grams between a vocabulary term and the query
    - **Jaccard coefficient**: measuring overlap between two sets, $\lvert A \cap B \rvert / \lvert A \cup B \rvert$
        - The set of $k$-grams in the query $q$
        - Proceed from one vocabulary term $t$ to the next
        - $t$ might be written in some encoding, but we only need its length (?)

### 3.3.5 Context sensitive spelling correction

- Isolated-term correction would fail to correct when individual terms in the query are correctly spelled
- Simplest solution: Enumerate corrections of the all individual query terms and try substituting each
    - Expensive
    - Heuristic: Look for most *frequent* combinations in the collection or the query logs

## 3.4 Phonetic correction

- Phonetic correction: Correcting a query that *sounds like* the target term
- Main idea: Generate a **phonetic hash** that similar-sounding terms would all hash to
- **Soundex Algorithms**
    1. Turn every term to be indexed into *a 4-character reduced form*, and build an inverted index called *soundex index* from these reduced forms to the original terms
    2. Do the same with query terms
    3. When the query calls for a soundex match, search this soundex index
- Instead of 4-character reduced form, there is an alternative in the form of alphabet plus three digits
- Observations:
    - Vowels are viewed as interchangeable, in transcribing names
    - Consonants with similar sounds are put in equivalence classes: related names would often have same soundex codes
- These methods are writing system dependent