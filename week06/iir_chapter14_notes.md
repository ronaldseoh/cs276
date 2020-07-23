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

## 14.1 Document representations and measures of relatedness in vector spaces

- Document vectors are length-normalized unit vectors that point to the surface of a hypersphere.
- Distances on the surface of the spehere and on the projection plane are *approximately the same*
    - as long as we restrict ourselves to small areas of the surface
    - and choose an appropriate projection.
- Decisions of many vector space classifiers are based on a notion of distance.
    - We use Euclidean distance
- For unnormalized vectors, dot product, cosine similarity and Euclidean distance all have different behavior in general.
    - Smaller the region when computing the similarity, more similar their behaviors are.