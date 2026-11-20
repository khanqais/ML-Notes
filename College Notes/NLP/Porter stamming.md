---
tags: [NLP, stemming, porter, exam, mumbai-university]
subject: Natural Language Processing
---

# Porter Stemming Algorithm

## 1. Definition

**Porter Stemming** is a **rule-based** algorithm used in NLP to reduce English words to their **root/stem** form by removing common suffixes. Developed by **Martin Porter in 1980**.

| Word | Stem |
|---|---|
| running | run |
| connected | connect |
| studies | studi |

> [!note]
> The stem need not be a real word (studies → studi).

---

## 2. Prerequisites

### Vowels and Consonants
- **Vowels:** a, e, i, o, u
- **Consonants:** all other letters
- **y** is a vowel if the previous letter is a consonant (happy → y is a vowel).

### Word Form

$$[C](VC)^m[V]$$

- C = consonant group, V = vowel group
- **m = measure** = number of VC groups

| Word | Form | m |
|---|---|---|
| tree | CV | 0 |
| trouble | C VC V | 1 |
| private | C VC VC V | 2 |

> [!important]
> The measure **m** decides whether a suffix can be removed.

---

## 3. Steps of the Algorithm

```mermaid
flowchart LR
    A[Word] --> B[Step 1a/1b/1c] --> C[Step 2] --> D[Step 3] --> E[Step 4] --> F[Step 5] --> G[Stem]
```

### Step 1a: Plural Removal

| Rule | Example |
|---|---|
| sses → ss | caresses → caress |
| ies → i | ponies → poni |
| ss → ss | caress → caress |
| s → (remove) | cats → cat |

### Step 1b: Remove -ed / -ing

| Rule | Example |
|---|---|
| (m>0) eed → ee | agreed → agree |
| (stem has vowel) ed → (remove) | plastered → plaster |
| (stem has vowel) ing → (remove) | running → runn |

**Cleanup after removing ed/ing:**

| Rule | Example |
|---|---|
| at → ate | conflated → conflate |
| bl → ble | troubled → trouble |
| iz → ize | sized → size |
| double consonant (not l, s, z) → single | runn → run |

### Step 1c: y → i

**Rule:** if the stem contains a vowel, y → i

| Example | Reason |
|---|---|
| happy → happi | stem "happ" has a vowel |
| enjoy → enjoi | stem "enjo" has a vowel |
| sky → sky | stem "sk" has no vowel, so no change |

### Step 2: Derivational Suffixes (m > 0)

| Rule | Example |
|---|---|
| ational → ate | relational → relate |
| tional → tion | conditional → condition |
| izer → ize | digitizer → digitize |
| fulness → ful | hopefulness → hopeful |
| ousness → ous | callousness → callous |
| biliti → ble | sensibiliti → sensible |

### Step 3: Remove More Suffixes (m > 0)

| Rule | Example |
|---|---|
| icate → ic | triplicate → triplic |
| ative → (remove) | formative → form |
| alize → al | formalize → formal |
| ical → ic | electrical → electric |
| ful → (remove) | hopeful → hope |
| ness → (remove) | goodness → good |

### Step 4: Remove Remaining Suffixes (only if m > 1)

Suffixes: **al, ance, ence, er, ment, ion, ate, ous, ive**

- **ion** is removed only if the stem ends with **s** or **t**.

| Example | Result |
|---|---|
| revival | reviv |
| adjustment | adjust |
| adoption | adopt |

### Step 5: Final Cleanup

**5a: Remove final e**
- If m > 1: remove final e (probate → probat)
- If m = 1 and stem does **not** end in CVC: remove final e
- Otherwise keep it (rate → rate)

**5b: Remove double l**
- If m > 1 and word ends with **ll**: remove one l (controll → control)

---

## 4. Solved Example: relational

| Step | Rule applied | Result |
|---|---|---|
| Step 1 | no rule applies | relational |
| Step 2 | ational → ate | relate |
| Step 3, 4 | no rule applies | relate |
| Step 5a | m = 2 (> 1), remove final e | **relat** |

**Final stem: relat**

---

## 5. Advantages
- Simple and fast
- Useful in search engines and information retrieval
- Reduces different forms of a word to one stem (smaller index)

## 6. Disadvantages
- Output may not be a real word (studies → studi)
- Less accurate than **lemmatization** (which uses a dictionary and gives valid words)
- English only; can over-stem or under-stem

---

## 7. Quick Revision
- Algorithm is **rule-based**, suffix stripping, **5 steps**
- **m** decides whether a suffix is removed
- Step 1: plurals, -ed/-ing, y → i
- Step 2 and 3: m > 0, Step 4: m > 1
- Step 5: final e and double l cleanup
- Stem may not be a valid word
