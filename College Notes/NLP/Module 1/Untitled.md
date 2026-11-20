---
title: Morphological Analysis using FST
subject: Natural Language Processing
semester: 7
tags: [NLP, morphology, FST, MU, BTech-CS, SEM7]
---
---
title: Morphological Analysis Output
subject: Natural Language Processing
semester: 7
tags: [NLP, morphology, FSA, FST, MU, BTech-CS, SEM7]
---

# Output of Morphological Analysis

Covers: **Regular verb, Irregular verb, Singular noun, Plural noun**

> Morphological analysis takes a **surface word** and gives its **stem + features**.
> It is done by an FST (input tape = surface form, output tape = lexical form).
> There is **one unified machine for nouns** and **one unified machine for verbs**. Regular and irregular forms are just different branches of the same machine.

---

## 1. Output Table

| Category | Input (surface) | Output (stem + features) |
|---|---|---|
| Singular noun (regular) | cat | `cat +N +SG` |
| Singular noun (irregular) | goose | `goose +N +SG` |
| Plural noun (regular) | cats | `cat +N +PL` |
| Plural noun (irregular) | geese | `goose +N +PL` |
| Regular verb (base) | walk | `walk +V +BASE` |
| Regular verb (3rd person) | walks | `walk +V +3SG` |
| Regular verb (past) | walked | `walk +V +PAST` |
| Regular verb (-ing) | walking | `walk +V +PRES-PART` |
| Irregular verb (base) | sing | `sing +V +BASE` |
| Irregular verb (past) | sang | `sing +V +PAST` |
| Irregular verb (past participle) | sung | `sing +V +PAST-PART` |
| Irregular verb (-ing) | singing | `sing +V +PRES-PART` |

**Tags:** `+N` noun, `+V` verb, `+SG` singular, `+PL` plural, `+3SG` 3rd person singular, `+PAST` past tense, `+PAST-PART` past participle, `+PRES-PART` present participle, `ε` empty string.

---

## 2. Unified Noun Machine (singular + plural, regular + irregular)

```mermaid
flowchart LR
    q0(("q0<br/>start")) -->|"reg-noun<br/>(cat, dog)"| q1(("q1"))
    q0 -->|"irreg-sg-noun<br/>(goose, mouse)"| q2(("q2"))
    q0 -->|"irreg-pl-noun<br/>(geese, mice)"| q3(("q3"))
    q1 -->|"ε : +N +SG"| q4((("q4<br/>final")))
    q1 -->|"s : +N +PL"| q4
    q2 -->|"ε : +N +SG"| q4
    q3 -->|"ε : +N +PL"| q4
```

| State | Meaning |
|---|---|
| q0 | Start |
| q1 | Regular noun stem read |
| q2 | Irregular singular noun read |
| q3 | Irregular plural noun read |
| q4 | Final (accept) |

**Outputs from this machine**

| Path | Example | Output |
|---|---|---|
| q0 → q1 → q4 (ε) | cat | `cat +N +SG` |
| q0 → q1 → q4 (s) | cats | `cat +N +PL` |
| q0 → q2 → q4 | goose | `goose +N +SG` |
| q0 → q3 → q4 | geese | `goose +N +PL` |

---

## 3. Unified Verb Machine (regular + irregular)

```mermaid
flowchart LR
    v0(("v0<br/>start")) -->|"reg-verb-stem<br/>(walk, talk)"| v1(("v1"))
    v0 -->|"irreg-verb-stem<br/>(sing, go)"| v2(("v2"))
    v0 -->|"irreg-past-verb<br/>(sang, went)"| v3(("v3"))
    v0 -->|"irreg-past-part<br/>(sung, gone)"| v4(("v4"))

    v1 -->|"ε : +V +BASE"| vf((("vf<br/>final")))
    v1 -->|"s : +V +3SG"| vf
    v1 -->|"ed : +V +PAST"| vf
    v1 -->|"ing : +V +PRES-PART"| vf

    v2 -->|"ε : +V +BASE"| vf
    v2 -->|"s : +V +3SG"| vf
    v2 -->|"ing : +V +PRES-PART"| vf

    v3 -->|"ε : +V +PAST"| vf
    v4 -->|"ε : +V +PAST-PART"| vf
```

| State | Meaning |
|---|---|
| v0 | Start |
| v1 | Regular verb stem read (takes -s, -ed, -ing) |
| v2 | Irregular verb stem read (takes -s, -ing, **not** -ed) |
| v3 | Irregular past form read (sang) |
| v4 | Irregular past participle read (sung) |
| vf | Final (accept) |

**Key difference:** a regular verb forms past with `-ed`. An irregular verb has its past and past participle **stored as separate words** in the lexicon, so it reaches the final state through v3 or v4 instead of `-ed`.

**Outputs from this machine**

| Path | Example | Output |
|---|---|---|
| v0 → v1 → vf (ε) | walk | `walk +V +BASE` |
| v0 → v1 → vf (s) | walks | `walk +V +3SG` |
| v0 → v1 → vf (ed) | walked | `walk +V +PAST` |
| v0 → v1 → vf (ing) | walking | `walk +V +PRES-PART` |
| v0 → v2 → vf (ε) | sing | `sing +V +BASE` |
| v0 → v2 → vf (ing) | singing | `sing +V +PRES-PART` |
| v0 → v3 → vf | sang | `sing +V +PAST` |
| v0 → v4 → vf | sung | `sing +V +PAST-PART` |

---

## 4. Step-by-Step State Transitions (one example each)

### Regular verb: walked → walk +V +PAST

```mermaid
flowchart LR
    A(("v0")) -->|"w:w"| B(("1"))
    B -->|"a:a"| C(("2"))
    C -->|"l:l"| D(("3"))
    D -->|"k:k"| E(("v1<br/>reg stem"))
    E -->|"ε:+V"| F(("4"))
    F -->|"e:+PAST"| G(("5"))
    G -->|"d:ε"| H((("vf<br/>final")))
```

### Irregular verb: sang → sing +V +PAST

```mermaid
flowchart LR
    A(("v0")) -->|"s:s"| B(("1"))
    B -->|"a:i"| C(("2"))
    C -->|"n:n"| D(("3"))
    D -->|"g:g"| E(("v3<br/>irreg past"))
    E -->|"ε:+V +PAST"| F((("vf<br/>final")))
```

### Singular noun: cat → cat +N +SG

```mermaid
flowchart LR
    A(("q0")) -->|"c:c"| B(("1"))
    B -->|"a:a"| C(("2"))
    C -->|"t:t"| D(("q1<br/>reg noun"))
    D -->|"ε:+N"| E(("3"))
    E -->|"ε:+SG"| F((("q4<br/>final")))
```

### Plural noun: geese → goose +N +PL

```mermaid
flowchart LR
    A(("q0")) -->|"g:g"| B(("1"))
    B -->|"e:o"| C(("2"))
    C -->|"e:o"| D(("3"))
    D -->|"s:s"| E(("4"))
    E -->|"e:e"| F(("q3<br/>irreg pl"))
    F -->|"ε:+N +PL"| G((("q4<br/>final")))
```

---

## 5. Quick Revision Points

- Output = **stem + morphological features**.
- **One FSA for nouns, one for verbs.** Regular and irregular are branches of the same machine, selected by the lexicon class.
- Regular noun: stem + `s` gives `+PL`, stem alone gives `+SG`.
- Irregular noun: whole form stored (`geese`, `mice`), mapped to stem + `+PL`.
- Regular verb: stem + `s` / `ed` / `ing`.
- Irregular verb: stored past (`sang`) and participle (`sung`) forms, no `-ed`.
- FST transition format is `input:output`, and `ε` means empty.
- 