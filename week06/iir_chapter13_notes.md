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

## 13.2 Naive Bayes text classification

- *Multinomial Naive Bayes*: $P(c \mid d) \propto P(c) \prod_{1 \leq k \leq n_d} P(t_k \mid c)$.
    - $P(t_k \mid c)$: A measure of how much $t_k$ contributes that $c$ is the correct class.
    - $n_d$: The number of tokens in $d$ (which are part of the vocabulary) that we use for classification.
- If a document's terms do not provide clear evidence for one particular class over another, then
    - we choose the one that has a higher prior probability.
- *Maximum a Posteriori (MAP)* class: The best class in NB classification is the most *likely* class
    - $c_{\text{map}} = arg\,max_{c \in \mathcal{C}} \hat{P}(c \mid d) = arg\,max_{c \in \mathcal{C}} \hat{P}(c) \prod_{1 \leq k \leq n_d} \hat{P}(t_k \mid c)$
- Multiplying many probabilities could cause a floating point underflow; hence it is better to add up *logarithms* of probabilities instead
    - The logarithm function is monotonic.
    - $c_{\text{map}} = arg\,max_{c \in \mathcal{C}} \lbrack \log \hat{P}(c) + \sum_{1 \leq k \leq n_d} \log \hat{P} (t_k \mid c) \rbrack$
- How do we estimate the parameters $\hat{P}(c)$ and $\hat{P}(t_k \mid c)$?
    1. Maximum Likelihood Estimate (MLE):
        - $\hat{P}(c) = \frac{N_c}{N}$
        - $\hat{P}(t_k \mid c) = \frac{T_{ct}}{\sum_{t' \in V} T_{ct'}}$
        - We have a positional independence assumption: $\hat{P}(t_{k_1} \mid c) = \hat{P}(t_{k_2} \mid c)$ for any $k_1$ and $k_2$.
    - The problem: Zero for a term-class combination that did not occur in the training data
        - *Sparseness*: The training data are never large enough to represent the frequency of rare events adequately.
    - *Laplace smoothing*: Simply add 1 to each count; can be interpreted as a uniform prior (of the term occurrence) and then updated as evidence from training data comes in
        - $\hat{P}(t \mid c) = \frac{T_{ct} + 1}{(\sum_{t' \in V} T_{ct'}) + B'}$, where $B=\lvert V \rvert$ is the number of terms in the vocabulary.
- Time complexity of NB
    - Computing the parameters: $\Theta(\lvert \mathcal{C} \rvert \lvert V \rvert)$ as the set of parameters consists of $\lvert \mathcal{C} \rvert \lvert V \rvert$ additional parameters and $\lvert C \rvert$ priors.
    - Preprocessing for parameters computation: can be done in *one pass* through the training data. $\Theta(\lvert \mathcal{D} \rvert L_{ave})$.
        - $\lvert \mathcal{D} \rvert$ is the number of documents
        - $L_{ave}$ is the average length of a document
    - `ApplyMultinomialNB`:
        - $L_a$: The number of tokens, $M_a$: The number of types, in the test document
        - The version we have in the text: $\Theta(\lvert \mathcal{C} \rvert L_a)$
        - But we could modify this to be $\Theta(L_a + \lvert \mathcal{C} \rvert M_a)$
        - And $\Theta(L_a + \lvert \mathcal{C} \rvert M_a) = \Theta(\lvert \mathcal{C} \rvert M_a)$ as $L_a < b \lvert C \rvert M_a$ for a fixed constant $b$.
    - Since $\lvert \mathcal{C} \rvert \lvert V \rvert < \lvert \mathcal{D} \rvert L_{ave}$, both training and testing complexity are linear in the time it takes to *scan the data*.

## 13.3 The Bernoulli model

- Instead of the multinomial model, the multivariate Bernoulli model
    - An indicator for each term of the vocabulary, either 1 indicating presence of the term in the document or 0 indicating absence
- Estimates $\hat{P}(t \mid c)$ as the fraction of *documents* of class $c$ that contain term $t$, not the number of tokens
- Makes many mistakes when classifying long documents.
- Unlike the multinomial model, the probability of nonoccurrence is factored in when computing $P(c \mid d)$.

## 13.4 Properties of Naive Bayes

- In both models, we assigned classes to documents based on the maximum a posteriori probability
    - $c_{map} = arg\,max_{c \in \mathcal{C}} P(c \mid d) = arg\,max_{c \in \mathcal{C}} \frac{P(d \mid c) P(c)}{P(d)} = arg\,max_{c \in \mathcal{C}} P(d \mid c) P(c)$
    - $P(d)$ is dropped as it's same for all classes.
- We can think of the equation above as a description of the process of generating text.
    1. We first choose class $c$ with probability $P(c)$.
    2. We generate the model following one of the probabilities below:
        - Multinomial: $P(d \mid c) = P(<t_1, \cdots, t_k, \cdots, t_{n_d}> \mid c)$
        - Bernoulli: $P(d \mid c) = P(<e_1, \cdots, e_i, \cdots, e_M> \mid c)$
- Naive Bayes *conditional independence assumption*
    - Multinomial: $P(d \mid c) = P(<t_1, \cdots, t_k, \cdots, t_{n_d}> \mid c) = \prod_{1 \leq k \leq n_d} P(x_k = t_k \mid c)$
        - $X_k$ is the random variable for position $k$ and $P(x_k = t_k \mid c)$ is the probability of the term $t$ occurring in position $k$, inside a document of class $c$.
    - Bernoulli: $P(d \mid c) = P(<e_1, \cdots, e_i, \cdots, e_M> \mid c) = \prod_{1 \leq i \leq M} P(U_i = e_i \mid c)$
        - $U_i$ is the random variable for vocabulary term $i$ and $P(U_i = e_i \mid c)$ is the probability that the term $t_i$ will occur in a document of class $c$, regardless of positions and the number of times it occur.
- Position Independence for Multinomial NB:
    - One single distribution for the conditional probabilities for a term, regardless of its position within a document.
        - $P(X_{k_1} = t \mid c) = P(X_{k_2} = t \mid c)$
    - Equivalent to adopting the bag of words model
- With the two independence assumptions, we only need to estimate $\Theta(M \lvert \mathcal{C} \rvert)$ parameters.
- For a completely specified document generation model, we would have to define a distribution $P(n_d \mid c)$ over *lengths*.
    - Without this, our multinomial model is a *token* generation model rather than a document generation model.
- The probability estimates of NB are of low quality, its classification decisions are surprisingly good.
    - The winning class in NB classification usually has a much larger probability than the other classes.
    - Correct estimation implies accurate prediction, but *accurate prediction does not imply correct estimation*.
- NB excels when
    - There are many *equally* important features that jointly contribute to the classification decision.
    - Robust to noise features
    - Robust to *concept drift* - the gradual change over time of the concept underlying a class. (e.g. US president)
        - Unlike k-NNs, which can be carefully tuned to idiosyncratic properties of a *particular time period*.
        - Bernoulli model is particularly robust to this.
        - Decent performance when using fewer than a dozen terms, since the most important indicators for a class are less likely to change.
        - So a model that only relies on these features is more likely to maintain a certain level of accuracy despite concept drift.
    - Efficiency: Training and classification can be accomplished with one pass over data.
        - Baseline in text classification research
        - Good if squeezing out a few more points of accuracy is not very important
        - Good if the training data is huge and we could learn more by training on a lot of data rather than a better classififer on a small dataset

### 13.4.1 A variant of the multinomial model

- Represent a document as an $M$-dimensional vector of counts $<tf_{t_1, d}, \cdots, tf_{t_M, d}>$