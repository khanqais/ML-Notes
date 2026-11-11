---
tags:
  - NLP
  - semester-notes
  - exam-prep
---

# NLP: Morphology, FSA, Preprocessing and WSD

> [!abstract] Topics covered
> - Morphological analysis outputs
> - Role of FSA in morphological analysis
> - FSA for nouns and verbs
> - FSA for English numbers 1–99
> - Tokenization, stemming and preprocessing
> - Word Sense Disambiguation and Lesk algorithm

---

## 1. Morphological Analysis

### Definition

Morphological analysis breaks a word into its root/stem and affixes, and identifies grammatical information such as tense, number, and part of speech.

### Output format

```text
word → root + category + grammatical features
```

### Regular verb

```text
Input: played
Output: play + V + Past
Root: play
Category: Verb
Tense: Past
```

### Irregular verb

```text
Input: went
Output: go + V + Past
Root: go
Category: Verb
Tense: Past
```

> [!note] Irregular verb
> `went` does not contain the regular suffix `-ed`, so it must be mapped to its root `go` using a lexicon.

### Singular noun

```text
Input: book
Output: book + N + Sg
Root: book
Category: Noun
Number: Singular
```

### Plural noun

```text
Input: books
Output: book + N + Pl
Root: book
Category: Noun
Number: Plural
```

### Summary table

| Type | Input | Output |
|---|---|---|
| Regular verb | played | `play + V + Past` |
| Irregular verb | went | `go + V + Past` |
| Singular noun | book | `book + N + Sg` |
| Plural noun | books | `book + N + Pl` |

---

## 2. Role of FSA in Morphological Analysis

### What is an FSA?

A **Finite State Automaton (FSA)** is a mathematical model used to recognize patterns in input strings. It consists of:

- A finite set of states
- A start state
- Input transitions
- One or more accepting/final states

An FSA reads a word character by character and accepts it if it reaches a final state after consuming the complete input.

### Role in morphological analysis

FSA is used to:

- Recognize valid word forms
- Store roots and affixes in a compact structure
- Check whether morphemes occur in a valid order
- Handle inflectional forms such as plural nouns and verb tenses
- Support morphological generation and parsing

### Example: plural noun

```text
cats → cat + N + Pl
```

```text
q0 --cat--> q1 --s--> q2 (final)
```

Here:

- `q0` = start state
- `q1` = noun stem state
- `q2` = plural noun accepting state

### FSA vs FST

| FSA | FST |
|---|---|
| Recognizes whether a word is valid | Maps surface form to lexical form |
| Output: accepted or rejected | Output: morphological analysis |
| Example: `cats` is accepted | Example: `cats → cat + N + Pl` |
| Cannot generate analysis | Can perform analysis and generation |

### Interview answer

> In morphological analysis, an FSA acts as a recognizer for valid word forms. It represents stems, affixes, and their legal combinations as states and transitions. For example, it can accept `cats` by following the path `cat` + `s`. To produce the full analysis `cat + N + Pl`, we use a Finite State Transducer (FST), which maps surface forms to lexical forms and can also generate words in the reverse direction.

---

## 3. FSA for Nouns

### Purpose

An FSA for nouns recognizes singular and plural noun forms.

### Regular nouns

```text
cat   → cat + N + Sg
cats  → cat + N + Pl
bus   → bus + N + Sg
buses → bus + N + Pl
baby  → baby + N + Sg
babies→ baby + N + Pl
```

### Transition diagram

```text
q0 --[noun stem]--> q1 (final: singular noun)
q1 --[s/es/ies]--> q2 (final: plural noun)
```

### Irregular nouns

Irregular plurals are stored directly in the lexicon:

```text
man → men
mouse → mice
child → children
ox → oxen
```

### Example

```text
cats → cat + N + Pl
mice → mouse + N + Pl
```

---

## 4. FSA for Verbs

### Purpose

An FSA for verbs recognizes:

- Base form
- Third-person singular form
- Past tense
- Present participle

### Regular verbs

```text
walk    → walk + V + Base
walks   → walk + V + 3Sg
walked  → walk + V + Past
walking → walk + V + PresPart

play    → play + V + Base
plays   → play + V + 3Sg
played  → play + V + Past
playing → play + V + PresPart
```

### Transition diagram

```text
q0 --[verb stem]--> q1 (final: base form)
q1 --[s]--> q2 (final: third-person singular)
q1 --[ed]--> q3 (final: past tense)
q1 --[ing]--> q4 (final: present participle)
```

### Irregular verbs

Irregular verbs require separate lexical entries:

```text
go → went
eat → ate
catch → caught
run → ran
```

### Example

```text
went → go + V + Past
ate  → eat + V + Past
```

---

## 5. FSA for English Numbers 1–99

### Number groups

| Group | Words |
|---|---|
| Units: 1–9 | one, two, three, four, five, six, seven, eight, nine |
| Teens: 10–19 | ten, eleven, twelve, thirteen, fourteen, fifteen, sixteen, seventeen, eighteen, nineteen |
| Tens: 20–90 | twenty, thirty, forty, fifty, sixty, seventy, eighty, ninety |
| Compound numbers | twenty-one, thirty-five, ninety-nine |

### States

| State | Meaning |
|---|---|
| S | Start state |
| U | Unit number state |
| T | Teen number state |
| D | Tens state |
| H | Hyphen state |
| F | Final/accepting state |

### Transitions

```text
S --[one ... nine]--> U --> F
S --[ten ... nineteen]--> T --> F
S --[twenty ... ninety]--> D --> F
D --[-]--> H --[one ... nine]--> U --> F
```

### Diagram

```text
                 ┌──────────────────────────────┐
                 │              S               │
                 └───┬──────────┬───────────┬───┘
                     │          │           │
          one...nine │ ten...19 │ twenty... │
                     ▼          ▼           ▼
                  [ U ]      [ T ]       [ D ]
                     │          │           │
                     └──────┬───┴───────────┤
                            ▼               │
                          [ F ]             │
                                            │ hyphen "-"
                                            ▼
                                          [ H ]
                                            │ one...nine
                                            ▼
                                          [ U ]
                                            │
                                            ▼
                                          [ F ]
```

### Accepted words

```text
seven       → S → U → F
fifteen     → S → T → F
thirty      → S → D → F
forty-two   → S → D → H → U → F
ninety-nine → S → D → H → U → F
```

### Rejected words

```text
ten-one    → invalid because ten is a teen number
twenty-    → invalid because hyphen is not followed by a unit
ninety-ten → invalid because ten is not a unit word
zero       → outside the range 1–99
```

---

## 6. Tokenization

### Definition

**Tokenization** is the process of breaking raw text into smaller units called **tokens**. Tokens can be words, sentences, punctuation marks, subwords, or characters.

### Example

```text
Raw text:
"I love Natural Language Processing!"

Word tokens:
["I", "love", "Natural", "Language", "Processing", "!"]
```

### Types of tokenization

| Type | Splits into | Example |
|---|---|---|
| Sentence tokenization | Sentences | `"I love NLP. It is useful."` → `["I love NLP.", "It is useful."]` |
| Word tokenization | Words and punctuation | `"I love NLP!"` → `["I", "love", "NLP", "!"]` |
| Subword tokenization | Word pieces | `"unhappiness"` → `["un", "happi", "ness"]` |
| Character tokenization | Characters | `"cat"` → `["c", "a", "t"]` |

### Why tokenization is needed

- Converts raw text into processable units
- Helps in word counting and indexing
- Required before POS tagging, NER, and sentiment analysis
- Used in search engines and language models

---

## 7. Stemming

### Definition

**Stemming** is a rule-based process that reduces a word to its stem by removing suffixes. The resulting stem may or may not be a valid dictionary word.

### Examples

```text
jumps   → jump
jumping → jump
jumped  → jump
studies → studi
caring  → car
```

### Common algorithms

- Porter Stemmer
- Snowball Stemmer
- Lancaster Stemmer

### Advantages

- Fast
- Reduces vocabulary size
- Groups related word forms together
- Useful for search and information retrieval

### Disadvantages

- May produce invalid words
- Can over-stem words
- Less accurate than lemmatization

### Example of over-stemming

```text
universe → univers
```

Here, `univers` is not a valid English word.

---

## 8. Other Preprocessing Concepts

### Lowercasing

Converts all characters to lowercase.

```text
"Natural Language Processing"
→ "natural language processing"
```

### Stop-word removal

Removes frequently occurring words that add little meaning.

```text
"The cat is sitting on the mat"
→ ["cat", "sitting", "mat"]
```

Common stop words:

```text
a, an, the, is, are, in, on, of, and, or, to
```

### Punctuation removal

Removes punctuation symbols.

```text
"Hello, world!"
→ "Hello world"
```

### Removing extra whitespace

```text
"Hello    world"
→ "Hello world"
```

### Removing URLs, emails and special characters

```text
"Contact me at abc@gmail.com"
→ "Contact me at [EMAIL]"
```

### Noise removal

Removes irrelevant content such as HTML tags, URLs, hashtags, and repeated characters.

```text
"Great product!!! #shopping http://example.com"
→ "great product shopping"
```

### Vectorization

Converts processed text into numerical form.

| Method | Idea |
|---|---|
| Bag of Words | Counts word occurrences |
| TF-IDF | Gives importance to important words |
| Word embeddings | Represents words as dense vectors |
| Contextual embeddings | Represents words based on sentence context |

### Preprocessing pipeline

```text
Raw text
  ↓
Lowercasing
  ↓
Remove URLs, HTML, special characters
  ↓
Tokenization
  ↓
Stop-word removal
  ↓
Stemming or lemmatization
  ↓
Vectorization
  ↓
ML / NLP model
```

---

## 9. Stemming vs Lemmatization

| Aspect | Stemming | Lemmatization |
|---|---|---|
| Method | Removes suffixes using rules | Uses dictionary and morphology |
| Output | May be invalid word | Always valid dictionary word |
| Speed | Fast | Slower |
| Accuracy | Lower | Higher |
| Example | `studies → studi` | `studies → study` |
| Example | `better → better` | `better → good` |

> [!tip] When to use what
> Use **stemming** for fast search or text classification. Use **lemmatization** when correct meaning and grammar are important, such as chatbots and question answering.

---

## 10. Word Sense Disambiguation (WSD)

### Definition

**Word Sense Disambiguation (WSD)** is the process of identifying the correct meaning of an ambiguous word based on its context.

### Example

```text
"I deposited money in the bank."
```

Here, `bank` means **financial institution**.

```text
"We sat near the bank of the river."
```

Here, `bank` means **land beside a river**.

### Why WSD is needed

Many words have multiple meanings:

| Word | Possible meanings |
|---|---|
| bank | Financial institution, river side |
| bass | Low sound, type of fish |
| crane | Bird, lifting machine |
| plant | Living organism, factory |
| bat | Sports equipment, animal |

### Applications

- Machine translation
- Information retrieval
- Question answering
- Speech synthesis
- Semantic analysis

---

## 11. Dictionary-Based Approach for WSD

### Definition

The dictionary-based approach uses a machine-readable dictionary, thesaurus, or WordNet to identify the correct sense of an ambiguous word.

### Main idea

Words occurring together in a sentence are usually semantically related. Therefore, the correct sense of a word should have maximum overlap with the words in its context.

### Steps

```text
1. Identify the ambiguous target word.
2. Find all possible senses from a dictionary or WordNet.
3. Extract context words around the target word.
4. Remove stop words.
5. Compare each sense definition with the context words.
6. Count overlapping words.
7. Select the sense with the highest overlap.
```

### Example

```text
Sentence:
"The bank can guarantee deposits will cover future tuition costs."

Target word:
bank

Context words:
guarantee, deposits, cover, future, tuition, costs
```

| Sense | Definition | Overlap | Score |
|---|---|---|---:|
| Financial institution | A place that accepts deposits and provides loans | deposits | 1 |
| River side | Land beside a river | none | 0 |

Selected sense:

```text
bank = financial institution
```

### Advantages

- No labelled training data required
- Uses existing dictionaries or WordNet
- Simple and explainable
- Easy to implement

### Limitations

- Dictionary definitions may be too short
- May fail when there is no word overlap
- Depends on dictionary quality
- Struggles with domain-specific meanings

---

## 12. Lesk Algorithm

### Definition

The **Lesk algorithm** is a dictionary-based WSD algorithm. It selects the sense of an ambiguous word whose dictionary definition has maximum overlap with the definitions of surrounding context words.

### Original idea

```text
Score(sense) = overlap between
               sense definition of target word
               and
               definitions of context words
```

The sense with the highest score is selected.

### Simplified Lesk algorithm

```text
Score(sense) = overlap between
               sense definition of target word
               and
               context words
```

### Algorithm

```text
1. Input:
   - Sentence containing ambiguous word W
   - Dictionary containing senses and definitions

2. Extract context words C around W.
   Remove stop words.

3. For each sense S of W:
       Extract definition D of S
       Compute overlap = |D ∩ C|
       Store score(S) = overlap

4. Select sense S with maximum score(S).

5. Output the selected sense.
```

### Example

```text
Sentence:
"He went to the bank to withdraw money."

Target word:
bank

Context words after stop-word removal:
went, withdraw, money
```

| Sense | Definition | Overlap | Score |
|---|---|---|---:|
| Financial institution | A place where people deposit and withdraw money | withdraw, money | 2 |
| River side | Land along the edge of a river | none | 0 |

Selected sense:

```text
bank = financial institution
```

### Interview-ready answer

> Word Sense Disambiguation identifies the correct meaning of an ambiguous word using its context. The dictionary-based approach compares possible word senses from a dictionary or WordNet with the surrounding context. The Lesk algorithm selects the sense with the maximum overlap between its dictionary definition and the context words. For example, in “I deposited money in the bank,” the financial-institution sense is selected because its definition overlaps with words such as `deposit` and `money`.

---

## Quick Revision

```text
Morphological analysis → root + POS + features
FSA → recognizes valid word forms
FST → maps surface form to lexical form
Tokenization → splits text into tokens
Stemming → removes suffixes, may create invalid words
Lemmatization → returns proper dictionary word
Stop words → low-meaning frequent words
WSD → finds correct meaning using context
Dictionary-based WSD → compares sense definitions with context
Lesk algorithm → selects sense with maximum definition-context overlap
```