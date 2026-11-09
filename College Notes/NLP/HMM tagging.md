![[Pasted image 20261109195414.png]]So the first algo is:
![[Pasted image 20261109195436.png]]![[Pasted image 20261109195447.png]]but this doesnt work all the time cuz word have different meaning accordgin to the context

![[Pasted image 20261109195535.png]]
to solve this we can make a bigram trigram and compared all of them to find the context
![[Pasted image 20261109195620.png]]but there is one more problem:
![[Pasted image 20261109195813.png]]Your table stores word pairs such as `jane-can`, `can-see`, and `will-see`, along with their possible tag pairs. But the test sentence contains `Jane will`, which does not appear as a pair in the training sentences shown. So a direct lookup has no evidence for that pair.[image.jpg](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/images/50124771/70594132-3d21-46be-866e-829c190907f9/image.jpg)

This is called **sparsity**: many valid combinations are missing from limited training data. Using longer word sequences—trigrams, four-grams, etc.—makes exact matches even harder to find.[aclanthology](https://aclanthology.org/N09-2054.pdf)

## 2. How does an HMM help?

Instead of requiring the exact word pair, it combines two reusable probabilities:

- Transition: P(next tag∣previous tag)P(\text{next tag}\mid\text{previous tag})P(next tag∣previous tag), such as how likely `N → M` is.
    
- Emission: P(word∣tag)P(\text{word}\mid\text{tag})P(word∣tag), such as how likely the tag `M` is to produce “will.”[web.stanford](https://web.stanford.edu/~jurafsky/slp3/old_dec21/8.pdf)
    

So even without seeing `Jane will` together, it can combine evidence that Jane occurs as a noun, “will” occurs as a modal, and nouns can be followed by modals
![[Pasted image 20261109195906.png]]![[Pasted image 20261109195930.png]]![[Pasted image 20261109195942.png]]![[Pasted image 20261109200010.png]]![[Pasted image 20261109200023.png]]![[Pasted image 20261109200035.png]]
# Limitations

The main limitation of an HMM POS tagger is that it makes strong simplifying assumptions: a tag depends mostly on a very small local context, and a word depends only on its own tag. This makes HMMs fast and easy to train, but weak for long-distance syntax, unknown words, and rich contextual features.

## Main limitations

### 1. Limited context

A basic HMM uses the **Markov assumption**: the current tag depends only on the previous tag.

```
P(t_i \mid t_1,\ldots,t_{i-1}) \approx P(t_i \mid t_{i-1})
```

A trigram HMM uses the previous two tags, but it still cannot capture dependencies spread across a long sentence

Example:

> “The keys to the cabinet on the table **are** missing.”

Here, _keys_ and _are_ are grammatically connected even though many words separate them. A bigram HMM may fail because it only looks at the immediately preceding tag.

### 2. Strong independence assumptions

HMM assumes that each word is emitted independently based only on its own POS tag:

```
P(w_i \mid t_i)
```

It does not naturally model interactions between words, such as subject-verb agreement, semantic meaning, or phrase structure.[[cl.cam.ac](https://www.cl.cam.ac.uk/teaching/1516/L101/clark_lecs/notes_2.pdf)][[scribd](https://www.scribd.com/document/933301741/finalnlp)]

Example:

> “The dogs bark.”

The plural noun _dogs_ and plural verb _bark_ are related, but a basic HMM treats each word’s emission separately. It may not fully use this agreement information.

### 3. Unknown-word problem

If a word was not seen during training, the HMM has no emission probability for it:

```
P(\text{new word} \mid \text{tag}) = 0
```

This can incorrectly force the probability of an entire tag sequence to zero. HMM taggers therefore require smoothing and special handling for unknown words.[[cl.cam.ac](https://www.cl.cam.ac.uk/teaching/1617//L101/slides2.pdf)][[uima.apache](https://uima.apache.org/downloads/sandbox/hmmTaggerUsersGuide/hmmTaggerUsersGuide.html)]

Example:

> “The startup is developing quantumcrypto tools.”

If _quantumcrypto_ never appeared in training data, the model has no direct probability for it. It may assign the wrong tag or fail to tag it properly.

### 4. Data sparsity

HMMs need a large annotated corpus to estimate transition and emission probabilities reliably. Rare tag transitions, rare word-tag pairs, and unseen tag sequences may have zero or unreliable counts.[[cl.cam.ac](https://www.cl.cam.ac.uk/teaching/1516/L101/clark_lecs/notes_2.pdf)][[cl.cam.ac](https://www.cl.cam.ac.uk/teaching/1617//L101/slides2.pdf)]

Example:

> “The CEO announced the acquisition.”

If the training data rarely contains the transition:

```
\text{PROPER NOUN} \rightarrow \text{VERB}
```

then the HMM may mislabel _announced_. Smoothing helps, but it cannot fully solve the sparsity problem.

### 5. Difficulty with ambiguity

POS ambiguity is still difficult because the same word can have multiple possible tags. The HMM chooses based mainly on nearby tags and emission counts, not deeper sentence meaning.[[scribd](https://www.scribd.com/document/933301741/finalnlp)][[link.springer](https://link.springer.com/article/10.1186/s40537-022-00561-y?error=cookies_not_supported&code=75dfb8c3-5d50-4a3c-a52b-01823a3919e7)]

Example:

|Sentence|Word|Correct tag|
|---|---|---|
|“I read a **book**.”|book|Noun|
|“Please **book** a ticket.”|book|Verb|
|“I saw a **bear**.”|bear|Noun|
|“I can’t **bear** this.”|bear|Verb|

A simple HMM may choose the wrong tag when the surrounding context is unusual or insufficient.

### 6. Weak feature representation

HMMs are generative models, so adding rich overlapping features—such as word suffixes, capitalization, previous words, semantic information, or dependency information—is difficult and can make probability estimation complicated. Discriminative models such as CRFs handle such features more naturally.[[cl.cam.ac](https://www.cl.cam.ac.uk/teaching/1516/L101/clark_lecs/notes_2.pdf)][[cl.cam.ac](https://www.cl.cam.ac.uk/teaching/1617//L101/slides2.pdf)]

Example:

- A word ending in **-ing** is often a verb form, but it can also be a noun or adjective.
- A capitalized word is often a proper noun, but not always.
- A word after _the_ is often a noun, but context can change this.

An HMM cannot easily combine all these clues in a flexible way.

### 7. Domain dependence

A model trained on one type of text may perform poorly on another domain because vocabulary, writing style, and tag distributions change. Accuracy typically decreases when the test corpus differs from the training corpus, mainly because of more unknown words.[[uima.apache](https://uima.apache.org/downloads/sandbox/hmmTaggerUsersGuide/hmmTaggerUsersGuide.html)]

Example:

- A tagger trained on news articles may struggle with tweets.
- It may fail on code-mixed Hindi-English text.
- It may mislabel medical or legal terms not seen during training.

Example:

> “Bhai, this reel is fire 🔥.”

Informal words, slang, emojis, and mixed-language text may be absent from a formal training corpus.

### 8. Fixed tag set

The HMM can output only the tags present in its training data. If a new annotation scheme or language uses tags not seen during training, the model cannot produce them.

Example:

- Training data uses `NN` for nouns.
- A new dataset uses `PROPN` for proper nouns.
- The HMM cannot output `PROPN` unless it is retrained or the tag set is changed.

### 9. Lower accuracy on rare tags

Frequent tags such as nouns, verbs, determiners, and prepositions have many examples, while rare tags may have very few. This class imbalance causes lower accuracy for less frequent POS categories.[[link.springer](https://link.springer.com/article/10.1186/s40537-022-00561-y?error=cookies_not_supported&code=75dfb8c3-5d50-4a3c-a52b-01823a3919e7)]

Example:

- Interjections such as _wow_, _oops_, and _hmm_ may occur rarely.
- Rare verb forms or unusual grammatical constructions may be poorly learned.

### 10. Less powerful than modern models

HMMs are still useful because they are fast, simple, and can be trained efficiently by counting probabilities. However, modern sequence models such as CRFs, RNNs, and transformer-based models can use richer context and features, so they generally perform better on complex language tasks.
