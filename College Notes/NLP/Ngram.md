Explain 1 gram by gram and n gram Model with formula, and in detail what it is.

An **n-gram model** is a statistical language model that predicts the next word using only the previous $$n-1$$ words. A **1-gram (unigram) model** is the simplest case: it predicts each word independently, using only how frequently that word appeared in the training corpus.[1][2]

## What is an n-gram?

An **n-gram** is a contiguous sequence of $$n$$ tokens (usually words).

| $$n$$ | Name | Example |
|---:|---|---|
| 1 | Unigram / 1-gram | `"I"` |
| 2 | Bigram / 2-gram | `"I love"` |
| 3 | Trigram / 3-gram | `"I love NLP"` |
| 4 | 4-gram | `"I love NLP models"` |

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

An n-gram model estimates the probability of a word based on a fixed window of preceding words; for an $$n$$-gram model, that window contains $$n-1$$ previous words.[1][2][3]

## Language model idea

A language model assigns a probability to a sentence:

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

Using the chain rule of probability, the exact probability is:

$$
P(w_1,w_2,\ldots,w_N)
=
P(w_1)
P(w_2 \mid w_1)
P(w_3 \mid w_1,w_2)
\cdots
P(w_N \mid w_1,w_2,\ldots,w_{N-1})
$$

This is expensive because a word would depend on all previous words. N-gram models approximate it by using only the last $$n-1$$ words.[4][5]

## 1-gram model

A **1-gram model** is also called a **unigram model**.

It assumes every word is independent of every other word:

$$
P(w_1,w_2,\ldots,w_N)
\approx
P(w_1)P(w_2)\cdots P(w_N)
$$

The probability of a single word is estimated from its frequency in the corpus:

$$
P(w_i)
=
\frac{C(w_i)}{N}
$$

Where:

- $$C(w_i)$$ = number of times word $$w_i$$ appears in the corpus.
- $$N$$ = total number of word tokens in the corpus.

### Example

Suppose a corpus has 100 words:

```text
I love NLP. I love AI. I study AI.
```

Word counts:

| Word | Count |
|---|---:|
| `I` | 3 |
| `love` | 2 |
| `NLP` | 1 |
| `AI` | 2 |
| `study` | 1 |

Total tokens: $$N = 10$$ if we count only these words.

So:

$$
P(I)=\frac{3}{10}=0.3
$$

$$
P(love)=\frac{2}{10}=0.2
$$

$$
P(NLP)=\frac{1}{10}=0.1
$$

For the sentence `I love NLP`, the unigram probability is:

$$
P(I,love,NLP)
=
P(I)P(love)P(NLP)
$$

$$
=
0.3 \times 0.2 \times 0.1
=
0.006
$$

A unigram model does **not** understand word order. It treats these two sentences as equally likely if they contain the same words:

```text
I love NLP
NLP love I
```

That is the main limitation of a 1-gram model.[6][7]

## 2-gram model

A **2-gram model** is called a **bigram model**.

It predicts the next word using only the immediately previous word:

$$
P(w_i \mid w_1,w_2,\ldots,w_{i-1})
\approx
P(w_i \mid w_{i-1})
$$

The bigram probability formula is:

$$
P(w_i \mid w_{i-1})
=
\frac{C(w_{i-1},w_i)}{C(w_{i-1})}
$$

Where:

- $$C(w_{i-1},w_i)$$ = count of the two-word sequence.
- $$C(w_{i-1})$$ = count of the previous word.

For example:

$$
P(\text{love} \mid \text{I})
=
\frac{C(\text{I love})}{C(\text{I})}
$$

If `I love` appears 2 times and `I` appears 3 times:

$$
P(\text{love} \mid \text{I})
=
\frac{2}{3}
\approx 0.67
$$

A bigram model understands limited local word order, such as `I love`, but it cannot use longer context like `I love machine`.[3][8]

## 3-gram model

A **3-gram model** is called a **trigram model**.

It predicts the next word using the previous two words:

$$
P(w_i \mid w_{i-2},w_{i-1})
$$

The formula is:

$$
P(w_i \mid w_{i-2},w_{i-1})
=
\frac{C(w_{i-2},w_{i-1},w_i)}{C(w_{i-2},w_{i-1})}
$$

Example:

$$
P(\text{NLP} \mid \text{I love})
=
\frac{C(\text{I love NLP})}{C(\text{I love})}
$$

If `I love NLP` occurs once and `I love` occurs twice:

$$
P(\text{NLP} \mid \text{I love})
=
\frac{1}{2}
=
0.5
$$

Trigram models usually produce more natural predictions than bigram models because they use more context, but they need much more training data.[9][10]

## General n-gram formula

For an $$n$$-gram model, the probability of the next word is:

$$
P(w_i \mid w_{i-n+1},\ldots,w_{i-1})
=
\frac{C(w_{i-n+1},\ldots,w_i)}{C(w_{i-n+1},\ldots,w_{i-1})}
$$

And the sentence probability is:

$$
P(w_1,w_2,\ldots,w_N)
\approx
\prod_{i=1}^{N}
P(w_i \mid w_{i-n+1},\ldots,w_{i-1})
$$

For the first few words, special start tokens such as `<s>` are commonly added so that the model has enough previous context.[2][9]

## Comparison

| Model | Context used | Formula | Example |
|---|---|---|---|
| Unigram | None | $$P(w_i)$$ | $$P(\text{love})$$ |
| Bigram | 1 previous word | $$P(w_i \mid w_{i-1})$$ | $$P(\text{love} \mid \text{I})$$ |
| Trigram | 2 previous words | $$P(w_i \mid w_{i-2},w_{i-1})$$ | $$P(\text{NLP} \mid \text{I love})$$ |
| N-gram | $$n-1$$ previous words | $$P(w_i \mid w_{i-n+1},\ldots,w_{i-1})$$ | $$P(\text{models} \mid \text{I love NLP})$$ for $$n=4$$ |

## Zero probability problem

If a bigram such as `I mango` never appears in training data:

$$
C(\text{I mango}) = 0
$$

Then:

$$
P(\text{mango} \mid \text{I}) = 0
$$

This makes the whole sentence probability zero, even if the sentence is valid. To avoid this, we use **smoothing**, such as Laplace smoothing:

$$
P_{\text{Laplace}}(w_i \mid w_{i-1})
=
\frac{C(w_{i-1},w_i)+1}{C(w_{i-1})+V}
$$

Where $$V$$ is the vocabulary size. This gives unseen n-grams a small nonzero probability.[11]

## Interview-ready answer

> An n-gram language model predicts the next word using the previous $$n-1$$ words. A 1-gram or unigram model assumes words are independent and estimates $$P(w_i)=C(w_i)/N$$. A bigram model uses one previous word: $$P(w_i \mid w_{i-1})=C(w_{i-1},w_i)/C(w_{i-1})$$. A trigram model uses two previous words. Larger $$n$$ gives better context but needs more data and suffers more from sparse counts, so smoothing techniques are used.[1][3][11]

Citations:
[1] N-gram Language Models https://web.cs.hacettepe.edu.tr/~ilyas/Courses/CMP711/lec03-LanguageModels.pdf
[2] 8 n-Gram Language Models https://princeton-introml.github.io/files/ch8.pdf
[3] N-gram Language Models https://web.stanford.edu/~jurafsky/slp3/3.pdf
[4] Introduction to N-gram Models https://cs-114.org/wp-content/uploads/2016/02/CS114_L5_NgramModels.pdf
[5] [PDF] N-gram Language Models - Stanford University https://web.stanford.edu/~jurafsky/slp3/old_aug25/3.pdf
[6] Word n-gram language model https://en.wikipedia.org/wiki/Word_n-gram_language_model
[7] Unigram, Bigram, Trigram Models in NLP | PDF | Applied Mathematics https://www.scribd.com/document/975739735/UNIT-2-Language-Models
[8] [PDF] N-gram Language Models - Stanford University https://web.stanford.edu/~jurafsky/slp3/old_sep21/3.pdf
[9] N-Gram https://www.natalieparde.com/teaching/cs_421_fall2025/N-Gram_Language_Models_and_HMMs.pdf
[10] N-gram language models https://liu-nlp.ai/dl4nlp/modules/module1/n-gram-language-models.pdf
[11] N-gram in NLP https://www.geeksforgeeks.org/nlp/n-gram-in-nlp/
[12] N-gram Language Modeling https://web.stanford.edu/~jurafsky/slp3/slides/lm_jan25.pdf
[13] N-gram Language Models https://web.stanford.edu/~jurafsky/slp3/old_oct19/3.pdf
[14] N-gram Introduction to N-gram Language Language Models Modeling https://web.stanford.edu/class/cs124/lec/lm_jan25.pdf
[15] CHAPTER Language Modeling with N- grams - web.stanford.edu https://web.stanford.edu/~jurafsky/slp3/old/4.pdf
Give me Obsidian notes of this code and remove all these links and all.References.Interview-ready answer.Comparison Make sure the formula should display correctly in the Obsidian node.

# 1-Gram and N-Gram Models

An **n-gram language model** is a statistical model that predicts the next word using only the previous $$n-1$$ words. A **1-gram model**, also called a **unigram model**, is the simplest case: it predicts each word independently based on how frequently it appeared in the training corpus.[1][2]

## What is an n-gram

An **n-gram** is a contiguous sequence of $$n$$ tokens, usually words.

| $$n$$ | Name | Example |
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

An n-gram model estimates the probability of a word using a fixed window of preceding words. For an $$n$$-gram model, that window contains $$n-1$$ previous words.[1][3]

## Language model idea

A language model assigns a probability to a sentence:

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

Using the chain rule of probability, the exact probability is:

$$
P(w_1,w_2,\ldots,w_N)
=
P(w_1)
P(w_2 \mid w_1)
P(w_3 \mid w_1,w_2)
\cdots
P(w_N \mid w_1,w_2,\ldots,w_{N-1})
$$

This is expensive because a word would depend on all previous words. N-gram models approximate it by using only the last $$n-1$$ words.[1][3]

## 1-gram model

A **1-gram model** is also called a **unigram model**.

It assumes every word is independent of every other word:

$$
P(w_1,w_2,\ldots,w_N)
\approx
P(w_1)P(w_2)\cdots P(w_N)
$$

The probability of a single word is estimated from its frequency in the corpus:

$$
P(w_i)
=
\frac{C(w_i)}{N}
$$

Where:

- $$C(w_i)$$ is the number of times word $$w_i$$ appears in the corpus.
- $$N$$ is the total number of word tokens in the corpus.

### Example

Suppose a corpus has 10 words:

```text
I love NLP. I love AI. I study AI.
```

Word counts:

| Word | Count |
|---|---:|
| `I` | 3 |
| `love` | 2 |
| `NLP` | 1 |
| `AI` | 2 |
| `study` | 1 |

Total tokens:

$$
N = 10
$$

So:

$$
P(I)=\frac{3}{10}=0.3
$$

$$
P(love)=\frac{2}{10}=0.2
$$

$$
P(NLP)=\frac{1}{10}=0.1
$$

For the sentence `I love NLP`, the unigram probability is:

$$
P(I,love,NLP)
=
P(I)P(love)P(NLP)
$$

$$
=
0.3 \times 0.2 \times 0.1
=
0.006
$$

A unigram model does **not** understand word order. It treats these two sentences as equally likely if they contain the same words:

```text
I love NLP
NLP love I
```

That is the main limitation of a 1-gram model.[1][2]

## 2-gram model

A **2-gram model** is called a **bigram model**.

It predicts the next word using only the immediately previous word:

$$
P(w_i \mid w_1,w_2,\ldots,w_{i-1})
\approx
P(w_i \mid w_{i-1})
$$

The bigram probability formula is:

$$
P(w_i \mid w_{i-1})
=
\frac{C(w_{i-1},w_i)}{C(w_{i-1})}
$$

Where:

- $$C(w_{i-1},w_i)$$ is the count of the two-word sequence.
- $$C(w_{i-1})$$ is the count of the previous word.

For example:

$$
P(\text{love} \mid \text{I})
=
\frac{C(\text{I love})}{C(\text{I})}
$$

If `I love` appears 2 times and `I` appears 3 times:

$$
P(\text{love} \mid \text{I})
=
\frac{2}{3}
\approx 0.67
$$

A bigram model understands limited local word order, such as `I love`, but it cannot use longer context like `I love machine`.[2][3]

## 3-gram model

A **3-gram model** is called a **trigram model**.

It predicts the next word using the previous two words:

$$
P(w_i \mid w_{i-2},w_{i-1})
$$

The formula is:

$$
P(w_i \mid w_{i-2},w_{i-1})
=
\frac{C(w_{i-2},w_{i-1},w_i)}{C(w_{i-2},w_{i-1})}
$$

Example:

$$
P(\text{NLP} \mid \text{I love})
=
\frac{C(\text{I love NLP})}{C(\text{I love})}
$$

If `I love NLP` occurs once and `I love` occurs twice:

$$
P(\text{NLP} \mid \text{I love})
=
\frac{1}{2}
=
0.5
$$

Trigram models usually produce more natural predictions than bigram models because they use more context, but they need much more training data.[1][3]

## General n-gram formula

For an $$n$$-gram model, the probability of the next word is:

$$
P(w_i \mid w_{i-n+1},\ldots,w_{i-1})
=
\frac{C(w_{i-n+1},\ldots,w_i)}{C(w_{i-n+1},\ldots,w_{i-1})}
$$

And the sentence probability is:

$$
P(w_1,w_2,\ldots,w_N)
\approx
\prod_{i=1}^{N}
P(w_i \mid w_{i-n+1},\ldots,w_{i-1})
$$

For the first few words, special start tokens such as `<s>` are commonly added so that the model has enough previous context.[1][2]

## Zero probability problem

If a bigram such as `I mango` never appears in training data:

$$
C(\text{I mango}) = 0
$$

Then:

$$
P(\text{mango} \mid \text{I}) = 0
$$

This makes the whole sentence probability zero, even if the sentence is valid. To avoid this, we use **smoothing**, such as Laplace smoothing:

$$
P_{\text{Laplace}}(w_i \mid w_{i-1})
=
\frac{C(w_{i-1},w_i)+1}{C(w_{i-1})+V}
$$

Where $$V$$ is the vocabulary size. This gives unseen n-grams a small nonzero probability.[1][2]

