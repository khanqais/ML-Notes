# 1-Gram and N-Gram Models

An **n-gram language model** is a statistical model that predicts the next word using only the previous $n-1$ words. A **1-gram model** (also called a **unigram model**) is the simplest case: it predicts each word independently, based only on how frequently that word appeared in the training corpus.

---

## What is an n-gram?

An **n-gram** is a contiguous sequence of $n$ tokens, usually words.

| n | Name | Example |
|---:|---|---|
| 1 | Unigram / 1-gram | `I` |
| 2 | Bigram / 2-gram | `I love` |
| 3 | Trigram / 3-gram | `I love NLP` |
| 4 | 4-gram | `I love NLP models` |

For the sentence:

```text
I love NLP
```

The unigrams are:

```text
I
love
NLP
```

The bigrams are:

```text
I love
love NLP
```

The trigram is:

```text
I love NLP
```

An n-gram model estimates the probability of a word using a fixed window of preceding words. For an $n$-gram model, that window contains $n-1$ previous words.

---

## Language model idea

A language model assigns a probability to a whole sentence:

$$
P(w_1, w_2, \ldots, w_N)
$$

For example, it should assign a higher probability to:

```text
I love machine learning
```

than to:

```text
learning machine love I
```

Using the **chain rule of probability**, the exact probability is:

$$
P(w_1, w_2, \ldots, w_N) = P(w_1) \, P(w_2 \mid w_1) \, P(w_3 \mid w_1, w_2) \cdots P(w_N \mid w_1, \ldots, w_{N-1})
$$

Here $P(w_2 \mid w_1)$ is read as "the probability of $w_2$ given $w_1$".

This is impractical because each word would depend on **all** previous words, and most long histories never appear in any training data. N-gram models solve this with the **Markov assumption**: only the last $n-1$ words matter.

---

## 1-gram (unigram) model

A unigram model assumes every word is independent of every other word:

$$
P(w_1, w_2, \ldots, w_N) \approx P(w_1) \, P(w_2) \cdots P(w_N)
$$

The probability of a single word is estimated from its frequency in the corpus:

$$
P(w_i) = \frac{C(w_i)}{N}
$$

Where:

- $C(w_i)$ is the number of times the word $w_i$ appears in the corpus.
- $N$ is the total number of word tokens in the corpus.

### Example

Suppose the corpus is:

```text
I love NLP. I love AI. I study AI.
```

Ignoring punctuation, it has 9 tokens: `I love NLP I love AI I study AI`.

| Word | Count |
|---|---:|
| `I` | 3 |
| `love` | 2 |
| `NLP` | 1 |
| `AI` | 2 |
| `study` | 1 |

Total tokens:

$$
N = 9
$$

So:

$$
P(\text{I}) = \frac{3}{9} \approx 0.333
$$

$$
P(\text{love}) = \frac{2}{9} \approx 0.222
$$

$$
P(\text{NLP}) = \frac{1}{9} \approx 0.111
$$

For the sentence `I love NLP`:

$$
P(\text{I love NLP}) = P(\text{I}) \cdot P(\text{love}) \cdot P(\text{NLP})
$$

$$
= \frac{3}{9} \times \frac{2}{9} \times \frac{1}{9} = \frac{6}{729} \approx 0.0082
$$

### Limitation

A unigram model does **not** understand word order. These two sentences get exactly the same probability because they contain the same words:

```text
I love NLP
NLP love I
```

---

## 2-gram (bigram) model

A bigram model predicts the next word using only the **immediately previous word**:

$$
P(w_i \mid w_1, \ldots, w_{i-1}) \approx P(w_i \mid w_{i-1})
$$

The bigram probability is estimated from counts:

$$
P(w_i \mid w_{i-1}) = \frac{C(w_{i-1}, w_i)}{C(w_{i-1})}
$$

Where:

- $C(w_{i-1}, w_i)$ is the count of the two-word sequence.
- $C(w_{i-1})$ is the count of the previous word.

For example:

$$
P(\text{love} \mid \text{I}) = \frac{C(\text{I love})}{C(\text{I})}
$$

In the corpus above, `I love` appears 2 times and `I` appears 3 times:

$$
P(\text{love} \mid \text{I}) = \frac{2}{3} \approx 0.67
$$

A bigram model captures limited local word order (like `I love`), but it cannot use longer context such as `I love machine`.

---

## 3-gram (trigram) model

A trigram model predicts the next word using the **previous two words**:

$$
P(w_i \mid w_{i-2}, w_{i-1}) = \frac{C(w_{i-2}, w_{i-1}, w_i)}{C(w_{i-2}, w_{i-1})}
$$

Example:

$$
P(\text{NLP} \mid \text{I love}) = \frac{C(\text{I love NLP})}{C(\text{I love})}
$$

If `I love NLP` occurs once and `I love` occurs twice:

$$
P(\text{NLP} \mid \text{I love}) = \frac{1}{2} = 0.5
$$

Trigram models usually give more natural predictions than bigram models because they use more context, but they need much more training data.

---

## General n-gram formula

For an $n$-gram model, the probability of the next word is:

$$
P(w_i \mid w_{i-n+1}, \ldots, w_{i-1}) = \frac{C(w_{i-n+1}, \ldots, w_i)}{C(w_{i-n+1}, \ldots, w_{i-1})}
$$

And the probability of the whole sentence is:

$$
P(w_1, w_2, \ldots, w_N) \approx \prod_{i=1}^{N} P(w_i \mid w_{i-n+1}, \ldots, w_{i-1})
$$

For the first few words, special start tokens such as `<s>` are added so the model has enough previous context. An end token `</s>` is usually added too, so the model knows where a sentence stops.

### Trade-off when choosing n

- **Small $n$:** less context, but counts are reliable.
- **Large $n$:** more context, but most n-grams are rare or never seen (the *sparsity* problem), so more data is needed.

---

## Zero probability problem

If a bigram such as `I mango` never appears in the training data:

$$
C(\text{I mango}) = 0
$$

Then:

$$
P(\text{mango} \mid \text{I}) = 0
$$

This makes the probability of the **whole sentence** zero, even if the sentence is perfectly valid, because the sentence probability is a product of terms.

To avoid this we use **smoothing**. The simplest is **Laplace (add-one) smoothing**:

$$
P_{\text{Laplace}}(w_i \mid w_{i-1}) = \frac{C(w_{i-1}, w_i) + 1}{C(w_{i-1}) + V}
$$

Where $V$ is the vocabulary size (the number of distinct words). Every n-gram gets a count of at least 1, so unseen n-grams receive a small nonzero probability. The $+V$ in the denominator keeps all probabilities summing to 1.
