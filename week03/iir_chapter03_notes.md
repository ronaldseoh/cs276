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
