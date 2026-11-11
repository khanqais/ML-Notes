**POS (Part-of-Speech) tagging** is an NLP task where every token in a sentence is assigned a grammatical category, such as noun, verb, adjective, adverb, preposition, or pronoun. It is a foundational step for tasks like parsing, named-entity recognition, machine translation, and information extraction.

## Example

Consider:

> “The quick brown fox jumps over the lazy dog.”

A POS-tagged output could be:

| Word  | POS tag          |
| ----- | ---------------- |
| The   | Determiner (DT)  |
| quick | Adjective (JJ)   |
| brown | Adjective (JJ)   |
| fox   | Noun (NN)        |
| jumps | Verb (VBZ)       |
| over  | Preposition (IN) |
| the   | Determiner (DT)  |
| lazy  | Adjective (JJ)   |
| dog   | Noun (NN)        |

Here, the tagger identifies the grammatical role of each word in context. POS tagging is also called **grammatical tagging**.
## Main challenges

### 1. Ambiguity

This is the biggest challenge. Many words can belong to more than one grammatical category, and the correct tag depends on the surrounding words.[[geeksforgeeks](https://www.geeksforgeeks.org/nlp/nlp-part-of-speech-default-tagging/)][[people.engr.tamu](https://people.engr.tamu.edu/huangrh/Fall18-638/HMM_notes_collins.pdf)]

Examples:

|Sentence|Word|Correct POS|
|---|---|---|
|“I saw a **bear**.”|bear|Noun|
|“She can’t **bear** the pain.”|bear|Verb|
|“I read a **book**.”|book|Noun|
|“Please **book** a ticket.”|book|Verb|
|“This **can** is difficult to open.”|can|Noun|
|“You **can** open it.”|can|Modal verb|

A tagger must use context to decide whether _bear_ is an animal or an action, and whether _book_ refers to an object or means “reserve.

### 2. Unknown or out-of-vocabulary words

A POS tagger may fail on words that were not present in its training data, especially new words, slang, names, abbreviations, and technical terms. Rare or unseen words are a known limitation of statistical taggers.

Examples:

- “The startup is developing **cryptojacking** tools.”
- “He posted a **selfie** on Instagram.”
- “The patient was diagnosed with **hyperlipidemia**.”

If the tagger has never seen _cryptojacking_ or _hyperlipidemia_, it may incorrectly classify them. Modern subword-based models reduce this problem, but it still matters for low-resource languages and specialized domains.

### 3. Contextual limitations

Simple statistical models, such as Hidden Markov Models, often decide a word’s tag using only a limited window of nearby tags. They may fail when the required context is farther away or when sentence structure is complex.

Example:

> “The old man the boats.”

This is a garden-path sentence. A tagger may initially treat _man_ as a noun, but in this sentence it functions as a verb. Correct tagging requires understanding the full syntactic structure.

### 4. Domain dependence

A model trained on news articles may perform poorly on social-media posts, medical reports, legal documents, or code-mixed text because vocabulary and grammar differ across domains.

Examples:

- In general English, “tweet” is usually a verb or noun.
- In social-media text, “tweeting,” hashtags, emojis, and abbreviations create unusual patterns.
- In legal or medical text, terms such as “plaintiff,” “lesion,” or “adverse event” may be rare in general training corpora.

Therefore, models often need domain-specific fine-tuning or retraining.[[link.springer](https://link.springer.com/article/10.1186/s40537-022-00561-y)][[pdfs.semanticscholar](https://pdfs.semanticscholar.org/1bb3/375736171f2657ca429b060d2cf3b47553ee.pdf)]

### 5. Multi-word expressions

Some phrases act as one grammatical unit, but a token-level tagger may split them incorrectly.[[scribd](https://www.scribd.com/document/871611175/Issues-in-Pos-Tagging-pptx)]

Example:

- “New York” should be treated as a proper noun, not as adjective _New_ plus noun _York_.
- “Give up” functions as a phrasal verb, not as verb _give_ plus preposition _up_.
- “In spite of” acts as a preposition-like phrase.

Incorrect splitting can affect downstream parsing and meaning extraction.

### 6. Informal language and idioms

Slang, typos, emojis, contractions, and idiomatic expressions are difficult because they may not follow standard grammatical patterns.[[geeksforgeeks](https://www.geeksforgeeks.org/nlp/nlp-part-of-speech-default-tagging/)][[tinkogroup](https://tinkogroup.com/part-of-speech-tagging-in-nlp-and-machine-learning/)]

Examples:

- “That movie was **lit**.”
- “I’m **gonna** go.”
- “He **kicked the bucket**.”

Here, _lit_ may be an adjective in slang, _gonna_ is an informal contraction of “going to,” and _kicked the bucket_ is an idiom meaning “died.” A tagger trained mainly on formal text may mislabel these.

### 7. Data scarcity and class imbalance

Some POS tags occur far more frequently than others. For example, nouns, verbs, determiners, and prepositions are common, while interjections, particles, and rare grammatical forms may have very few examples. This imbalance can reduce tagging accuracy for minority categories.[[link.springer](https://link.springer.com/article/10.1186/s40537-022-00561-y)]

Also, low-resource languages and dialects often lack large, high-quality annotated corpora, making it difficult to train reliable taggers.[[link.springer](https://link.springer.com/article/10.1186/s40537-022-00561-y)][[pdfs.semanticscholar](https://pdfs.semanticscholar.org/1bb3/375736171f2657ca429b060d2cf3b47553ee.pdf)]

### 8. Language-specific complexity

Languages differ greatly in grammar. Languages with rich morphology, flexible word order, or agglutinative structure can be harder to tag than English.[[tinkogroup](https://tinkogroup.com/part-of-speech-tagging-in-nlp-and-machine-learning/)]

For example, in Hindi or many Indian languages, a word may encode gender, number, person, tense, and case through suffixes. A tagger must handle these inflections correctly, which requires language-specific training data and tag sets.

## Interview-ready answer

