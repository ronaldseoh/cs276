# Chapter 5: Index compression

- Compression techniques for dictionary and inverted index
- Obvious benefits: Need less space
- More subtle benefits:
    - *Increased* use of caching: Commonly used terms could have its postings list in memory
    - Faster trasnfer of data from disk to memory
- Decompression speeds must be high
- In this chapter, we define a *posting* as a `docID` in a postings list
    - Do not consider frequency and position information

## 5.1 Statistical properties of terms in information retrieval

- Preprocessing affects the size of the dictionary and the number of *nonpositional* postings greatly.
- The Rule of 30: The 30 most common words account for 30% of the tokens in written text
    - Although a stop list of 150 words reduces the number of postings by a quarter or more, this size reduction does *not* carry over to the size of the compressed index.
    - The postings lists of frequent words require only a few bits per posting after compression.
- **Loseless** compression: All information is preserved.
    - Better compression ratios with *lossy* compression: Case folding, stemming, and stop word elimination
        - Makes sense when the 'lost' information would never be used by search system

- `M`: The number of *distinct* terms
    - The Oxford English Dictionary (OED): 600,000 words
    - But vocabularies of large collections are usually larger than OED: Names of people, locations, products, etc.
        - These do need to be included

### 5.1.1 Heaps' law: Estimating the number of terms

- **Heaps' law**: A better way to guess `M` - as a function of *collection size*
    - `M = kT^b`
        - `T` is the number of tokens in the collection
        - `30 <= k <= 100`, `b \approx 0.5`
- The motivation for Heaps' law:
    - The simplest possible relationship between *collection size* and *vocabulary size* is *linear* in log-log space
    - The assumption of *linearity* is usually born out in practice
- The parameter `k` depends a lot on the nature of the collection and how it is processed
- Implications of Heaps' law:
    - The dictionary size *continues* to increase with more documents, rather than reaching certain maximum vocab size
    - The size of the dictionary is indeed quite large for large collections
- The law have been empirically shown to be true; hence dictionary compression is important!

### 5.1.2 Zipf's law: Modeling the distribution of terms

- Want to know how terms are distributed *across documents*
- **Zipf's law**: If `t_1` is the most common term in the collection, `t_2` is the next most common and so on,
    - Then the collection frequency `cf_i` of the `i`-th most common term is *proportional to* `1/i`:
    - `cf_i \propto \frac{1}{i}`
    - If the most frequent term occurs `cf_1` times, then the second most frequent term has *half* as many occurrences, and so on.
- The intuition: Frequency decreases very *rapidly* with rank
- *Power law* with exponent `k=-1`: We can write Zipf's law equivalently as `cf_i = ci^k` where `k=-1` and `c` is some constant.

## 5.2 Dictionary compression

- Dictionary data structures that achieve increasingly higher compression ratios
- The main goal: Make the dictionary fit into the main memory, or at least a large portion of it
    - So that it could support *high* query throughput.

### 5.2.1 Dictionary as a string

- The simplest solution: Sort the vocab lexicographically and store it in an array of fixed-width entries
    - 20 bytes for the term itself
    - 4 bytes for its document frequency
    - 4 bytes for the pointer to its postings list: Resolves 4GB address space
    - `REUTERS-RCV1`: `M * (20+4+4) = 400,000 * 28 = 11.2MB`
- The average length of a term in English is about 8 characters, so on average we would be wasting 12 characters in the fixed-width scheme.
    - At the same time, no way to store terms longer than 20 characters
    - Solution: store dictionary terms as *one long string*
        - Pointers mark the end of the preceding term and the beginning of the next. 
        - Saves space, although we now need to store pointers
            - `400,000 * 8 = 3.2 * 10^6`
            - `log_2(3.2 * 10^6) \approx 22 bits or 3 bytes long`
- In total, we now need `400,000 * (4+4+3+8) = 7.6MB`
    - 4 bytes each for frequency and postings pointer
    - 3 bytes for the term pointer
    - 8 bytes bytes on average for the term itself

## 5.2.2 Blocked storage

- Group terms in the string into blocks of size `k` and keep pointers only for the *first* term of each block.
    - Plus, we store the *length* of the term in the string as an additional byte at the beginning of the term.
- We eliminate `k-1` pointers, but instead introduce an additional `k` bytes for storing the length of each term.
- In total, we save 5 bytes per 4-term block.
    - `REUTERS-RCV1`: `400,000 * 1/4 * 5 = 0.5MB saved -> 7.1MB`
- We can save even more space by increasing the block size `k`, at the expense of worsening the *speed of term lookup*.
- **Front coding**: Consecutive entries in an alphabetically sorted list share common prefixes
- Minimal perfect hashing: A hash function that maps `M` terms *without any collisions*
    - Cannot be used in dynamic environment as new terms will create collision, requiring a completely new hash function all the time
- Even with all the compression, it may be the case that storing the entire dictionary on memory is not feasible
    - If we have to partition the dictionary onto pages that are stored on disk, then we can index the *first* term of each page using a B-tree.
    - One additional seek for retrieving the dictionary page is a significant but tolerable cost.
