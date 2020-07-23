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

## 14.2 Rocchio classification

- *Decision Boundaries*: To classify a new document, we determine the region it occurs in and assign it the class of that region.
- Our task in vector space classification is to devise algorithms that compute good boundaries, where "good" means high classification accuracy on data *unseen during training*.
- *Rocchio classification*
    - The centroid of a class $c$ is the vector average or center of mass of its members: $\vec{\mu}(c) = \frac{1}{\lvert D_c \rvert} \sum_{d \in D_c} \vec{v}(d)$
        - $D_c$ is the set of documents in $\mathcal{D}$ whose class is $c$: $D_c = \lbrace d: <d, c> \in \mathcal{D} \rbrace$
        - $\vec{v}(d)$ is the normalized vector of $d$.
    - The boundary between two classes in Rocchio classification is the set of *points* with *equal distance* from the two centroids, which is always a *line*.
- The generalization of a line in $M$-dimensional space is a *hyperplane*, which we define as the set of points $\vec{x}$ that satisfy $\vec{w}^T \vec{x} = b$. $\vec{w}$ is the $M$-dimensional normal vector of the hyperplane, and $b$ is a constant.
    - Thus, the boundaries of class regions in Rocchio are hyperplanes.
- The assignment criterion is Euclidean distance; an alternative is cosine similarity.
- The classes in Rocchio classification must be approximate spheres with similar radii.
    - Ignores details of the distribuion of points in a class and only uses distance from the centroid for classification
    - Often misclassifies multimodal classes
    - Two-class classification problems rarely have classes distributed like spheres with similar radii.
- Time complexity:
    - Adding up document vectors: $\Theta(\lvert \mathcal{D} \rvert L_{\text{ave}})$ ($L_{\text{ave}}$ instead of $\lvert V \rvert$ as we only need to consider non-zero entries)
    - Dividing each vector sum by the size of its class: $\Theta(\lvert \mathcal{C} \rvert \lvert V \rvert)$
    - Testing: $\Theta(L_a + \lvert \mathcal{C} \rvert M_a) = \Theta(\lvert \mathcal{C} \rvert M_a)$. $L_a$ and $M_a$ are the numbers of tokens and types(??) in the *test* document.