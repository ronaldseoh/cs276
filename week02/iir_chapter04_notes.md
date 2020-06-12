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
        - Note that quick sort takes `O(N ln N)` expected steps.
    - However, the actual indexing time is usually dominated by the time it takes to *parse the documents* and *to do the final merge*.

## 4.3 Single-pass in-memory indexing

- BSBI scales well, but needs a data structure for mapping terms to `termID`s. For very large collection, this data structure does not fit into memory.
- **Single-pass in-memory indexing (SPIMI)**:
    - Uses terms instead of `termID`s
    - Writes each block's *dictionary* (best implemented as hash) to disk, then starts a new dictionary for the next block
- Tokens are processed one by one during each successive call of `SPIMI-INVERT`.
- Differences between BSBI and SPIMI:
    - SPIMI adds a posting directly to its postings list, instead of first collecting all pairs and sorting them: Each postings list is *dynamic*
        - Faster because no sorting is required
        - Saves memory because we keep track of the term a postings list belongs to, so the `termID`s of postings need not be stored
        - As a result, blocks that SPIMI process could be much larger
- Because we do not know how large the postings list of a term will be when first encounter it, we allocate space for a short postings list initially
    - Double the space each time it is full
    - Some memory could be wasted
    - However, the overall memory requirements for the dynamically constructed index of a block in SPIMI are still lower than in BSBI.
- When memory has been exhausted, we write the index of the block (dictionary and postings lists) to disk.
    - We have to sort the terms first because we want to write postings lists in lexicographic order to facilitate the final merging step.
- Both the postings and the dictionary terms can be stored compactly on disk if we employ compression.
- Time Complexity: `Theta(T)` because no sorting of tokens is required and all operations are at most linear in the size of the collection.

## 4.4 Distributed indexing

- Collections are often too large to process on a single machine.
- Web search engines in particular use *distributed indexing* algorithms for index construction.
- We describe indexing for a *term-partitioned* index. (Most large search engines prefer a document-partitioned index)
- **MapReduce**: A general architecture for distributed computing on large computer clusters
    - Hundreds of machines would be on such clusters, but individual machines could fail any time
    - Hence one requirement for robust distributed indexing is that we divide up the work up into chunks that we can easily assign, and *reassign* upon any failure.
    - A *master node* directs the process of assigning and reassigning tasks to individual worker nodes.
- The map and reduce phases of MapReduce split up the computing job into *chunks* that standard machines can process in a short time.
    - The input data (i.e. a collection of web pages) are split into `n` splits
    - Splits are not preassigned to machines, but are instead assigned by the master node on an ongoing basis
        - If a machine dies or becomes a laggard due to hardware problems, the split it is working on is simply reassigned to another machine.
- In general, MapReduce breaks a large computing problem into smaller parts by recasting it in terms of manipulation of *key-value pairs*.
    - For indexing, a key-value pair has the form `(termID, docID)`.
    - How do we manage the mapping from term to `termID`s in distributed indexing?
        - Maintain a (perhaps precomputed) mapping for *frequent* terms that is copied to all nodes
        - and use terms directly (instead of `termID`s) for infrequent terms
        - We will assume that all nodes share a consistent term-`termID` mapping.
- *Map Phase*: Mapping splits of the input data to key-value pairs.
    - Since this is the same parsing task we've seen in BSBI and SPIMI, we call the machines that execute the map phase **parsers**.
    - Each parser writes its output to local intermediate files, the *segment files*.
- *Reduce Phase*: We want all values for a given key to be stored *close together*, so they can be read and processed quickly.
    - Partition the keys into `j` term partitions
    - Have the parsers write key-value pairs for each term partition into *a separate segment file*.
        Example: The term partitions according to first letter: `a-f`, `g-p`, `q-z`, and `j=3`. (In general, key ranges need not correspond to contiguous terms or `termID`s.)
    - Each term partition thus corresponds to `r` segment files, where `r` is the number of parsers.
    - Collecting all values (`docID`s) for a given key (`termID`s) into one list is the task of the **inverters** in the reduce phase.
        - The master assigns each term partition to a different inverter (we assume that segment files are of a size that a single machine can handle)
    - The list of values is sorted for each key and written to the final sorted postings list.
- Parsers and inverters are not separate set of machines - the master identifies idle machines and assigns tasks to them.
- Each segment file only requires one sequential read because all data relevant to a particular inverter were written to a *single segment file* by the parsers.

