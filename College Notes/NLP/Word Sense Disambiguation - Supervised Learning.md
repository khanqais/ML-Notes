

> [!abstract] Topics covered
> - Supervised learning approach for WSD
> - Naive Bayes algorithm for WSD
> - Yarowsky bootstrapping approach

---

## 1. Supervised Learning Approach for WSD

### Definition

In the **supervised learning approach** to Word Sense Disambiguation (WSD), a machine-learning model is trained using a **sense-annotated corpus**. Each occurrence of an ambiguous word is manually labelled with its correct sense.

The trained model then predicts the correct sense of the word in a new sentence using its surrounding context.

### Example

```text
Sentence: "I deposited money in the bank."
Target word: bank
Correct sense: financial institution

Sentence: "We sat near the bank of the river."
Target word: bank
Correct sense: river side
```

### WSD as classification

Supervised WSD treats sense disambiguation as a classification problem.

```text
Input:  target word + context
Classes: possible senses of the target word
Output: most likely sense
```

For the word `bank`:

```text
Class 1: financial institution
Class 2: river side
```

### Training data

Supervised WSD requires **sense-tagged text**. Every occurrence of the target word is labelled with its correct sense.

| Sentence | Target word | Sense label |
|---|---|---|
| “I deposited money in the bank.” | bank | financial institution |
| “The bank approved my loan.” | bank | financial institution |
| “We sat on the bank of the river.” | bank | river side |
| “The river bank was muddy.” | bank | river side |

### Feature extraction

Each training example is converted into a feature vector. The classifier learns the relationship between features and the correct sense.

| Feature | Explanation | Example |
|---|---|---|
| Surrounding words | Words near the target word | `money`, `deposit`, `loan` |
| POS tag | Grammatical category of the target word | `bank` is a noun |
| Collocations | Frequently occurring word combinations | `river bank`, `savings bank` |
| Co-occurrence words | Words that often appear with a sense | `deposit`, `account`, `money` |
| Position features | Words at fixed positions around target word | Previous word, next word |
| Topic/domain features | General topic of the sentence | finance, river, nature |
| Word embeddings | Dense numerical vectors for words | Vector for `money`, `loan`, `deposit` |

### Training phase

```text
1. Collect sentences containing the target word.
2. Manually label each occurrence with its correct sense.
3. Tokenize and preprocess the text.
4. Extract features such as surrounding words, POS tags, collocations, and embeddings.
5. Convert each example into a feature vector.
6. Train a classifier using feature vectors and sense labels.
```

### Testing phase

```text
1. Take a new sentence containing the ambiguous word.
2. Extract the same features used during training.
3. Convert the sentence into a feature vector.
4. Give the vector to the trained classifier.
5. The classifier predicts the most probable sense.
```

### Example

```text
New sentence:
"He opened an account at the bank."

Predicted sense:
bank = financial institution
```

### Common classifiers

| Algorithm | Working principle |
|---|---|
| Naive Bayes | Calculates probability of each sense using context features |
| Decision tree | Uses feature-based rules |
| Decision list | Applies ordered if-then rules |
| Support Vector Machine | Separates examples of different senses |
| Neural network | Learns nonlinear patterns |
| Transformer model | Uses contextual embeddings |

### Advantages

- High accuracy when sufficient labelled data is available
- Learns context-specific patterns automatically
- Can use rich features such as POS tags, collocations, and embeddings
- Works well for frequently occurring ambiguous words

### Limitations

- Requires a large sense-annotated dataset
- Manual annotation is expensive and time-consuming
- A separate classifier may be needed for each target word
- Poor performance for rare words or unseen senses

> [!tip] Interview-ready answer
> In supervised WSD, we treat sense disambiguation as a classification problem. A sense-annotated corpus is used to train a classifier, where each occurrence of an ambiguous word is labelled with its correct sense. Features such as surrounding words, POS tags, collocations, co-occurring words, and word embeddings are extracted and converted into feature vectors. During training, the classifier learns the relationship between these features and the correct sense. During testing, it predicts the most likely sense for a new occurrence.

---

## 2. Naive Bayes Algorithm for WSD

### Definition

The **Naive Bayes supervised algorithm** is a probabilistic classification method used for WSD. It learns from sense-tagged examples and predicts the most probable sense of an ambiguous word using its context.

It is called:

- **Naive** because it assumes all context features are independent given the sense
- **Bayes** because it uses Bayes’ theorem

### Basic idea

For an ambiguous word \(w\), the algorithm has:

```text
Target word: bank
Possible senses: financial institution, river side
Features: context words, POS tags, collocations
Output: most likely sense
```

The classifier calculates:

\[
P(\text{sense} \mid \text{features})
\]

and selects the sense with the maximum probability.

### Bayes theorem

\[
P(S \mid F) = \frac{P(F \mid S)P(S)}{P(F)}
\]

Where:

- \(S\) = possible sense of the target word
- \(F\) = observed context features
- \(P(S)\) = prior probability of the sense
- \(P(F \mid S)\) = likelihood of features given the sense
- \(P(F)\) = probability of the features

Since \(P(F)\) is the same for all senses, the algorithm compares:

\[
P(S \mid F) \propto P(F \mid S)P(S)
\]

### Naive independence assumption

The algorithm assumes that context features are conditionally independent given the sense:

\[
P(f_1, f_2, ..., f_n \mid S) =
P(f_1 \mid S)P(f_2 \mid S)...P(f_n \mid S)
\]

For example, for the word `bank`:

```text
Features: deposit, money, loan
Sense: financial institution
```

Naive Bayes assumes:

```text
P(deposit, money, loan | financial institution)
= P(deposit | financial institution)
× P(money | financial institution)
× P(loan | financial institution)
```

### Training phase

The training data contains sentences where the ambiguous word is already labelled with its correct sense.

| Sentence | Sense label |
|---|---|
| “I deposited money in the bank.” | financial institution |
| “The bank approved my loan.” | financial institution |
| “We sat on the bank of the river.” | river side |
| “The river bank was muddy.” | river side |

#### Steps

```text
1. Collect sense-tagged sentences.
2. Identify the target word in each sentence.
3. Extract context features around the target word.
4. Remove stop words if required.
5. Count how often each feature occurs with each sense.
6. Calculate prior probabilities P(S).
7. Calculate conditional probabilities P(feature | sense).
8. Store these probabilities for testing.
```

### Prior probability

\[
P(S) = \frac{\text{number of examples with sense } S}
{\text{total number of training examples}}
\]

Example:

```text
60 examples: financial institution
40 examples: river side
```

\[
P(\text{financial institution}) = \frac{60}{100} = 0.6
\]

\[
P(\text{river side}) = \frac{40}{100} = 0.4
\]

### Conditional probability

\[
P(f \mid S) =
\frac{\text{number of times feature } f \text{ occurs with sense } S}
{\text{total number of features in sense } S}
\]

Example:

```text
P(money | financial institution) = 0.30
P(money | river side) = 0.01
```

This means `money` is much more likely to occur with the financial-institution sense of `bank`.

### Testing phase

```text
1. Take the new sentence containing the ambiguous word.
2. Extract context features.
3. For each possible sense:
       Calculate P(S)
       Calculate P(feature | S) for every feature
       Multiply all feature probabilities
       Multiply by P(S)
4. Select the sense with the highest score.
```

### Formula

For features \(f_1, f_2, ..., f_n\):

\[
Score(S) =
P(S) \times P(f_1 \mid S) \times P(f_2 \mid S) \times ... \times P(f_n \mid S)
\]

### Example: bank

Test sentence:

```text
"He deposited money in the bank."
```

Target word:

```text
bank
```

Context features after stop-word removal:

```text
deposited, money
```

Possible senses:

```text
Sense 1: financial institution
Sense 2: river side
```

| Feature | P(feature | financial institution) | P(feature | river side) |
|---|---:|---:|
| deposited | 0.30 | 0.02 |
| money | 0.40 | 0.01 |

Prior probabilities:

```text
P(financial institution) = 0.6
P(river side) = 0.4
```

Calculate scores:

\[
Score(\text{financial institution})
= 0.6 \times 0.30 \times 0.40
= 0.072
\]

\[
Score(\text{river side})
= 0.4 \times 0.02 \times 0.01
= 0.00008
\]

Since:

```text
0.072 > 0.00008
```

The predicted sense is:

```text
bank = financial institution
```

### Zero probability problem

If a feature never appeared with a sense during training:

\[
P(f \mid S) = 0
\]

Multiplying by zero makes the whole score zero.

To avoid this, use **Laplace smoothing**:

\[
P(f \mid S) =
\frac{\text{count}(f, S) + 1}
{\text{total features in sense } S + V}
\]

Where \(V\) is the vocabulary size.

### Features used

| Feature | Example |
|---|---|
| Previous word | Word before the target word |
| Next word | Word after the target word |
| Bag of words | All words in a context window |
| Collocations | Fixed word combinations such as `river bank` |
| POS tags | Noun, verb, adjective |
| Co-occurrence words | `money`, `deposit`, `loan` |
| Word embeddings | Dense numerical vectors |

### Advantages

- Simple and easy to implement
- Fast training and prediction
- Works well with limited training data
- Handles many features efficiently
- Provides probabilistic output

### Limitations

- Independence assumption is unrealistic
- Context words are often related
- May perform poorly with small or biased training data
- Rare senses may not be predicted well

> [!tip] Interview-ready answer
> Naive Bayes is a supervised probabilistic algorithm for WSD. It treats each possible sense of an ambiguous word as a class and uses context features such as surrounding words, POS tags, and collocations. During training, it calculates the prior probability of each sense and the conditional probability of each feature given that sense. During testing, it applies Bayes’ theorem and selects the sense with the highest posterior probability. It assumes that context features are independent given the sense, which is why it is called “naive.”

---

## 3. Yarowsky Bootstrapping Approach

### Definition

The **Yarowsky bootstrapping approach** is a **semi-supervised learning** method used mainly for Word Sense Disambiguation (WSD). It starts with a small set of manually labelled seed examples and gradually labels a large amount of unlabelled text using the model’s own high-confidence predictions.

It is called **bootstrapping** because the system starts small and repeatedly improves itself by adding newly labelled examples to the training set.

### Why it is semi-supervised

It uses:

```text
Small labelled seed data
+
Large unlabelled corpus
```

This reduces the need for expensive manual sense annotation.

```text
Small labelled seed data
        ↓
Initial classifier
        ↓
Label high-confidence unlabelled examples
        ↓
Add them to training data
        ↓
Retrain classifier
        ↓
Repeat until convergence
```

### Two key principles

#### One sense per collocation

A word tends to have the same sense when it occurs with the same nearby words or collocations.

```text
river bank      → river side
savings bank    → financial institution
bank account    → financial institution
bank loan       → financial institution
```

So, if `bank` appears with `river`, `water`, or `shore`, it likely means **river side**. If it appears with `money`, `deposit`, `loan`, or `account`, it likely means **financial institution**.

#### One sense per discourse

Within the same document or discourse, an ambiguous word usually keeps the same sense.

Example:

```text
"The bank approved the loan. The bank also increased interest rates."
```

Both uses of `bank` are likely to mean **financial institution**.

### Algorithm steps

#### Step 1: Start with seed examples

Choose a small number of manually labelled examples for each sense.

For the word `bank`:

| Sense | Seed examples |
|---|---|
| Financial institution | “I deposited money in the bank.” |
| River side | “We sat on the bank of the river.” |

#### Step 2: Extract features

For every occurrence of the target word in the unlabelled corpus, extract features such as:

```text
- Nearby words
- Collocations
- Previous and next words
- POS tags
- Sentence-level context
- Document-level context
```

Example:

```text
Sentence: "The bank approved my loan."

Target word: bank
Features: approved, my, loan, finance-related words
```

#### Step 3: Build an initial classifier

Using the seed examples, train a simple classifier. Yarowsky commonly uses a **decision-list learner**.

A decision list contains rules such as:

```text
If context contains "river" → river side
If context contains "deposit" → financial institution
If context contains "loan" → financial institution
```

#### Step 4: Label unlabelled examples

Apply the current classifier to unlabelled examples. Assign a sense only when the classifier is confident enough.

```text
If P(financial institution | context) > threshold
→ label bank as financial institution
```

Low-confidence examples remain unlabelled for the next iteration.

#### Step 5: Add confident examples to training data

The newly labelled high-confidence examples are added to the seed set.

```text
Initial seed data + newly labelled examples
→ enlarged labelled training set
```

#### Step 6: Repeat until convergence

```text
Train classifier
→ label unlabelled data
→ select high-confidence examples
→ add them to training set
→ retrain classifier
```

The algorithm stops when no new reliable examples or rules can be added, or when labels become stable.

### Example: bank

Suppose the target word is `bank`, with two senses:

```text
Sense A: financial institution
Sense B: river side
```

#### Initial seed examples

```text
"I deposited money in the bank." → financial institution
"We sat on the bank of the river." → river side
```

#### First iteration

The classifier learns rules:

```text
money → financial institution
deposit → financial institution
river → river side
water → river side
```

It then labels unlabelled sentences:

```text
"The bank approved my loan." → financial institution
"The river bank was muddy." → river side
```

#### Second iteration

The newly labelled examples are added to training data:

```text
loan → financial institution
approved → financial institution
muddy → river side
```

The classifier becomes stronger and can label more unlabelled examples.

### Decision-list learning

Yarowsky uses a decision-list learner that selects high-precision rules.

A rule has the form:

```text
If feature f occurs, predict sense j
```

Examples:

```text
If "loan" occurs → financial institution
If "river" occurs → river side
```

The algorithm calculates confidence using:

\[
P(\text{sense} \mid \text{feature})
\]

Rules with the highest confidence are placed at the top of the decision list.

### Advantages

- Requires only a small labelled seed set
- Can use a large amount of unlabelled text
- Reduces manual annotation cost
- Iteratively improves the classifier
- Useful when labelled WSD data is scarce

### Limitations

- Initial seed examples must be carefully chosen
- Wrong early labels can propagate and reduce accuracy
- May become biased toward the majority sense
- Needs confidence thresholds and stopping criteria
- Works best when the two Yarowsky assumptions hold

> [!tip] Interview-ready answer
> Yarowsky bootstrapping is a semi-supervised WSD method that starts with a small set of manually labelled seed examples and a large unlabelled corpus. It trains an initial classifier on the seed examples, applies it to unlabelled data, and adds only high-confidence predictions to the labelled set. This enlarged set is used to retrain the classifier, and the process repeats. It relies on two assumptions: **one sense per collocation** and **one sense per discourse**.

---

## Quick Revision

```text
Supervised WSD → trains classifier using sense-tagged corpus
Features → surrounding words, POS tags, collocations, embeddings
Naive Bayes → probabilistic classifier using Bayes theorem
Naive assumption → features are independent given the sense
Yarowsky → semi-supervised bootstrapping using seed data
One sense per collocation → same nearby words imply same sense
One sense per discourse → same sense within a document
```