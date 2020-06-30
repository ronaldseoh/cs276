# Chapter 8: Evaluation in information retrieval

- How do we determine whether certain IR system is effective or not?
- IR is a highly *empirical discipline*
- User utility? User happiness?

## 8.1 Information retrieval system evaluation

- *Relevant* and *nonrelevant* documents
- Gold standard or ground truth judgment of relevance
- A test suite of information needs to be of a reasonable size: around 50 information needs has usually been found to be a sufficient minimum.
- Relevance is assessed relative to an information need, *not a query*.
- To evaluate a system, we require an *overt expression* of an information need.
- We will assume a binary decision of relevance.
- Wrong to report results on a test collection after tuning the parameters against it: Need to use one or more *development test collections* instead for tuning

