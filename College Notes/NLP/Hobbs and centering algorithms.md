# Hobbs Algo:

![[Pasted image 20261108172345.png]]
![[Screenshot 2026-11-08 171708.png]]


# Centering Algorithm

**Centering Theory** is an NLP method used to understand how the **focus/center of a conversation or text shifts from one sentence to the next**. It helps a computer resolve which noun or pronoun refers to which entity.

## Why we need it

In a conversation, humans easily understand who pronouns like **he, she, him, her, it** refer to. A computer needs an algorithm to do the same.

**Example:**

1. Rahul is reading a book.
2. He took help from Vineet.
3. He helped him.

Here:

- In sentence 2, **“he”** refers to **Rahul**.
- In sentence 3, **“he”** refers to **Vineet**, while **“him”** refers to **Rahul**.

So the focus shifted from **Rahul** to **Vineet**. Centering theory tracks this shift.

## Main purpose

Centering algorithm helps computers figure out:

- Which **noun** or **pronoun** refers to which **entity**
- Who or what the conversation is mainly about
- How the **focus** changes across sentences

It is mainly used for **coreference resolution** and discourse understanding.

## Difference from Hobbs algorithm

|Point|Hobbs Algorithm|Centering Algorithm|
|---|---|---|
|Main goal|Find antecedent of a pronoun|Track focus/center across sentences|
|Approach|Uses syntactic parsing and search rules|Uses focus, center, and ranking|
|Works with|Syntax-based rules|Discourse-level attention/focus|
|Example use|Resolving “he” or “him”|Understanding who the conversation is about|

Hobbs algorithm depends more on **syntactic parsing**, whereas centering focuses on the **main entity or focus** of the sentence.

## Important terms

### Entity

An entity is a person, object, or thing being talked about.

Example:

> Rahul is reading a book.

Entities:

- Rahul
- Book

### Forward-looking centers (CF)

**Forward-looking centers (CF)** are all entities mentioned in the **current sentence** that may become important in the **next sentence**.

Example:

> Rahul is reading a book.

CF:

- Rahul
- Book

Both may be referred to later in the text.

### Backward-looking centers (CB)

**Backward-looking centers (CB)** are entities that were the main focus in the **previous sentence** and are still mentioned in the **current sentence**.

Example:

1. Rahul is reading a book.
2. He took help from Vineet.

For sentence 2:

- **He** refers to Rahul.
- Rahul was the main focus of sentence 1.
- Therefore, Rahul is the **CB** of sentence 2.

### Preferred center (CP)

**Preferred center (CP)** is the most important entity in the current sentence.

It is usually the highest-ranked entity in the CF list.

Example:

> Rahul is reading a book.

CF list:

1. Rahul
2. Book

So:

```
CP = Rahul
```

Because **Rahul** is the subject and has higher priority than **book**.

## Ranking / priority order

The centering algorithm assigns priority using this order:

1. **Subject**
2. **Existential predicate nominal**
3. **Object**
4. **Indirect object**
5. **Other entities**

Example:

> Rahul is reading a book.

|Rank|Entity|Role|
|---|---|---|
|1|Rahul|Subject|
|2|Book|Object|

Therefore:

```
CF = [Rahul, Book]
CP = Rahul
```

The subject gets the highest priority because it usually represents the main focus of the sentence.

## Steps of centering algorithm

### Step 1: Identify all entities

Find all nouns and pronouns in each sentence.

Example:

> Rahul is reading a book.

Entities:

- Rahul
- Book

### Step 2: Rank the entities

Use the priority order:
1. **Subject** — Who/what performs the action.
2. **Predicate nominal** — A noun after “is/am/are/was/were” that describes or identifies the subject.
3. **Object** — Who/what receives the action.
4. **Indirect object** — To whom/for whom the action is done.
5. **Others** — Adverbs, modifiers, prepositional phrases, etc.

```
Subject > Predicate nominal > Object > Indirect object > Others
```

For the above sentence:

```
1. Rahul
2. Book
```

### Step 3: Find backward-looking centers (CB)

Check the previous sentence and identify which entity is still mentioned in the current sentence.

If the current sentence is the first sentence, then:

```
CB = None
```

### Step 4: Find preferred center (CP)

Choose the highest-ranked entity from the current sentence.

```
CP = highest-ranked CF
```

### Step 5: Determine transition

Compare **CB** and **CP** to find what type of transition happened.

### Step 6: Repeat for every sentence

Apply the same steps for each new sentence in the text.

## Types of transitions

|Transition|Meaning|Example|
|---|---|---|
|**Continue**|The same entity remains the main focus|Rahul remains the focus|
|**Retain**|The same entity is mentioned, but focus starts shifting|Rahul is still mentioned, but Vineet is introduced|
|**Smooth shift**|Focus changes to a new but related entity|Focus moves from Rahul to Vineet|
|**Rough shift**|Topic suddenly jumps to an unrelated entity|A completely new unrelated topic/person appears|

In most simple examples, **continue**, **retain**, and **smooth shift** are commonly used. **Rough shift** happens when the topic changes abruptly.

## Full solved example

### Given text

1. Rahul is reading a book.
2. He took help from Vineet.
3. He helped him.

### Sentence 1: Rahul is reading a book

**Entities:**

```
Rahul
Book
```

**Ranking:**

```
1. Rahul  -> Subject
2. Book   -> Object
```

**CF:**

```
[Rahul, Book]
```

**CB:**

```
None
```

Because there is no previous sentence.

**CP:**

```
Rahul
```

**Transition:**

```
No transition
```

Because it is the first sentence.

### Sentence 2: He took help from Vineet

**Entities:**

```
He
Vineet
```

Here, **He** refers to **Rahul**.

**CF:**

```
[Rahul, Vineet]
```

**CB:**

```
Rahul
```

Because Rahul was the main focus in sentence 1 and is still mentioned in sentence 2 through “he”.

**CP:**

```
Rahul
```

Because Rahul is the subject of the current sentence.

**Transition:**

```
Continue
```

Because the focus is still on **Rahul**.

### Sentence 3: He helped him

**Entities:**

```
He
Him
```

Here:

- **He** refers to **Vineet**
- **Him** refers to **Rahul**

**CF:**

```
[Vineet, Rahul]
```

**CB:**

```
Vineet
```

Because Vineet was introduced in sentence 2 and is now the subject in sentence 3.

**CP:**

```
Vineet
```

**Transition:**

```
Smooth Shift
```

Because the focus shifted from **Rahul** to **Vineet**.

## Final answer table

|Sentence|CF|CB|CP|Transition|
|---|---|---|---|---|
|Rahul is reading a book.|Rahul, Book|None|Rahul|No transition|
|He took help from Vineet.|Rahul, Vineet|Rahul|Rahul|Continue|
|He helped him.|Vineet, Rahul|Vineet|Vineet|Smooth Shift|

## Exam-ready explanation

**Centering algorithm** is an NLP technique used to track the main focus or center of a conversation across multiple sentences. It helps determine which noun or pronoun refers to which entity.

It uses three main terms:

- **CF (Forward-looking centers):** Entities mentioned in the current sentence that may be important later.
- **CB (Backward-looking centers):** Entity that was the main focus in the previous sentence and is still mentioned now.
- **CP (Preferred center):** Most important entity in the current sentence, based on ranking.

Priority is given as:

```
Subject > Predicate nominal > Object > Indirect object > Others
```

Transitions can be:

```
Continue, Retain, Smooth Shift, Rough Shift
```

## Quick example for exam

**Text:**

1. Sita is writing a letter.
2. She gave it to Ram.
3. He read it.

### Sentence 1

```
CF = [Sita, Letter]
CB = None
CP = Sita
Transition = No transition
```

### Sentence 2

```
She = Sita
It = Letter

CF = [Sita, Letter, Ram]
CB = Sita
CP = Sita
Transition = Continue
```

### Sentence 3

```
He = Ram
It = Letter

CF = [Ram, Letter]
CB = Ram
CP = Ram
Transition = Smooth Shift
```

Here, the focus shifts from **Sita** to **Ram**, so the transition is **smooth shift**.

## Important points to remember

- Centering algorithm is used in **NLP** for discourse understanding.
- It helps in **pronoun resolution** and **coreference resolution**.
- **CF** means forward-looking centers.
- **CB** means backward-looking centers.
- **CP** means preferred center.
- Subject gets the highest priority.
- The focus may remain the same or shift to another entity.
- Main transitions are **continue, retain, smooth shift, and rough shift**.