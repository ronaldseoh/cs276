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
    - Testing: $\Theta(L_a + \lvert \mathcal{C} \rvert M_a) = \Theta(\lvert \mathcal{C} \rvert M_a)$. $L_a$ and $M_a$ are the numbers of tokens and types(?? - number of distinct vocabs?) in the *test* document.

## 14.3 $k$ nearest neighbor

- Unlike Rocchio, kNN determines the decision boundary *locally*.
- We assign each document to the majority class of its $k$ closest neighbors.
    - Based on the contiguity hypothesis, we expect a test document $d$ to have the same label as the training documents located in the *local region* surrounding $d$.
- For $k \in \mathcal{N}$ in kNN, consider the region in the space for which the set of $k$ nearest neighbors is the *same*.
    - For $k=1$, such region is called *Voronoi cells*.
    - The space is partitioned into *convex polygons*, within each of which the set of $k$ nearest neighbors is *invariant*.
- 1NN is not very robust since a single training document might be incorrectly label or atypical.
- A probabilistic version of kNN: The probability of membership in class $c$ to be *the proportion* of the $k$ nearest neighbors in $c$.
- The parameter $k$ is often chosen based on experience or knowledge about the classification problem at hand.
    - Or we could choose it based on the performance on a held-out portion of the training set.
- We could also give weights to the votes of the $k$ neighbors with their cosine similarity
    - $\text{score}(c,d) = \sum_{d' \in S_k(d)} I_c(d') cos(\vec{v}(d'), \vec{v}(d))$
    - This is often more accurate than simple voting.

## 14.3.1 Time complexity and optimality of kNN

- Training a kNN consists of determining $k$ and preprocessing documents
    - Without preprocessing, no training at all.
- Test time is $\Theta(\lvert \mathcal{D} \rvert M_{\text{ave}} M_a)$
    - Linear in the size of the training set; since we need to compute the distance of each training document from the test document
    - Independent of the number of classes: a potential advantage for problems with large number of classes.
- kNN don't do parameter estimation: hence it is often called memory-based learning or *instance-based learning*.
    - Large training sets could come with a severe efficiency penalty.
- Could we make kNN more efficient than $\Theta(\lvert \mathcal{D} \rvert)$?
    - Fast kNN algorithms for small dimensionality $M$
    - Approximations for large $M$
    - Haven't been extensively tested for text classification applications
- Is the inverted index also the solution for efficient kNN? (Like ad hoc retrieval in Chapter 6.3.2)
    - The inverted index will be efficient if the test document has *no term overlap* with a large number of training documents.
    - Postings lists grow *sublinearly* with the length of the collection, since the vocabulary increases according to Heaps' law
        - If the probability of occurrence of some terms *increase*, then the probability of others *must decrease*.
    - However, most new terms are infrequent.
    - So the complexity of inverted index search to be $\Theta(T)$
    - Assuming average document length does not change over time, $\Theta(T) = \Theta(\lvert \mathcal{D} \rvert)$.
- kNN's effectiveness is close to that of the most accurate learning methods in text classification.
    - *Bayes error rate*: The average error rate of classifiers learned by a certain learning method, for a particular problem.
        - (According to Wikipedia) This is analogous to the *irreducible* error.
    - kNN is not optimal for problems with a non-zero Bayes error rate: problems that even its best possible classifier has a non-zero classification error.
    - The error rate of 1NN is asymptotically bounded by *2x* the Bayes error rate.
        - This is due to the effect of noise.
    - Noise affects two components of kNN: the test document and the *closest* training document.
        - The two sources of noise are additive
    - For problems with Bayes error rate of 0, the error rate of 1NN will approach 0 as the size of the training set increases.

## 14.4 Linear versus nonlinear classifiers

- We will only consider two-class classifiers
- Linear classifiers: two-class classifiers that decides class membership by comparing a linear combination of the features to a threshold.
- More formally: Assign to $c$ if $\vec{w}^T \vec{x} > b$, and to $\bar{c}$ if $\vec{w}^T \vec{x} \leq b$.
- Rocchio is a linear classifier:
    - A vector $\vec{x}$ is on the decision boundary if it has equal distance to the two class centroids: $\lvert \vec{\mu}(c_1) - \vec{x} \rvert = \lvert \vec{\mu}(c_2) - \vec{x} \rvert$
    - By manipulating the equation above, we can see that it corresponds to a linear classifier with $\vec{w} = \vec{\mu}(c_1) - \vec{\mu}(c_2)$ and $b = 0.5 \cdot (\lvert \vec{\mu}(c_1) \rvert^2 - \lvert \vec{\mu}(c_2) \rvert^2$.
- Naive Bayes is a linear classifier:
    - $\log \frac{\hat{P}(c \mid d)}{\hat{P}(\bar{c} \mid d)} = \log \frac{\hat{P}(c)}{\hat{P}(\bar{c})} + \sum_{1 \leq k \leq n_d} \log \frac{\hat{P}(t_k \mid c)}{\hat{P}(t_k \mid \bar{c})}$
    - We choose class $c$ if the odds are greater than 1 or the log odds are greater than 0.
    - $w_i = \log \frac{\hat{P}(t_i \mid c)}{\hat{P}(t_i \mid \bar{c})}$ and $b = - \log \frac{\hat{P}(c)}{\hat{P}(\bar{c})}$.
    - So in log space, Naive Bayes is a linear classifier.
- Noise features and noise documents
- If there exists a hyperplane that perfectly separates the two classes, then we call the two classes *linearly separable*.
    - If linear separability holds, then there is an *infinite* number of linear separators.
    - We need a criterion for selecting among all decision planes, since some will do well on new data, while some won't.
- An example of a *nonlinear* classifier is kNN.

## 14.5 Classification with more than two classes

- The method to use for multi-class classification depends on whether the classes are *mutually exclusive or not*.
    - *Any-of*, *Multilabel*, or *Multivalue* classification
        - Belong to everal classes simultaneously, or to a single class, or to none of the classes.
        - Build a classifier for each class. Given the test document, apply each classifier separately.
    - *One-of*, *Multinomial*, or *Single-label* classification
        - The classes are mutually exclusive. Each document must belong to exactly one of the classes.
        - A single classification function $\gamma$ in one-of classification whose range is $\mathcal{C}$
- True one-of problems are less common in text classification than any-of problems.
    - We will often make a one-of assumption, even if classes are not really mutually exclusive.
- Since $J$ hyperplanes do not divide $\mathcal{R}^{\lvert V \rvert}$ into $J$ distinct regions, we must use a combination method when using two-class classifiers for one-of classification.
    1. Rank classes and select top-ranked ones, by measuring distances from the $J$ linear separators: Since documents close to a separator are more likely to be misclassified, greater distance would indicate more plausibility
    2. Another way: Build and run classifiers for each class, and assign the document to the class with the maximum score/confidence value/probability.
- Confusion matrix: Can help pinpoint opportunities for improving the accuracy of the system

## 14.6 The bias-variance tradeoff

- Just because nonlinear classifiers are generally considered "more powerful", should we always use nonlinear classifiers for optimal effectiveness in statistical text classification?
- learning error = bias + variance
- Bias: The squared difference between $P(c \mid d)$ (the true conditional probability of $d$ being in $c$) and $\Gamma_{\mathcal{D}}(d)$ (the prediction of learned classifier, averaged over training sets)
    - Bias is large if the learning method produces classifiers that are consistently wrong.
    - We can think of bias as resulting from our domain knowledge that we build into the classifier.
    - Linear classsifier have high bias, while non-linear methods have low bias.
- Variance: The variation of the *prediction* of learned classifiers
    - Variance is large if different training sets $\mathcal{D}$ give rise to very different classifiers $\Gamma_{\mathcal{D}}$.
    - Linear methods low variance, nonlinear methods high variance
- Surprising that so many of the best-known text classification algorithms are linear.
    - With increased dimensionality, the likelihood of linear separability increases rapidly.