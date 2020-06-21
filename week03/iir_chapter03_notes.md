# Chapter 3: Dictionaries and tolerant retrieval

- Techniques that are robust to typographical errors in the query, and alternative spellings

## 3.1 Search structures for dictionaries

- Vocabulary lookup operation using dictionaries: hashing and search trees
    1. How many keys are we likely to have?
    2. Is the number likely to remain static, or change a lot?
        - And in the case of changes, are we likely to only have new keys inserted, or to also have some keys in the dictionary be deleted?
    3. What are the relative frequencies with which various keys will be accessed?
- Hashing:
    - Key is hashed into an integer over quite large space, making collsion unlikely
        - Collisions get resolved by auxiliary structures that would require some care to maintain
    - No easy way to find minor variants of a query term as they could be hashed into very different integers
    - A given hash function might not be enough in the long run
- Search trees:
    - Allow us to do various things like enumerating through all vocab terms starting with `automat*`
    - Binary tree: With two children at each internal node
        - Efficient search would require `O(log M)` comparisons, assuming balanced tree
        - The numbers of terms under the two subtrees of any node are either equal or differ by one
        - *Rebalancing*: We need to rebalance the tree when new terms gets added, to maintain the balance property
    - *B-tree*: Allow the number of subtrees under an internal node to *vary in a fixed interval* (i.e. we could make those trees to have from 2 to 4 child nodes)
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
- Trailing wildcard query: `*` symbol occurs only once at the end - use search tree to go through all documents containing terms starting with the prefix
- Leading wildcard query: Use *reverse B-tree* - each root-to-leaf path of the tree corresponds to a term in the dictionary written *backwards*
- Wildcard in the middle of a search query:
    - Use a regular B-tree first
    - then use a reverse B-tree
    - Intersect the two sets

### 3.2.1 General wildcard queries

- Express the given wildcard query `q_w` as a Boolean query `Q` on a specially constructed index
    - such that the *answer* to `Q` is a *superset* of the set of vocabulary terms matching `q_w`.
- Then we check each term in the answer to `Q` against `q_w`, discarding those vocab terms that do not match `q_w`.
- **Permuterm Indexes**
    - `$`: a special symbol to mark the end of a term. First attach this to the original term.
    - Construct a permuterm index: the various *rotations* of each term (augmented with `$`) all link to the original vocabulary term (ex. `hello -> hello$, ello$h, llo$he, lo$hel, ...`) We will refer to these rotations as *permuterm vocabularies.*
- For wildcard queries, we would want to rotate the query in a way that the `*` symbol appears at the *end* of the string. (ex. `m*n -> n$m*`)
    - Once if there are permuterm vocabularies that match the rotation, then the original terms linked to the permuterm vocabularies must match the original query.
- Multiple wildcards
    - We first rotate as if the letters between the two wildcards don't exist: (ex. `fi*mo*er -> er$fi*`)
    - Then we go through all the matched vocabularies to see if they contain the crossed out letter sequences.
- Dictionaries for permuterm indexes would be really large, in order to store all different rotations of each terms

### 3.2.2 k-gram indexes for wildcard queries

- A `k`-gram is a sequence of `k` characters. 
    (Example: `$ca, cas, ast, stl, tle, le%` are all 3-grams occuring in `castle`. Note that `$` is still a character!)
- `re*ve` turns into Boolean queries of `$re` and `ve$`. We look these up in the 3-gram index and get a list of matching terms. This list is a *conjunction* of the two 3-grams.
- We then do a post-filtering step to find the terms that actually match the original query.

## 3.3 Spelling correction

- Edit distance
- `k`-gram overlap

### 3.3.1 Implementing spelling correction

- Principle 1: Choose the *nearest* one among all possible *correct* spellings.
- Principle 2: Given two tied correct spellings, choose more *common* one.
    - Consider the number of occurrences *within the collection?*

### 3.3.2 Forms of spelling correction

- *Isolated-term correction*: Correct each query term individually, without considering other terms in the query
    - Edit distance
    - `k`-gram overlap
- *Context-sensitive correction*

### 3.3.3 Edit distance

- **Edit distance** (Levenshtein distance): Minimum number of edit operations required to transform `s_1` to `s_2`.
    - Insert a character into a string
    - Delete a character from a string
    - Replace a character of a string by another character
- We can generalize edit distance by allowing varying weights for different edit operations
- How to compute edit distance:
    - Use the dynamic programming algorithm: `O(|s_1| * |s_2|)` time
        - Fills in the entries in a matrix `m` where two dimensions equal the lengths of the two strings whose edit distances is being computed
        - `m[i, j]` is the edit distance between the string of the first `i` characters of `s_1` and the first `j` characters of `s_2`.
- But simplying calculating edit distance for all terms is really expensive
- Heuristics:
    1. Restrict the search to dictionary terms beginning with the same letter as the query string
    2. Use a version of permuterm index: the set of all rotations of the query string - for each rotation `r`, retrieve all terms with rotations beginning with `r`, that *do not have* a small edit distance from `q`
        - Could miss more pertinent terms
        - Refinement: For each rotation, discard a *suffix* of `l` characters before going through the permuterm index
            - Each retrieved term would be long enough to have a some substring in common with `q`.

### 3.3.4 k-gram indexes for spelling correction

- Further limit the set of vocabulary terms to compute edit distances
- Find terms that have many `k`-grams *in common* with the query: For "reasonable definitions" of "many `k`-grams in common", we can simply do a *single* scan through the `k`-grams index for the *original* query `q`.
    - Simply finding some terms with common `k`-grams may lead to an *implausible* (unlikely?) correction of the original query
    - Need more *nuanced* measures of the overlap in `k`-grams between a vocabulary term and the query
    - **Jaccard coefficient**: measuring overlap between two sets, `|A \cap B| / |A \cup B|`
        - The set of `k`-grams in the query `q`
        - Proceed from one vocabulary term `t` to the next
        - `t` might be written in some encoding, but we only need its length (?)

