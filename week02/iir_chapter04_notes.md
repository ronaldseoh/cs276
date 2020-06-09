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

