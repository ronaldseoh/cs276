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

