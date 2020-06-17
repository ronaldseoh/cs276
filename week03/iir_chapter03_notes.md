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
