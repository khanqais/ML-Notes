## Topics covered

- Supervised learning approach for WSD
- Naive Bayes algorithm for WSD
- Yarowsky bootstrapping approach

---

## 1. Supervised Learning Approach for WSD

### Definition

In the supervised learning approach to **Word Sense Disambiguation (WSD)**, a machine-learning model is trained using a **sense-annotated corpus**. Each occurrence of an ambiguous word is manually labelled with its correct sense.

The trained model then predicts the correct sense of the word in a new sentence using its surrounding context.

### Example

**Sentence:** "I deposited money in the bank."
- Target word: `bank`
- Correct sense: financial institution

**Sentence:** "We sat near the bank of the river."
- Target word: `bank`
- Correct sense: river side

### WSD as classification

Supervised WSD treats sense disambiguation as a **classification problem**.

```text
Input:   target word + context
Classes: possible senses of the target word
Output:  most likely sense
```

For the word `bank`:

- Class 1: financial institution
- Class 2: river side

### Training data

Supervised WSD requires sense-tagged text. Every occurrence of the target word is labelled with its correct sense.

| Sentence | Target word | Sense label |
|---|---|---|
| "I deposited money in the bank." | bank | financial institution |
| "The bank approved my loan." | bank | financial institution |
| "We sat on the bank of the river." | bank | river side |
| "The river bank was muddy." | bank | river side |

### Feature extraction

Each training example is converted into a **feature vector**. The classifier learns the relationship between features and the correct sense.

| Feature | Explanation | Example |
|---|---|---|
| Surrounding words | Words near the target word | money, deposit, loan |
| POS tag | Grammatical category of the target word | bank is a noun |
| Collocations | Frequently occurring word combinations | river bank, savings bank |
| Co-occurrence words | Words that often appear with a sense | deposit, account, money |
| Position features | Words at fixed positions around the target word | previous word, next word |
| Topic/domain features | General topic of the sentence | finance, river, nature |
| Word embeddings | Dense numerical vectors for words | vector for money, loan, deposit |

### Training phase

1. Collect sentences containing the target word.
2. Manually label each occurrence with its correct sense.
3. Tokenize and preprocess the text.
4. Extract features such as surrounding words, POS tags, collocations, and embeddings.
5. Convert each example into a feature vector.
6. Train a classifier using the feature vectors and sense labels.

### Testing phase

1. Take a new sentence containing the ambiguous word.
2. Extract the same features used during training.
3. Convert the sentence into a feature vector.
4. Give the vector to the trained classifier.
5. The classifier predicts the most probable sense.

**Example**

New sentence: "He opened an account at the bank."

Predicted sense: `bank` = financial institution

### Common classifiers

| Algorithm | Working principle |
|---|---|
| Naive Bayes | Calculates the probability of each sense using context features |
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

### Interview-ready answer

> In supervised WSD, we treat sense disambiguation as a classification problem. A sense-annotated corpus is used to train a classifier, where each occurrence of an ambiguous word is labelled with its correct sense. Features such as surrounding words, POS tags, collocations, co-occurring words, and word embeddings are extracted and converted into feature vectors. During training, the classifier learns the relationship between these features and the correct sense. During testing, it predicts the most likely sense for a new occurrence.

---

## 2. Naive Bayes Algorithm for WSD

### Definition

The **Naive Bayes** supervised algorithm is a probabilistic classification method used for WSD. It learns from sense-tagged examples and predicts the most probable sense of an ambiguous word using its context.

It is called:

- **Naive** because it assumes all context features are independent of each other, given the sense
- **Bayes** because it uses Bayes' theorem

### Basic idea

For an ambiguous word $w$, the algorithm has:

- **Target word:** `bank`
- **Possible senses:** financial institution, river side
- **Features:** context words, POS tags, collocations
- **Output:** most likely sense

The classifier calculates:

$$
P(\text{sense} \mid \text{features})
$$

and selects the sense with the **maximum** probability.

### Bayes' theorem

$$
P(S \mid F) = \frac{P(F \mid S) \, P(S)}{P(F)}
$$

Where:

- $S$ is a possible sense of the target word
- $F$ is the set of observed context features
- $P(S)$ is the **prior probability** of the sense (how common the sense is overall)
- $P(F \mid S)$ is the **likelihood** of the features given the sense
- $P(F)$ is the probability of the features
- $P(S \mid F)$ is the **posterior probability** of the sense given the features (what we want)

Since $P(F)$ is the same for every sense, it does not affect which sense wins. So the algorithm only compares:

$$
P(S \mid F) \propto P(F \mid S) \, P(S)
$$

where $\propto$ means "is proportional to".

### Naive independence assumption

The algorithm assumes that context features are **conditionally independent** given the sense:

$$
P(f_1, f_2, \ldots, f_n \mid S) = P(f_1 \mid S) \, P(f_2 \mid S) \cdots P(f_n \mid S)
$$

This can be written compactly as:

$$
P(f_1, \ldots, f_n \mid S) = \prod_{j=1}^{n} P(f_j \mid S)
$$

**Example** for the word `bank`:

- Features: `deposit`, `money`, `loan`
- Sense: financial institution

Naive Bayes assumes:

$$
P(\text{deposit}, \text{money}, \text{loan} \mid \text{financial institution}) = P(\text{deposit} \mid \text{financial institution}) \times P(\text{money} \mid \text{financial institution}) \times P(\text{loan} \mid \text{financial institution})
$$

The assumption is not strictly true (`money` and `loan` often appear together), but it makes the model simple and it works well in practice.

### Decision rule

Combining Bayes' theorem with the independence assumption, the predicted sense $\hat{S}$ is:

$$
\hat{S} = \underset{S}{\arg\max} \; P(S) \prod_{j=1}^{n} P(f_j \mid S)
$$

$\arg\max$ means "choose the sense $S$ that gives the largest value".

### Training phase

The training data contains sentences where the ambiguous word is already labelled with its correct sense. From this data, the algorithm estimates two kinds of probabilities by counting.

**1. Prior probability of each sense**

$$
P(S) = \frac{C(S)}{C(\text{all examples})}
$$

where $C(S)$ is the number of training examples labelled with sense $S$.

**2. Likelihood of each feature given a sense**

$$
P(f_j \mid S) = \frac{C(f_j, S)}{C(S)}
$$

where $C(f_j, S)$ is the number of times feature $f_j$ appears in examples labelled with sense $S$.

**Smoothing:** if a feature never appears with a sense, $P(f_j \mid S) = 0$ and the whole product becomes zero. To avoid this, use Laplace (add-one) smoothing:

$$
P(f_j \mid S) = \frac{C(f_j, S) + 1}{C(S) + V}
$$

where $V$ is the number of distinct features (vocabulary size).

### Testing phase

1. Take a new sentence containing the ambiguous word.
2. Extract its context features $f_1, \ldots, f_n$.
3. For each sense $S$, compute $P(S) \prod_j P(f_j \mid S)$.
4. Choose the sense with the highest score.

### Worked example

Suppose the training data for `bank` gives these probabilities:

| Quantity | financial institution | river side |
|---|---:|---:|
| $P(S)$ | 0.6 | 0.4 |
| $P(\text{deposit} \mid S)$ | 0.4 | 0.05 |
| $P(\text{money} \mid S)$ | 0.5 | 0.1 |

New sentence: "I want to **deposit** **money** in the bank."

Features: `deposit`, `money`.

Score for **financial institution**:

$$
0.6 \times 0.4 \times 0.5 = 0.12
$$

Score for **river side**:

$$
0.4 \times 0.05 \times 0.1 = 0.002
$$

Since $0.12 > 0.002$, the predicted sense is **financial institution**.

### Advantages

- Simple and fast to train
- Works well with small to medium amounts of labelled data
- Handles many features easily
- Gives probabilities, not just a label

### Limitations

- The independence assumption is unrealistic
- Needs sense-annotated training data
- Needs smoothing for unseen features
- A separate model is usually needed for each target word

## 3. Lesk Algorithm for WSD

### Definition

The **Lesk algorithm** (Michael Lesk, 1986) is a **dictionary-based (knowledge-based)** approach to Word Sense Disambiguation. It does **not** need a sense-tagged corpus. Instead, it uses the **definitions (glosses)** in a dictionary such as WordNet.

**Core idea:** the correct sense of an ambiguous word is the one whose dictionary definition shares the **most words** with the surrounding context.

> Words that appear together in a sentence tend to talk about the same topic, so the right sense should have a definition that uses related words.

### Key terms

| Term | Meaning |
|---|---|
| Target word | The ambiguous word to disambiguate (e.g. `bank`) |
| Sense | One possible meaning of the word |
| Gloss | The dictionary definition of a sense |
| Context | The words surrounding the target word (sentence or a window) |
| Overlap | Number of words shared between two bags of words |
| Signature | The bag of words used to represent a sense (its gloss, possibly expanded) |

### Basic formula

For a target word $w$ with senses $s_1, s_2, \ldots, s_m$, the score of each sense is the overlap between its signature and the context:

$$
\text{score}(s_i) = \left| \text{signature}(s_i) \cap \text{context}(w) \right|
$$

The chosen sense is the one with the highest score:

$$
\hat{s} = \underset{s_i \in \text{Senses}(w)}{\arg\max} \; \text{score}(s_i)
$$

Here $|A \cap B|$ means the number of words common to both sets $A$ and $B$.

---

### Steps of the (Simplified) Lesk algorithm

We use this running example:

**Sentence:** "I deposited money in the bank."
**Target word:** `bank`

#### Step 1: Identify the target word

Pick the ambiguous word to be disambiguated.

```text
Target word = bank
```

#### Step 2: Retrieve all senses and glosses from the dictionary

Look up every sense of the target word. (Glosses below are simplified from WordNet.)

| Sense | Gloss |
|---|---|
| bank#1 | a financial institution that accepts deposits and channels the money into lending activities |
| bank#2 | sloping land beside a body of water |
| bank#3 | a building in which the business of banking is transacted |

#### Step 3: Extract the context

Take the words around the target word. The context can be the whole sentence or a window (for example, 5 words on each side).

```text
Context = I, deposited, money, in, the
```

(The target word itself is not part of the context.)

#### Step 4: Preprocess both glosses and context

Apply these to the context and to every gloss:

1. Convert to lowercase.
2. Remove **stop words** (I, in, the, a, of, that, and, into, ...).
3. **Lemmatize or stem** words (deposited → deposit, deposits → deposit, accepts → accept).
4. Remove the target word itself.

Result:

| Item | Preprocessed bag of words |
|---|---|
| Context | {deposit, money} |
| bank#1 gloss | {financial, institution, accept, deposit, channel, money, lend, activity} |
| bank#2 gloss | {slope, land, beside, body, water} |
| bank#3 gloss | {building, business, transact} |

#### Step 5: Count the overlap for each sense

Count the words common to the context and each gloss.

| Sense | Common words | Overlap score |
|---|---|---:|
| bank#1 | deposit, money | **2** |
| bank#2 | none | 0 |
| bank#3 | none | 0 |

#### Step 6: Choose the sense with the maximum overlap

$$
\hat{s} = \underset{s_i}{\arg\max} \; \text{score}(s_i) = \text{bank#1}
$$

**Output:** `bank` = *financial institution*. This is correct.

#### Step 7: Handle ties and zero overlap

- If several senses tie, pick the **most frequent sense** (in WordNet, sense #1).
- If all scores are 0, also fall back to the **most frequent sense**.

---

### Example where the simple version fails

**Sentence:** "We sat on the bank of the river."

```text
Context (after preprocessing) = {sit, river}
```

| Sense | Overlap with {sit, river} |
|---|---:|
| bank#1 | 0 |
| bank#2 | 0 |
| bank#3 | 0 |

The gloss of bank#2 says "body of water", but never uses the word `river`. So all scores are 0 and the algorithm falls back to the most frequent sense (bank#1), which is **wrong**.

This weakness led to the **original Lesk** version below, which also uses the glosses of the context words.

---

### Original Lesk algorithm

The original algorithm does not compare glosses with context **words**. It compares the glosses of the senses of the target word with the **glosses of the senses of the context words**.

$$
\text{score}(s_i) = \sum_{w_j \in \text{context}} \; \max_{s_{jk} \in \text{Senses}(w_j)} \left| \text{gloss}(s_i) \cap \text{gloss}(s_{jk}) \right|
$$

In words: for each context word $w_j$, find the sense $s_{jk}$ of that word whose gloss overlaps most with the gloss of the target sense $s_i$, then add up these best overlaps.

#### Steps with the classic example: "pine cone"

**Step 1: Target words.** Both `pine` and `cone` are ambiguous, so disambiguate them together.

**Step 2: Get the senses and glosses.**

| Sense | Gloss |
|---|---|
| pine#1 | kinds of evergreen tree with needle-shaped leaves |
| pine#2 | waste away through sorrow or illness |
| cone#1 | solid body which narrows to a point |
| cone#2 | something of this shape whether solid or hollow |
| cone#3 | fruit of certain evergreen trees |

**Step 3: Preprocess** (remove stop words, lemmatize `trees` → `tree`).

| Sense | Bag of words |
|---|---|
| pine#1 | {kind, evergreen, tree, needle-shaped, leaf} |
| pine#2 | {waste, away, sorrow, illness} |
| cone#1 | {solid, body, narrow, point} |
| cone#2 | {something, shape, solid, hollow} |
| cone#3 | {fruit, certain, evergreen, tree} |

**Step 4: Compute the overlap for every sense pair.**

| | cone#1 | cone#2 | cone#3 |
|---|---:|---:|---:|
| **pine#1** | 0 | 0 | **2** (evergreen, tree) |
| **pine#2** | 0 | 0 | 0 |

**Step 5: Pick the best combination.** The highest overlap is 2, for pine#1 and cone#3.

**Output:**

- `pine` = *evergreen tree*
- `cone` = *fruit of an evergreen tree*

#### Second example: "bank" and "river"

**Sentence:** "We sat on the bank of the river."

Using the original Lesk idea, we also look up the gloss of `river`:

| Sense | Gloss (preprocessed) |
|---|---|
| river#1 | {large, natural, stream, water} |
| bank#1 | {financial, institution, accept, deposit, channel, money, lend, activity} |
| bank#2 | {slope, land, beside, body, water} |

| Pair | Common words | Score |
|---|---|---:|
| bank#1 and river#1 | none | 0 |
| bank#2 and river#1 | water | **1** |

So `bank` = bank#2 (*sloping land beside water*), which is **correct**. The word `water` links the two glosses even though the sentence never said it.

---

### Extended Lesk (Banerjee and Pedersen, 2002)

The glosses in a dictionary are very short, so overlaps are often 0. **Extended Lesk** makes each signature longer by adding the glosses of **related senses** from WordNet:

- hypernyms (more general: *bank* → *slope*)
- hyponyms (more specific)
- meronyms and holonyms (part and whole)
- similar-to and see-also relations

$$
\text{signature}(s_i) = \text{gloss}(s_i) \;\cup\; \text{gloss}(\text{hypernyms}) \;\cup\; \text{gloss}(\text{hyponyms}) \;\cup\; \ldots
$$

It also rewards **longer matches**. An overlap of $k$ consecutive words scores $k^2$ instead of $k$:

$$
\text{score} = \sum_{\text{overlaps}} k^2
$$

For example, a shared phrase `body of water` (3 words) scores $3^2 = 9$, while three separate shared words score only $1 + 1 + 1 = 3$.

**Example (illustrative):** for "He fished from the bank of the river", the expanded signature of bank#2 includes words from related glosses (such as *land*, *water*, *slope*). The context words `fish` and `river`, expanded with their own glosses (such as *water*, *stream*, *swim*), now share `water` with bank#2 and none with bank#1. So bank#2 wins.

---

### Pseudocode (Simplified Lesk)

```text
function SimplifiedLesk(word, sentence):
    best_sense  = most_frequent_sense(word)
    max_overlap = 0
    context     = preprocess(words in sentence, excluding word)

    for each sense in senses(word):
        signature = preprocess(gloss(sense))
        overlap   = |signature ∩ context|
        if overlap > max_overlap:
            max_overlap = overlap
            best_sense  = sense

    return best_sense
```

### Code example (NLTK)

```python
from nltk.wsd import lesk
from nltk.tokenize import word_tokenize

sentence = "I deposited money in the bank"
sense = lesk(word_tokenize(sentence), "bank", pos="n")

print(sense)               # a WordNet synset
print(sense.definition())  # its gloss
```

(You need `nltk`, plus the `wordnet` and `punkt` data packages.)

---

### Comparison of Lesk variants

| Variant | Compares | Strength | Weakness |
|---|---|---|---|
| Simplified Lesk | Sense glosses with context words | Fast and simple | Fails when the gloss and context share no words |
| Original Lesk | Sense glosses with glosses of context senses | Finds indirect links | Slow, since it checks many sense pairs |
| Extended Lesk | Expanded glosses (related senses) with the context | Fewer zero-overlap cases | Needs WordNet relations and is more complex |

### Advantages

- Needs **no sense-annotated corpus**, so it is cheap to apply
- Simple and easy to implement
- Works for any word that has a dictionary entry
- Easy to explain, since the chosen sense comes with an overlap count

### Limitations

- Very sensitive to the **exact wording** of dictionary glosses
- Glosses are short, so overlap is often 0 or ties are common
- Stop-word handling and lemmatization strongly affect the result
- The original version is computationally expensive for long sentences
- Usually less accurate than supervised methods
- Ignores word order and syntax

### Supervised WSD vs Lesk

| Point | Supervised (Naive Bayes) | Lesk |
|---|---|---|
| Needs labelled corpus | Yes | No |
| Knowledge source | Training data | Dictionary glosses |
| Accuracy | Higher with enough data | Lower |
| Type | Machine learning | Knowledge-based |

### Interview-ready answer

> The Lesk algorithm is a dictionary-based WSD method. For an ambiguous word, it retrieves the gloss of each sense from a dictionary such as WordNet, preprocesses the glosses and the context by removing stop words and lemmatizing, and counts the overlapping words between each gloss and the context. The sense with the highest overlap is chosen, with the most frequent sense as a fallback for ties. The original version compares glosses of the context words' senses, and Extended Lesk adds glosses of related senses and rewards longer overlaps. Its main weakness is its dependence on the exact wording of the glosses.
> 