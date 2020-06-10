# Chapter 4: Index construction

- How to construct an inverted index
- The design of indexing algorithms is governed by hardware constraints
- Blocked sort-based indexing
- Single-pass in-memory indexing
- Indexing distributed over computer clusters
- Dynamic indexing
- Complicating issues: security and indexes for ranked retrieval
- The reader should be aware that building the subsystem that feeds raw text to the indexing process can in itself be a challenging problem

## 4.1 Hardware basics

- Access to data in memory is much faster than access to data on disk.
- Caching: Keep frequently used data in main memory
- Seek time: The time needed for the disk head to move to the part of the disk where the data are located (averages 5ms for typical disks)
- To maximize data transfer rates, chunks of data that will be read together should therefore be stored *contiguously* on disk.
- Since OSes typically read and write entire *blocks*, reading a single byte from disk can take as much time reading the entire block.
- Buffer: the part of main memory where a block being read or written is stored
- Since data transfers from disk to memory are handled by the system bus, the processor would be available to process data during disk I/O.
    - We can hence speed up data transfers by storing *compressed* data on disk. Assuming an efficient decompression algorithm, the total time of reading and then decompressing data is usually *less than* reading uncompressed data.
- Servers used for IR systems typically have several GBs of main memory, and available disk space is several orders of magnitude larger.

## 4.2 Blocked sort-based indexing

- Recap of creating nonpositional indexes from Chapter 1:
    - We first make a pass through the collection assembling all term-`docID` pairs
    - Then sort the pairs with the term as the dominant key, and `docID` as the secondary key.
    - Finally, we organize the `docID`s for each term into a postings list and compute statistics like term and document frequency.
    - All these can be done in memory for small collections.
- Want methods for *large collections* that require the use of secondary storage
- For more efficiency, we now represent terms as `termID` - a unique serial number.
- The index construction algorithms all do a single pass through the data.
- `REUTERS-RCV1` collection: 800,000 documents, 100,000,000 tokens, average of 200 tokens per document
    - Typical collections today are often 1 or 2 orders of magnitude larger than `Reuters-RCV1`.
- With main memory insufficient, we need to use an *external sorting* algorithm that uses disks.
    - The central requirement is that it *minimizes* the number of *random disk seeks* during sorting.
- **Blocked Sort-based indexing algorithm (BSBI)**
    1. Segments the collection into parts of equal size
    2. Sorts the `termID`-`docID` pairs of *each part* in memory
    3. Stores intermediate sorted results on disk
    4. Merges all intermediate results into the final index
- The algorithm parses documents into `termID`-`docID` pairs and accumulates the pairs in memory until *a block of a fixed size is full.* We choose the block size to fit comfortably into memory to permit a fast in-memory sort. The block is then inverted and written to disk.
- Inversion:
    1. Sort the `termID`-`docID` pairs.
    2. Collect all `termID`-`docID` pairs with the same `termID` into a postings list
    3. Write the list to the disk.
- How expensive is BSBI?
    - `Theta(T log T)` as the step with the highest time complexity is sorting and `T` is an upper bound for the number of items we must sort (the number of `termID`-`sortID` pairs)
    - However, the actual indexing time is usually dominated by the time it takes to *parse the documents* and *to do the final merge*.

