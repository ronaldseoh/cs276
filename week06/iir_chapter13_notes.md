# Chapter 13: Text classification and Naive Bayes

- Many users have *ongoing* information needs.
- *Standing queries*: Queries periodically executed on a collection to which new documents are incrementally added over time.
    - To achieve good recall, standing queries would have to be refined over time and gradually become quite complex.
    - Often called *routing* or *filtering*
- *Classification* problem
    - Two-class classification: documents *about* the query and documents *not about* the query.
    - Classes can be more general subject areas, usually referred to as *topics*; *Text classification*, *Topic classification*, etc.
- Application of classification in IR
    - Preprocessing for indexing
    - Automatic detection of spam pages
    - Detecting sexually explicit content
    - Sentiment Detection
    - Personal email sorting
    - Topic-specific search
    - Ranking function in ad hoc information retrieval
- Manual classification, Rule-based classification (very difficult to write those rules), ML-based classification, Statistical text classification
    - The need for manual classification is not completely eliminated, as the training documents come from a person who has labeled them.
    - Labeling is easier than writing rules!

## 13.1 The text classification problem

- Given a description $d \in \mathcal{X}$ of a document
    - $\mathcal{X}$ is some high-dimensional document space
- And a fixed set of classes $\mathcal{C} = \lbrace c_1, c_2, \cdots, c_J \rbrace$
    - Human defined classes for the needs of an application
- Given a *training* set $\mathcal{D}$ of labeled documents $<d, c>$, where $<d,c> \in \mathcal{X} \times \mathcal{C}$.
- Using a learning algorithm, we wish to learn a classification function $\gamma$ that maps documents to classes: $\gamma: \mathcal{X} \rightarrow \mathcal{C}$
    - Supervised learning
    - Our goal in text classification is high accuracy on test data
- The classes in text classification often have some interesting structure such as the *hierarchy*, which can be an important aid in solving text classification problem
    - For now, we assume that the classes form a set with no subset relationships between them
- We assume that a document is a member of *exactly one* class.
    - This is not the most appropriate model for a hierarchy
        - *Any-of* problem: A document belonging to more than one class
        - *One-of* problem: A document belonging to exactly one class
- When we use the training set to learn a classifier for test data, we make the assumption that training data and test data are similar or from the same distribution.

