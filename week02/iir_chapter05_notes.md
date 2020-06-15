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