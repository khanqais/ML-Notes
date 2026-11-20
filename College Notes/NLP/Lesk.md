## 3. Lesk Algorithm for WSD

### Definition

The **Lesk algorithm** (Michael Lesk, 1986) is a **dictionary-based (knowledge-based)** approach to Word Sense Disambiguation. It does **not** need a sense-tagged corpus. Instead, it uses the **definitions (glosses)** given in a dictionary such as WordNet.

**Core idea:** the correct sense of an ambiguous word is the one whose dictionary definition shares the **most words** with the surrounding context.

> Words that appear together in a sentence usually talk about the same topic, so the right sense should have a definition that uses related words.

### Key terms

| Term | Meaning |
|---|---|
| Target word | The ambiguous word to disambiguate (e.g. `bank`) |
| Sense | One possible meaning of the word |
| Gloss | The dictionary definition of a sense |
| Context | The words surrounding the target word (whole sentence or a window) |
| Overlap | Number of words shared between two bags of words |
| Signature | The bag of words that represents a sense (its gloss, possibly expanded) |

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

## Part A: Simplified Lesk (step by step)

### The senses used in all examples

We use the **noun senses of `bank` from WordNet 3.0** (numbered as in NLTK). The glosses are shortened a little (the example sentences are removed).

| Sense | Meaning | Gloss |
|---|---|---|
| bank#1 | river side | sloping land (especially the slope beside a body of water) |
| bank#2 | financial institution | a financial institution that accepts deposits and channels the money into lending activities |
| bank#3 | ridge | a long ridge or pile |
| bank#4 | row of objects | an arrangement of similar objects in a row or in tiers |
| bank#5 | reserve supply | a supply or stock held in reserve for future use (especially in emergencies) |
| bank#6 | gambling funds | the funds held by a gambling house or the dealer in some gambling games |
| bank#7 | banked road | a slope in the turn of a road or track; the outside is higher than the inside in order to reduce the effects of centrifugal force |
| bank#8 | money box | a container (usually with a slot in the top) for keeping money at home |
| bank#9 | bank building | a building in which the business of banking is transacted |
| bank#10 | aircraft tilt | a flight maneuver; aircraft tips laterally about its longitudinal axis (especially in turning) |

You can print these yourself to check:

```python
from nltk.corpus import wordnet as wn

for i, s in enumerate(wn.synsets("bank", pos="n"), start=1):
    print(i, s.definition())
```

### Running example

**Sentence:** "I deposited money in the bank and applied for a loan."

**Target word:** `bank`

---

### Step 1: Identify the target word (and its part of speech)

Pick the ambiguous word and find its POS tag, so that only senses of the right part of speech are used.

```text
Sentence : I deposited money in the bank and applied for a loan.
POS tags : I/PRP deposited/VBD money/NN in/IN the/DT bank/NN and/CC applied/VBD for/IN a/DT loan/NN
Target   : bank  (tagged NN = noun)
```

Because `bank` is a **noun** here, we only consider the 10 noun senses above. The verb senses of `bank` (for example, "to tilt an aircraft" or "to rely on") are ignored.

---

### Step 2: Retrieve all senses and glosses from the dictionary

Look up every noun sense of `bank` in WordNet. This gives the table of 10 senses above (bank#1 to bank#10), each with its gloss.

---

### Step 3: Extract the context

Take the words around the target word. There are two common choices.

**Option 1: the whole sentence**

```text
Context = I, deposited, money, in, the, and, applied, for, a, loan
```

**Option 2: a window of 3 words on each side**

```text
Left 3  = deposited, money, in      (the word "the" directly before bank is also skipped as a stop word)
Right 3 = and, applied, for
Context = deposited, money, in, and, applied, for
```

We use the **whole sentence** in the rest of this example. The target word itself is never part of the context.

---

### Step 4: Preprocess the glosses and the context

Apply the same preprocessing to the context **and** to every gloss.

1. Convert to lowercase.
2. Remove **stop words** (I, in, the, a, for, and, that, of, into, ...).
3. **Lemmatize** (deposited → deposit, deposits → deposit, applied → apply, lending → lend).
4. Remove the target word itself.
5. Convert to a **set** of words (a bag of words).

**Worked example on the gloss of bank#2:**

```text
Original    : a financial institution that accepts deposits and channels the money into lending activities
Lowercase   : a financial institution that accepts deposits and channels the money into lending activities
No stopwords: financial institution accepts deposits channels money lending activities
Lemmatized  : financial institution accept deposit channel money lend activity
```

**Worked example on the context:**

```text
Original    : I deposited money in the bank and applied for a loan
No stopwords: deposited money bank applied loan
Remove target: deposited money applied loan
Lemmatized  : deposit money apply loan
```

So:

$$
\text{context} = \{\text{deposit}, \text{money}, \text{apply}, \text{loan}\}
$$

**All preprocessed glosses:**

| Sense | Preprocessed gloss (bag of words) |
|---|---|
| bank#1 | {slope, land, especially, beside, body, water} |
| bank#2 | {financial, institution, accept, deposit, channel, money, lend, activity} |
| bank#3 | {long, ridge, pile} |
| bank#4 | {arrangement, similar, object, row, tier} |
| bank#5 | {supply, stock, hold, reserve, future, use, especially, emergency} |
| bank#6 | {fund, hold, gambling, house, dealer, game} |
| bank#7 | {slope, turn, road, track, outside, high, inside, order, reduce, effect, centrifugal, force} |
| bank#8 | {container, usually, slot, top, keep, money, home} |
| bank#9 | {building, business, banking, transact} |
| bank#10 | {flight, maneuver, aircraft, tip, laterally, longitudinal, axis, especially, turn} |

---

### Step 5: Count the overlap for each sense

Compare every gloss with the context $\{\text{deposit}, \text{money}, \text{apply}, \text{loan}\}$.

| Sense | Common words with context | Overlap score |
|---|---|---:|
| bank#1 | none | 0 |
| bank#2 | deposit, money | **2** |
| bank#3 | none | 0 |
| bank#4 | none | 0 |
| bank#5 | none | 0 |
| bank#6 | none | 0 |
| bank#7 | none | 0 |
| bank#8 | money | 1 |
| bank#9 | none | 0 |
| bank#10 | none | 0 |

Note that bank#8 (money box) is a close competitor because its gloss also contains `money`. The extra word `deposit` is what pushes bank#2 ahead.

---

### Step 6: Choose the sense with the maximum overlap

$$
\hat{s} = \underset{s_i}{\arg\max} \; \text{score}(s_i) = \text{bank}_2
$$

The highest score is 2, for bank#2.

**Output:** `bank` = *a financial institution that accepts deposits*. This is **correct**.

---

### Step 7: Handle ties and zero overlap

Two special cases need a fallback rule. In both, the fallback is the **most frequent sense**. WordNet lists senses roughly in order of frequency, so this means the **lowest sense number** among the candidates.

#### Case A: a tie between several senses

**Sentence:** "The bank held my money."

Context after preprocessing: $\{\text{hold}, \text{money}\}$

| Sense | Common words | Score |
|---|---|---:|
| bank#1 | none | 0 |
| bank#2 | money | 1 |
| bank#5 | hold | 1 |
| bank#6 | hold | 1 |
| bank#8 | money | 1 |
| others | none | 0 |

Four senses tie with a score of 1: bank#2, bank#5, bank#6, and bank#8. We pick the most frequent among them, which is the lowest number: **bank#2** (financial institution). This is correct here.

#### Case B: zero overlap for all senses

**Sentence:** "She went to the bank to ask for a loan."

Context after preprocessing: $\{\text{go}, \text{ask}, \text{loan}\}$

No gloss contains `go`, `ask`, or `loan`, so every sense scores 0. The fallback gives the most frequent sense overall, **bank#1** (river side). This is **wrong**, because the sentence is about money.

This failure is the main weakness of Simplified Lesk: a gloss and a context can talk about the same topic without sharing a single word. The next two variants try to fix this.

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

sentence = "I deposited money in the bank and applied for a loan"
sense = lesk(word_tokenize(sentence), "bank", pos="n")

print(sense)               # a WordNet synset
print(sense.definition())  # its gloss
```

(You need `nltk`, plus the `wordnet` and `punkt` data packages.)

---

## Part B: Original Lesk (step by step)

The original algorithm does not compare glosses with context **words**. It compares the glosses of the target word's senses with the **glosses of the senses of the context words**. This helps when the context word itself does not appear in any gloss.

$$
\text{score}(s_i) = \sum_{w_j \in \text{context}} \; \max_{s_{jk} \in \text{Senses}(w_j)} \left| \text{gloss}(s_i) \cap \text{gloss}(s_{jk}) \right|
$$

In words: for each context word $w_j$, find the sense $s_{jk}$ of that word whose gloss overlaps most with the gloss of the target sense $s_i$. Then add these best overlaps over all context words.

### Example 1: "bank" and "river"

**Sentence:** "We sat on the bank of the river."

#### Step 1: Identify the target word and the context words

```text
Target word   : bank
Context words : sit, river      (after removing stop words and lemmatizing)
```

#### Step 2: Get the senses and glosses of the target word

The 10 noun senses of `bank` from the table in Part A.

#### Step 3: Get the senses and glosses of every context word

| Word | Sense | Gloss |
|---|---|---|
| river | river#1 | a large natural stream of water (larger than a creek) |
| river | river#2 | a large indefinite quantity |
| sit | sit#1 | be seated |
| sit | sit#2 | take a seat |

(Only a few senses are shown to keep the example readable.)

#### Step 4: Preprocess all glosses

| Gloss | Bag of words |
|---|---|
| river#1 | {large, natural, stream, water, creek} |
| river#2 | {large, indefinite, quantity} |
| sit#1 | {seat} |
| sit#2 | {take, seat} |

The bags for bank#1 to bank#10 are the same as in Part A, Step 4.

#### Step 5: Compute the overlap for every pair

Overlap between each `bank` sense and each sense of `river`:

| Bank sense | with river#1 | with river#2 | Best for river |
|---|---|---|---:|
| bank#1 {slope, land, especially, beside, body, **water**} | water (1) | none (0) | **1** |
| bank#2 {financial, institution, accept, deposit, channel, money, lend, activity} | 0 | 0 | 0 |
| bank#3 to bank#10 | 0 | 0 | 0 |

Overlap between each `bank` sense and each sense of `sit`:

| Bank sense | with sit#1 | with sit#2 | Best for sit |
|---|---|---|---:|
| bank#1 | 0 | 0 | 0 |
| bank#2 to bank#10 | 0 | 0 | 0 |

#### Step 6: Add the best overlaps for each target sense

$$
\text{score}(\text{bank}_1) = \underbrace{1}_{\text{river}} + \underbrace{0}_{\text{sit}} = 1
$$

$$
\text{score}(\text{bank}_2) = 0 + 0 = 0
$$

All other senses also score 0.

#### Step 7: Choose the sense with the maximum score

$$
\hat{s} = \underset{s_i}{\arg\max} \; \text{score}(s_i) = \text{bank}_1
$$

**Output:** `bank` = *sloping land beside water*. This is **correct**. The word `water` in the gloss of `river` linked it to bank#1, even though the sentence itself never said "water".

---

### Example 2: "pine cone" (the classic example from Lesk's paper)

Here **both** words are ambiguous, so they are disambiguated together.

#### Step 1: Identify the words

```text
Phrase : pine cone
Words  : pine, cone
```

#### Step 2: Get the senses and glosses

| Sense | Gloss |
|---|---|
| pine#1 | kinds of evergreen tree with needle-shaped leaves |
| pine#2 | waste away through sorrow or illness |
| cone#1 | solid body which narrows to a point |
| cone#2 | something of this shape whether solid or hollow |
| cone#3 | fruit of certain evergreen trees |

#### Step 3: Preprocess

| Sense | Bag of words |
|---|---|
| pine#1 | {kind, evergreen, tree, needle-shaped, leaf} |
| pine#2 | {waste, away, sorrow, illness} |
| cone#1 | {solid, body, narrow, point} |
| cone#2 | {something, shape, solid, hollow} |
| cone#3 | {fruit, certain, evergreen, tree} |

(Note that `trees` is lemmatized to `tree`.)

#### Step 4: Compute the overlap for every pair of senses

| | cone#1 | cone#2 | cone#3 |
|---|---:|---:|---:|
| **pine#1** | 0 | 0 | **2** (evergreen, tree) |
| **pine#2** | 0 | 0 | 0 |

(cone#1 and cone#2 share the word `solid`, but neither is a pine sense, so that pair is not scored.)

#### Step 5: Pick the best combination

The highest overlap is 2, for the pair (pine#1, cone#3).

**Output:**

- `pine` = *evergreen tree*
- `cone` = *fruit of an evergreen tree*

---

## Part C: Extended Lesk (Banerjee and Pedersen, 2002)

Glosses in a dictionary are very short, so overlaps are often 0 (as in Case B of Part A). **Extended Lesk** makes every signature longer by adding the glosses of **related senses** from WordNet:

- hypernyms (more general: *bank* → *financial institution*)
- hyponyms (more specific)
- meronyms and holonyms (part and whole)
- similar-to and see-also relations

$$
\text{signature}(s_i) = \text{gloss}(s_i) \;\cup\; \text{gloss}(\text{hypernyms}) \;\cup\; \text{gloss}(\text{hyponyms}) \;\cup\; \ldots
$$

It also rewards **longer matches**: an overlap of $k$ consecutive words scores $k^2$ instead of $k$.

$$
\text{score} = \sum_{\text{overlaps}} k^2
$$

For example, a shared phrase of 3 words (such as `body of water`) scores $3^2 = 9$, while three separate shared words score only $1 + 1 + 1 = 3$.

### Worked example (illustrative glosses)

**Sentence:** "She went to the bank to ask for a loan." (the sentence that failed in Part A, Case B)

The glosses below are **simplified for teaching** and are not the exact WordNet text.

#### Step 1: Context words

```text
Context words (after preprocessing) = go, ask, loan
```

#### Step 2: Build the expanded signature of each target sense

| Sense | Own gloss | Related gloss added | Expanded signature |
|---|---|---|---|
| bank#2 | accepts deposits and channels the money into lending activities | hypernym *financial institution*: an institution that deals with money, credit and investment | {accept, deposit, channel, money, lend, activity, institution, deal, credit, investment} |
| bank#1 | sloping land beside a body of water | hypernym *slope*: an elevated geological formation | {slope, land, beside, body, water, elevated, geological, formation} |

#### Step 3: Build the expanded signature of each context word

| Word | Gloss | Expanded signature |
|---|---|---|
| loan | a sum of money lent at interest | {sum, money, lend, interest} |
| ask | make a request | {make, request} |
| go | move from one place to another | {move, place} |

#### Step 4: Compute the overlap

| Sense | Shared words with the context signatures | Score |
|---|---|---:|
| bank#2 | money, lend | **2** |
| bank#1 | none | 0 |

#### Step 5: Choose the maximum

$$
\hat{s} = \text{bank}_2 \quad (\text{financial institution})
$$

The expanded signatures fixed the failure of Simplified Lesk. The gloss of `loan` shares `money` and `lend` with the gloss of bank#2, even though `loan` itself is not in that gloss.

---

## Comparison of Lesk variants

| Variant | Compares | Strength | Weakness |
|---|---|---|---|
| Simplified Lesk | Sense glosses with context words | Fast and simple | Fails when the gloss and context share no word |
| Original Lesk | Sense glosses with glosses of context-word senses | Finds indirect links | Slow, since it checks many sense pairs |
| Extended Lesk | Expanded glosses (related senses) with the context | Fewer zero-overlap cases | Needs WordNet relations and is more complex |

### Advantages

- Needs **no sense-annotated corpus**, so it is cheap to apply
- Simple and easy to implement
- Works for any word that has a dictionary entry
- Easy to explain, since the chosen sense comes with an overlap count

### Limitations

- Very sensitive to the **exact wording** of dictionary glosses
- Glosses are short, so overlap is often 0 and ties are common
- Stop-word handling and lemmatization strongly affect the result
- The original version is computationally expensive for long sentences
- Spurious matches on common words can pick a wrong sense
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

> The Lesk algorithm is a dictionary-based WSD method. For an ambiguous word, it retrieves the gloss of each sense from a dictionary such as WordNet, preprocesses the glosses and the context by removing stop words and lemmatizing, and counts the overlapping words between each gloss and the context. The sense with the highest overlap is chosen, with the most frequent sense as a fallback for ties or zero overlap. The original version compares glosses of the context words' senses, and Extended Lesk adds glosses of related senses and rewards longer overlaps. Its main weakness is its dependence on the exact wording of the glosses.
> 