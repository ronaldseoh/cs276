# Chapter 14: Vector space classification

- Different from Chapter 13, we adopt the vector space model (from Chapter 6) to represent a document
- Using the vector space model for classification relies on **Contiguity Hypothesis**:
    - Documents in the same class form a contiguous region, and *regions of different classes do not overlap*.
- Depends heavily on the particular choices we make for the document representation: type of weighting, stop lists, etc.
    - If the document representation is unfavorable, the contiguity hypothesis will not hold and successful vector space classification is not possible.
- *Unweighted* and *Unnormalized* counts should not be used in vector space classification.
- Two vector space classification methods in this chapter
    1. Rocchio
        - Regions centered centroids / prototypes, computed as the center of mass of all documents in the class
        - Inaccurate if classes are not approximately *spheres* wil similar radii.
    2. kNN
        - The majority class of the $k$ nearest neighbors to a test document
        - Can handle non-spherical and complex classes