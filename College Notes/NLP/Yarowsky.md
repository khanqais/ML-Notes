## 4. Yarowsky Bootstrapping Algorithm for WSD

### Definition

The **Yarowsky algorithm** (David Yarowsky, 1995) is a **semi-supervised (bootstrapping)** approach to Word Sense Disambiguation. It starts with only a **small set of seed examples** and a large amount of **unlabelled text**, and it grows its own training data automatically.

**Core idea:** train a classifier on a few labelled examples, use it to label more examples, add the confident ones to the training set, and repeat.

> The goal is accuracy close to supervised WSD, without paying for a large hand-annotated corpus.

### Where it fits

| Approach | Labelled data needed | Example |
|---|---|---|
| Supervised | A large sense-tagged corpus | Naive Bayes |
| Knowledge-based | None (uses a dictionary) | Lesk |
| **Semi-supervised** | **A few seeds + unlabelled text** | **Yarowsky** |

---

### Two key assumptions

The algorithm relies on two observations about language.

**1. One sense per collocation**

A word nearly always keeps the same sense when it appears in the same collocation (a close, fixed neighbouring word or phrase).

- `plant life` → always *living organism*
- `manufacturing plant` → always *factory*

**2. One sense per discourse**

Within one document or discourse, an ambiguous word is very likely to be used in only **one** sense.

- A gardening article that uses `plant` ten times almost always means *living organism* in all ten.

---

### Decision lists

Yarowsky uses a **decision list** as the classifier. It is an ordered list of rules, from the most reliable to the least reliable.

```text
IF feature = "workers"  THEN sense = factory   (score 6.3)
IF feature = "life"     THEN sense = living    (score 5.8)
IF feature = "water"    THEN sense = living    (score 3.2)
...
```

To classify a new example, we go down the list and use the **first rule that matches**. Only the single strongest piece of evidence decides, unlike Naive Bayes, which combines all features.

#### Features used

| Feature | Example for `plant` |
|---|---|
| Word immediately to the left | `manufacturing` in "manufacturing plant" |
| Word immediately to the right | `life` in "plant life" |
| Word at ±2 positions | `the` in "the big plant" |
| Any word within ±k words | `water`, `workers` |
| Pair of words in sequence | `plant` + `species` |

#### Scoring a rule (log-likelihood ratio)

For a feature $f_i$ and two senses $S_1$ and $S_2$, the reliability of the rule is:

$$
\text{score}(f_i) = \left| \log \frac{P(S_1 \mid f_i)}{P(S_2 \mid f_i)} \right|
$$

- A **large** score means the feature points strongly to one sense.
- A score near **0** means the feature does not help.

The rules are sorted by this score in decreasing order. The predicted sense for a new example is:

$$
\hat{S} = \text{sense of the highest-scoring rule whose feature is present}
$$

#### Smoothing

If a feature never appears with one sense, the ratio becomes infinite. We smooth the estimate:

$$
P(S \mid f) = \frac{C(S, f) + \alpha}{C(f) + k\alpha}
$$

Where:

- $C(S, f)$ is the number of examples with feature $f$ labelled as sense $S$
- $C(f)$ is the total number of examples with feature $f$
- $k$ is the number of senses
- $\alpha$ is a small constant (for example 0.1)

---

### Steps of the Yarowsky algorithm

Running example: disambiguate the word **`plant`** with two senses:

- **Sense A:** living organism (a plant in a garden)
- **Sense B:** factory (a manufacturing plant)

#### Step 1: Collect all examples of the target word

Gather every sentence containing `plant` from a large **unlabelled** corpus.

```text
1. "Plant life in the rainforest is diverse."
2. "The manufacturing plant employs 500 workers."
3. "The plant needs water and sunlight."
4. "Workers at the plant assemble car engines."
5. "The plant closed after the strike."
... (thousands more)
```

#### Step 2: Choose seed collocations (the only human input)

Pick one or two reliable seed words for each sense.

| Sense | Seed collocation |
|---|---|
| A (living) | `life` |
| B (factory) | `manufacturing` |

#### Step 3: Label the examples that contain a seed

Tag only the sentences that match a seed. Everything else stays **unlabelled** (the residual set).

| Sentence | Label |
|---|---|
| 1. "Plant **life** in the rainforest is diverse." | A |
| 2. "The **manufacturing** plant employs 500 workers." | B |
| 3. "The plant needs water and sunlight." | unlabelled |
| 4. "Workers at the plant assemble car engines." | unlabelled |
| 5. "The plant closed after the strike." | unlabelled |

Usually only a small fraction (a few percent) of the examples get a label here.

#### Step 4: Train a decision list on the labelled examples

From the seed-labelled sentences, learn every collocation that co-occurs with each sense, and rank them.

Suppose the seed-labelled examples show these counts for the feature `workers` and the feature `water`:

| Feature | Count with A | Count with B |
|---|---:|---:|
| `water` | 9 | 1 |
| `workers` | 0 | 8 |

With $\alpha = 0.1$ and $k = 2$:

$$
P(A \mid \text{workers}) = \frac{0 + 0.1}{8 + 0.2} \approx 0.012, \qquad P(B \mid \text{workers}) = \frac{8 + 0.1}{8 + 0.2} \approx 0.988
$$

$$
\text{score}(\text{workers}) = \log_2 \frac{0.988}{0.012} = \log_2 81 \approx 6.3
$$

For `water`, which has no smoothing problem:

$$
\text{score}(\text{water}) = \log_2 \frac{0.9}{0.1} = \log_2 9 \approx 3.2
$$

The resulting decision list (illustrative):

| Rank | Feature | Predicts | Score |
|---:|---|---|---:|
| 1 | `workers` | B (factory) | 6.3 |
| 2 | `life` | A (living) | 5.8 |
| 3 | `manufacturing` | B (factory) | 5.5 |
| 4 | `water` | A (living) | 3.2 |

#### Step 5: Apply the decision list to ALL examples

Run the learned list on the **entire** corpus, including the unlabelled examples. Keep only the labels whose rule score is **above a confidence threshold** (for example 3.0).

| Sentence | Matching rule | Score | Result |
|---|---|---:|---|
| 3. "The plant needs **water** and sunlight." | `water` → A | 3.2 | Labelled **A** |
| 4. "**Workers** at the plant assemble car engines." | `workers` → B | 6.3 | Labelled **B** |
| 5. "The plant closed after the strike." | no rule matches | none | Stays unlabelled |

The training set has grown from 2 to 4 labelled sentences.

#### Step 6: Use one sense per discourse (optional refinement)

Within each document, look at the labels already assigned to `plant`. If one sense clearly dominates, relabel the remaining or doubtful occurrences in that document with the dominant sense.

**Example:** in a gardening article with 10 occurrences of `plant`, 8 are labelled A, 1 is labelled B, and 1 is unlabelled. We relabel the odd B (a probable error) and the unlabelled one as **A**.

This step both **adds** new labels and **corrects** earlier mistakes.

#### Step 7: Repeat (retrain and relabel)

Go back to Step 4 using the larger labelled set. New collocations are discovered, which label even more examples.

**Example of the chain:** sentence 3 was labelled A by the rule `water`. Retraining on it now shows that `sunlight` often co-occurs with A, so a new rule is added:

```text
IF feature = "sunlight" THEN sense = living
```

In the next round, any sentence with `sunlight` but no seed or `water` is labelled A. Sentence 5 ("The plant closed after the strike.") may get labelled B once `strike` has been seen often enough with B examples.

```text
Iteration 1:  seeds only              → 2% of examples labelled
Iteration 2:  + water, workers, ...   → 15% labelled
Iteration 3:  + sunlight, strike, ... → 60% labelled
...
Iteration n:  nothing new gets labelled → stop
```

#### Step 8: Stop at convergence

Stop when the labelling no longer changes (the residual set stops shrinking), or after a fixed number of iterations.

#### Step 9: Use the final decision list

The final decision list is the trained model. To disambiguate `plant` in a **new** sentence:

1. Extract its features.
2. Go down the decision list.
3. Take the sense of the first (highest-scoring) matching rule.
4. If no rule matches, use the most frequent sense.

**Example:** "The new plant will hire 200 **workers**." → the rule `workers` matches → **B (factory)**.

---

### Summary of the flow

```text
Unlabelled corpus + seeds
        ↓
Label examples that match a seed
        ↓
Train decision list (rank by log-likelihood ratio)
        ↓
Label more examples above the confidence threshold
        ↓
Apply one sense per discourse (clean up)
        ↓
Repeat until convergence
        ↓
Final decision list → classify new text
```

### Pseudocode

```text
function Yarowsky(corpus, seeds):
    labelled = label examples in corpus that match a seed
    repeat:
        rules   = train_decision_list(labelled)            # rank by LLR
        labelled = labelled ∪ { examples labelled by rules above threshold }
        apply one-sense-per-discourse to labelled
    until labelled stops changing
    return rules
```

### Advantages

- Needs only a **few seed words**, not a large hand-labelled corpus
- Uses abundant **unlabelled** text
- Decision lists are **easy to read** (you can see why a sense was chosen)
- Reaches accuracy close to supervised methods on many words
- One-sense-per-discourse helps fix errors automatically

### Limitations

- Results depend heavily on the **choice of seeds**
- **Error propagation:** a wrong label added early can spread in later iterations
- The confidence threshold must be tuned
- The one-sense-per-collocation and one-sense-per-discourse rules are strong assumptions (not always true, especially for related senses)
- A separate model is usually needed for each target word
- Uses only the single best rule, so it ignores other evidence

### Comparison with other WSD methods

| Point | Supervised (Naive Bayes) | Lesk | Yarowsky |
|---|---|---|---|
| Type | Supervised | Knowledge-based | Semi-supervised |
| Labelled data | Large corpus | None | Few seeds |
| Knowledge source | Training data | Dictionary glosses | Unlabelled text + seeds |
| Classifier | Probabilistic | Gloss overlap | Decision list |
| Main risk | Annotation cost | Short glosses | Error propagation |

### Interview-ready answer

> The Yarowsky algorithm is a semi-supervised bootstrapping method for WSD. It starts with a few seed collocations for each sense, uses them to label a small part of an unlabelled corpus, and trains a decision list ranked by log-likelihood ratio. The decision list is then applied to the whole corpus, and the most confident new labels are added to the training set. The one-sense-per-collocation and one-sense-per-discourse assumptions help to extend and clean the labels. The process repeats until no new examples are labelled, and the final decision list is used to disambiguate new text. It needs very little annotation, but it can suffer from error propagation and depends on good seeds.
> 