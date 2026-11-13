# Natural Language Processing (NLP) Process

## Concept

**Natural Language Processing (NLP)** is a branch of Artificial Intelligence that enables computers to understand, interpret, analyze, and generate human language in the form of text or speech.

NLP converts unstructured human language into a form that a machine can process. It is used in chatbots, search engines, machine translation, sentiment analysis, voice assistants, and text summarization.

The classical NLP process consists of **five stages**:

1. Lexical Analysis / Morphological Analysis
2. Syntactic Analysis / Parsing
3. Semantic Analysis
4. Discourse Integration
5. Pragmatic Analysis

Each stage processes the input language at a deeper level:

```text
Raw Text
   ↓
Lexical Analysis
   ↓
Syntactic Analysis
   ↓
Semantic Analysis
   ↓
Discourse Integration
   ↓
Pragmatic Analysis
   ↓
Meaning / Action / Output
```

### Example sentence

Consider the following input:

```text
Rahul ordered a pizza. It was delivered late, so he was unhappy.
```

The NLP system must:

- Break the text into words and sentences
- Check the grammar of each sentence
- Understand the meaning of words such as `late` and `unhappy`
- Identify that `It` refers to `pizza`
- Identify that `he` refers to `Rahul`
- Understand that Rahul is dissatisfied with the delivery service

---

## 1. Lexical Analysis

Lexical analysis is the first stage of NLP. It is also called **morphological analysis**.

In this stage, the system breaks raw text into smaller meaningful units called **tokens**. A token can be a word, punctuation mark, number, or symbol.

Lexical analysis also identifies:

- Word boundaries
- Sentences
- Punctuation
- Roots of words
- Prefixes and suffixes
- Parts of speech

### Main tasks

- Tokenization
- Stemming
- Lemmatization
- Part-of-speech tagging
- Stop-word removal
- Text normalization

### Example

Input:

```text
The cats are running quickly.
```

Tokens:

```text
["The", "cats", "are", "running", "quickly", "."]
```

Morphological analysis:

```text
cats     = cat + s
running  = run + ing
quickly  = quick + ly
```

Part-of-speech tags:

```text
The      → Determiner
cats     → Noun
are      → Auxiliary verb
running  → Verb
quickly  → Adverb
```

Another example:

```text
Unbelievable = un + believe + able
```

Here:

- `un` is a prefix
- `believe` is the root word
- `able` is a suffix

### Challenges in Lexical Analysis

| Challenge | Explanation | Example |
|---|---|---|
| Tokenization problems | Deciding where one token ends and the next begins | `don't` → `do` + `n't` or one token? `U.S.A.` – the full stops are not sentence ends |
| Multi-word expressions | Some words together form one unit and should not be split | `New York`, `ice cream`, `kick the bucket` |
| No clear word boundaries | Some languages do not use spaces between words | Chinese, Japanese, Thai. Joined words in Hindi/Sanskrit (sandhi) |
| Spelling mistakes | Typos produce words that are not in the dictionary | `recieve`, `teh`, `adress` |
| Slang, abbreviations, and emojis | Informal text is not found in standard dictionaries | `gr8`, `lol`, `idk`, `😂` |
| Code-mixing | Two languages are mixed in one sentence | `Mujhe yeh phone bahut accha laga` (Hinglish) |
| Stemming errors | Stemmers cut words using rules, and sometimes cut wrongly | Over-stemming: `university` and `universe` → `univers`. Under-stemming: `data` and `datum` stay different |
| Rich morphology | Some languages have many word forms for one root | Hindi, Finnish, Turkish, Arabic |
| Irregular word forms | Rules fail for irregular words | `went` → `go`, `mice` → `mouse` |
| Special entities | Dates, URLs, emails, and hashtags need special handling | `15/01/2026`, `#NLP`, `abc@gmail.com` |

### Ambiguity in Lexical Analysis

Lexical ambiguity means a **single word (or word form) can be understood in more than one way** at the word level.

**1. Part-of-speech (POS) ambiguity**

The same word can belong to different parts of speech.

```text
I will book a ticket.      → book = Verb
I read a book.             → book = Noun

Time flies like an arrow.  → flies = Verb
Fruit flies like a banana. → flies = Noun
```

**2. Homonymy / polysemy (same spelling, different meanings)**

```text
bat → cricket bat / flying animal
bank → financial institution / river side
```

*(The meaning is finally decided in semantic analysis, but the ambiguity begins at the word level.)*

**3. Morphological ambiguity**

A word can be split into parts in more than one way.

```text
unlockable = (un + lock) + able → "able to be unlocked after it was locked"
unlockable = un + (lock + able) → "cannot be locked"
```

**4. Tokenization ambiguity**

```text
Dr. Sharma lives in St. Louis.
```

Here the full stop after `Dr.` and `St.` is not the end of a sentence, but a simple rule may treat it as one.

**5. Inflection ambiguity**

```text
saw → past tense of "see"  /  a cutting tool (noun)  /  "to cut" (verb)
```

---

## 2. Syntactic Analysis

Syntactic analysis is also called **parsing**.

In this stage, the NLP system checks whether a sentence follows the grammar rules of the language. It identifies the relationship between words, such as subject, verb, object, and modifiers.

The output of syntactic analysis is usually a **parse tree** or a **dependency structure**.

### Main tasks

- Grammar checking
- Sentence structure analysis
- Identifying subject, verb, and object
- Building a parse tree
- Dependency parsing
- Constituency parsing

### Example

Input:

```text
The boy ate the apple.
```

The parser identifies:

```text
The boy     → Noun Phrase / Subject
ate         → Verb
the apple   → Noun Phrase / Object
```

### Parse tree

```text
                    S
                  /   \
                NP     VP
               /  \   /  \
             Det   N V    NP
              |    | |   /  \
             The boy ate Det  N
                          |   |
                         the apple
```

### Grammar rules

```text
S  → NP VP
NP → Det N
VP → V NP
Det → the
N  → boy | apple
V  → ate
```

Here:

- `S` means Sentence
- `NP` means Noun Phrase
- `VP` means Verb Phrase
- `Det` means Determiner
- `N` means Noun
- `V` means Verb

### Important point

The second level of the parse tree contains only:

```text
S → NP VP
```

The object `the apple` is **not** a separate child of `S`.

It is part of the verb phrase:

```text
VP → V NP
```

So:

```text
ate the apple
```

is one complete verb phrase.

### Incorrect sentence

```text
Ate apple the boy an.
```

This sentence is syntactically incorrect because the words are not arranged according to English grammar rules.

### Challenges in Syntactic Analysis

| Challenge | Explanation | Example |
|---|---|---|
| Multiple possible parse trees | One sentence may fit the grammar in several ways | `I saw the man with the telescope.` |
| Free word order | Some languages allow words to move without changing the meaning | Hindi: `Ram ne seb khaya` = `Seb Ram ne khaya` |
| Long and complex sentences | Nested clauses make parsing slow and error-prone | `The book that the man who lives next door bought was torn.` |
| Ungrammatical input | Real-world text (chats, tweets, speech) often breaks grammar rules | `me going market tmrw` |
| Incomplete grammar coverage | It is impossible to write rules for every sentence structure | Rare or new sentence patterns |
| Ellipsis (missing words) | Parts of the sentence are omitted | `John likes tea and Mary coffee.` (the verb `likes` is missing in the second part) |
| Long-distance dependencies | Related words are far apart in the sentence | `The keys that I left on the table yesterday are missing.` (`keys` ↔ `are`) |
| Computational cost | Parsing all possible trees is expensive | Long documents take much time |
| Speech disfluencies | Spoken language has repetitions and fillers | `I, uh, I want, um, a ticket` |

### Ambiguity in Syntactic Analysis

Syntactic (structural) ambiguity means a **sentence can have more than one valid parse tree**, so more than one grammatical structure.

**1. Prepositional-phrase (PP) attachment ambiguity**

```text
I saw the man with the telescope.
```

```text
Meaning 1: I used a telescope to see the man.   → "with the telescope" attaches to the VERB "saw"
Meaning 2: The man had a telescope.             → "with the telescope" attaches to the NOUN "man"
```

**2. Coordination (scope) ambiguity**

```text
Old men and women were rescued.
```

```text
Meaning 1: (Old men) and (women)       → only the men are old
Meaning 2: Old (men and women)         → both are old
```

**3. Noun-phrase / modifier ambiguity**

```text
Flying planes can be dangerous.
```

```text
Meaning 1: The act of flying planes is dangerous.
Meaning 2: Planes that are flying are dangerous.
```

**4. Garden-path sentences**

The first reading looks correct, but fails later.

```text
The horse raced past the barn fell.
```

The parser first reads `raced` as the main verb, then fails when it reaches `fell`. The correct reading is "The horse (that was raced past the barn) fell."

**5. Headline ambiguity (very common in exams)**

```text
Teacher strikes idle kids.
```

```text
Meaning 1: A teacher hits kids who are doing nothing.
Meaning 2: A strike by teachers leaves kids without school.
```

---

## 3. Semantic Analysis

Semantic analysis determines the **literal meaning** of a sentence.

It maps words and grammar to real-world concepts. It also resolves word ambiguity and checks whether the sentence makes logical sense.

### Main tasks

- Word sense disambiguation
- Identifying relationships between words
- Understanding synonyms and antonyms
- Semantic role labeling
- Checking logical meaning

### Example 1: Word ambiguity

Input:

```text
The bank was closed.
```

The word `bank` can have two meanings:

- A financial institution
- The side of a river

In the sentence:

```text
I deposited money in the bank.
```

the word `bank` means a financial institution.

### Example 2: Logical meaning

Input:

```text
The apple ate a banana.
```

Grammatically, this sentence may look correct, but semantically it is incorrect because an apple cannot eat.

Semantic analysis detects this logical mismatch.

### Example 3: Relationships

Input:

```text
John gave Mary a book.
```

Semantic analysis identifies:

```text
John → Giver
Mary → Receiver
book → Object being given
```

Therefore, the system understands the action and the roles of the entities.

### Challenges in Semantic Analysis

| Challenge | Explanation | Example |
|---|---|---|
| Word sense disambiguation (WSD) | Choosing the correct meaning of a word from many senses | `bank`, `bat`, `light`, `match` |
| Idioms and phrases | The meaning is not the sum of the individual words | `kick the bucket` = die, `piece of cake` = very easy |
| Metaphor and figurative language | Words are used in a non-literal way | `Time is money.` / `He has a heart of stone.` |
| Need for world knowledge | The machine must know facts about the real world | `The trophy did not fit in the suitcase because it was too big.` (the machine must know that big things do not fit into smaller ones) |
| Negation | A small word can reverse the whole meaning | `The movie was not bad.` (≠ bad) |
| Synonyms and near-synonyms | Different words, same or similar meaning | `big / large / huge` |
| Domain-specific meaning | A word changes meaning in different fields | `cell` in biology, prison, phone, spreadsheet |
| Quantifier and scope problems | Words such as `every` or `some` create multiple logical readings | `Every student read a book.` |
| Semantic role difficulty | Same roles can be written in different structures | `John broke the window.` / `The window was broken by John.` |
| Low-resource languages | Few dictionaries and datasets exist | Many regional languages |

### Ambiguity in Semantic Analysis

Semantic ambiguity means a **sentence is grammatically clear, but its meaning is unclear**.

**1. Word-sense (lexical-semantic) ambiguity**

```text
He went to the bank.
```

```text
Meaning 1: He went to a financial institution.
Meaning 2: He went to the river bank.
```

```text
I like the bat.
```

```text
Meaning 1: A cricket/baseball bat.
Meaning 2: A flying animal.
```

**2. Quantifier scope ambiguity**

```text
Every student read a book.
```

```text
Meaning 1: Each student read some (possibly different) book.   → ∀ student ∃ book
Meaning 2: There is one particular book that all students read. → ∃ book ∀ student
```

**3. Idiom vs literal ambiguity**

```text
He kicked the bucket.
```

```text
Literal: He hit a bucket with his foot.
Idiom:   He died.
```

**4. Semantic role ambiguity**

```text
The chicken is ready to eat.
```

```text
Meaning 1: The chicken is hungry and ready to eat something.    → chicken = Agent
Meaning 2: The chicken is cooked and ready to be eaten.         → chicken = Patient / Object
```

**5. Vagueness**

```text
He is tall.
```

How tall? It depends on the context (tall for a child? for a basketball player?).

---

## 4. Discourse Integration

Discourse integration understands the meaning of a sentence using the **surrounding sentences**.

The meaning of a sentence often depends on the sentence before it. This stage connects ideas across a paragraph or conversation.

### Main tasks

- Coreference resolution
- Pronoun resolution
- Maintaining topic continuity
- Understanding paragraph meaning
- Handling conversations

### Example 1

Input:

```text
Seema wrote a blog. She published it yesterday.
```

Here:

```text
She → Seema
It  → the blog
```

Without the first sentence, the meaning of the second sentence would be incomplete.

### Example 2

Input:

```text
Jack is a bright student. He spends most of his time in the library.
```

Here:

```text
He → Jack
```

Discourse integration links the pronoun `He` with the noun `Jack`.

### Challenges in Discourse Integration

| Challenge | Explanation | Example |
|---|---|---|
| Pronoun resolution | Many possible nouns exist for one pronoun | `Rahul told Amit that he had won.` (he = Rahul or Amit?) |
| Long-distance references | The referred noun appears many sentences earlier | A pronoun in paragraph 5 refers to a person in paragraph 1 |
| Ellipsis | Words are left out, and must be recovered from earlier text | `Ram likes tea. So does Shyam.` (does = likes tea) |
| Implicit relations | The link between sentences is not stated | `It rained. The match was cancelled.` (cause–effect, without `because`) |
| Topic shifts | The subject of the conversation changes suddenly | `We were discussing the exam... By the way, did you eat?` |
| Multi-party conversations | Many speakers, many references | Group chats, meeting transcripts |
| Indefinite and bridging references | Something is mentioned that was not introduced directly | `I bought a car. The engine is noisy.` (the engine of that car) |
| Time and event ordering | Sequence of events must be understood | `He ate after he came back.` |
| Memory limit | The system may forget earlier context | Long chatbot conversations |
| Gender and number clues missing | Pronouns like `they` or Hindi `वह` do not show gender | `वह घर गया।` (he/she?) |

### Ambiguity in Discourse Integration

Discourse ambiguity means the **link between a sentence and the earlier text is unclear**.

**1. Anaphoric (pronoun) ambiguity**

```text
Rahul told Amit that he had won the prize.
```

```text
Meaning 1: he = Rahul (Rahul told Amit about his own win)
Meaning 2: he = Amit (Rahul told Amit about Amit's win)
```

**2. Winograd-schema ambiguity (needs common sense)**

```text
The trophy doesn't fit into the suitcase because it is too big.    → it = trophy
The trophy doesn't fit into the suitcase because it is too small.  → it = suitcase
```

Only one word changes, but the pronoun refers to a different noun.

**3. Reference to a group or a single item**

```text
Seema met Priya and Neha. They went to a mall.
```

`They` can mean Priya and Neha only, or all three people.

**4. Ellipsis ambiguity**

```text
Ravi likes Sita more than Gita.
```

```text
Meaning 1: Ravi likes Sita more than Gita likes Sita.
Meaning 2: Ravi likes Sita more than Ravi likes Gita.
```

**5. Discourse-connective ambiguity**

```text
Since he left, the office has been quiet.
```

`Since` can mean *time* ("from the time he left") or *reason* ("because he left").

---

## 5. Pragmatic Analysis

Pragmatic analysis interprets the **actual intention** behind the words.

It uses real-world context, speaker intent, tone, situation, and cultural knowledge to understand the real meaning of a sentence.

### Main tasks

- Understanding speaker intention
- Detecting sarcasm
- Identifying requests and commands
- Understanding indirect meaning
- Interpreting implied information

### Example 1: Polite request

Input:

```text
Can you open the window?
```

Literally, this asks whether the person is able to open the window.

But in normal conversation, it is usually a **polite request** to open the window.

### Example 2: Sarcasm

Input:

```text
What a great weather for a picnic!
```

If it is raining heavily, the speaker may be using sarcasm.

The literal words are positive, but the actual meaning is negative.

### Example 3: Implied meaning

Input:

```text
I am starving.
```

The literal meaning is that the person is dying from hunger.

But pragmatically, it means:

```text
I am very hungry.
```

### Challenges in Pragmatic Analysis

| Challenge | Explanation | Example |
|---|---|---|
| Sarcasm and irony | Words and intention are opposite | `Oh great, another Monday!` |
| Missing context | Text does not carry tone, facial expression, or gestures | `Fine.` (happy? angry?) |
| Indirect speech acts | A statement or question hides a command or request | `It's cold in here.` (= please close the window) |
| World and common-sense knowledge | Real-life knowledge is required | `Can you pass the salt?` is a request, not a question about ability |
| Cultural differences | Politeness and expressions differ across cultures | `We should have lunch sometime.` (may not be a real invitation) |
| Speaker intention is hidden | The real goal of the speaker is not visible | A customer says `Your service is amazing...` after a long delay |
| Humour, puns, and exaggeration | Needs creativity and background understanding | `I've told you a million times!` |
| Emotion and tone | Detecting anger, joy, or sadness | `Thanks a lot.` (grateful or annoyed?) |
| Changing context | The situation changes the meaning of the same sentence | `It's late.` (at a party vs. at an office) |
| Difficult evaluation | There is often no single correct answer, so it is hard to build datasets | Two humans may interpret the same sentence differently |

### Ambiguity in Pragmatic Analysis

Pragmatic ambiguity means **the same sentence can carry different intentions depending on the context, even when its literal meaning is clear**.

**1. Direct vs indirect speech act**

```text
Can you pass the salt?
```

```text
Literal:   A question about the listener's ability.
Intended:  A polite request – "Please pass the salt."
```

**2. Literal vs sarcastic meaning**

```text
Wow, you are so punctual!   (said to someone who came 1 hour late)
```

```text
Literal:   Praise.
Intended:  Criticism.
```

**3. Statement vs hidden request**

```text
It's very hot in here.
```

```text
Meaning 1: A simple observation about temperature.
Meaning 2: A request to switch on the fan / AC / open the window.
```

**4. Promise vs threat**

```text
I will see you later.
```

```text
Meaning 1: A friendly goodbye.
Meaning 2: A warning or threat (depending on the tone and situation).
```

**5. Information vs complaint**

```text
The food has arrived.
```

```text
Meaning 1: A simple statement (good news).
Meaning 2: A complaint – "It has come very late!"
```

---

## Summary: Ambiguity and Challenges at Each Level

| Level | Type of ambiguity | Example | Biggest challenge |
|---|---|---|---|
| **Lexical** | POS ambiguity, homonymy, morphological ambiguity, tokenization ambiguity | `book` (noun/verb), `unlockable` | Tokenization, spelling errors, slang, code-mixing |
| **Syntactic** | Structural ambiguity (PP attachment, coordination scope, garden-path) | `I saw the man with the telescope.` | Multiple parse trees, free word order, ungrammatical text |
| **Semantic** | Word-sense ambiguity, quantifier scope, idiom vs literal | `Every student read a book.` | Word sense disambiguation, idioms, world knowledge |
| **Discourse** | Anaphoric ambiguity, ellipsis, connective ambiguity | `Rahul told Amit that he won.` | Pronoun resolution, long-distance reference |
| **Pragmatic** | Direct vs indirect meaning, sarcasm, hidden intent | `Can you open the window?` | Sarcasm, tone, culture, speaker intention |

> **Key idea:** As we move from lexical to pragmatic, the analysis goes from **word level → sentence structure → meaning → context → intention**, and the ambiguity becomes **harder to solve** because more and more world knowledge is required.

---

## Modern NLP Pipeline

In practical NLP systems, the theoretical stages are implemented using a modern pipeline.

| Stage | Purpose | Example |
|---|---|---|
| Text ingestion | Collect input from chat, email, review, or speech | A customer review is received |
| Preprocessing | Clean and normalize the text | Lowercase conversion, punctuation removal, tokenization, lemmatization |
| Syntactic and semantic analysis | Understand grammar and meaning | POS tagging, parsing, named entity recognition |
| Model inference | Perform the required NLP task | Classify a review as positive or negative |
| Output generation | Produce the final result | Return sentiment label or generate a response |

### Example: Sentiment analysis

Input:

```text
The phone camera is amazing, but the battery life is disappointing.
```

### Step 1: Preprocessing

```text
["phone", "camera", "amazing", "battery", "life", "disappointing"]
```

### Step 2: Lexical analysis

```text
amazing       → Adjective
disappointing → Adjective
```

### Step 3: Syntactic analysis

The sentence has two clauses joined by `but`.

```text
Clause 1: The phone camera is amazing
Clause 2: The battery life is disappointing
```

### Step 4: Semantic analysis

```text
amazing       → Positive word
disappointing → Negative word
```

### Step 5: Discourse integration

The word `but` indicates contrast between the two opinions.

```text
Camera is good
but
Battery life is bad
```

### Step 6: Pragmatic analysis

The user appreciates the camera but is unhappy with the battery performance.

### Output

```text
Mixed / Negative sentiment
```

The camera is praised, but the battery life is criticized.

---

## Examples in Detail

### Example 1: Chatbot

Input:

```text
Can you book a flight from Mumbai to Delhi tomorrow?
```

### Lexical analysis

Tokens:

```text
["Can", "you", "book", "a", "flight", "from", "Mumbai", "to", "Delhi", "tomorrow", "?"]
```

### Syntactic analysis

```text
Can you book a flight → Request structure
from Mumbai to Delhi  → Route information
tomorrow              → Time information
```

### Semantic analysis

```text
Action       → Book
Object       → Flight
Source       → Mumbai
Destination  → Delhi
Date         → Tomorrow
```

### Discourse integration

If the user previously said:

```text
I want an early morning flight.
```

Then the chatbot should remember this preference while booking.

### Pragmatic analysis

The user is not asking whether the chatbot has the ability to book a flight.

The real intention is:

```text
Please book a flight from Mumbai to Delhi tomorrow.
```

### Output

```text
Flight booking request created.
```

### Example 2: Machine translation

Input:

```text
I am very hungry.
```

Target language: Hindi.

### Lexical analysis

```text
I      → Pronoun
am     → Auxiliary verb
very   → Adverb
hungry → Adjective
```

### Syntactic analysis

```text
Subject → I
Verb    → am
Adverb  → very
Adjective → hungry
```

### Semantic analysis

```text
Meaning → The speaker feels strong hunger.
```

### Pragmatic analysis

The sentence does not literally mean that the speaker is dying.

It means:

```text
The speaker wants food.
```

### Output

```text
मुझे बहुत भूख लगी है।
```

### Example 3: Named Entity Recognition

Input:

```text
Apple opened a new office in Mumbai on 15 January 2026.
```

### Lexical analysis

Tokens:

```text
["Apple", "opened", "a", "new", "office", "in", "Mumbai", "on", "15", "January", "2026", "."]
```

### Syntactic analysis

```text
Apple → Subject
opened → Verb
a new office → Object
in Mumbai → Location
on 15 January 2026 → Time
```

### Semantic analysis

```text
Apple → Company
Mumbai → City
15 January 2026 → Date
```

### Output

```text
Organization: Apple
Location: Mumbai
Date: 15 January 2026
```

### Example 4: Text summarization

Input:

```text
Rahul ordered a pizza. It was delivered late. The pizza was cold when it arrived. Rahul was unhappy with the service.
```

### Lexical analysis

The system identifies important words:

```text
Rahul, pizza, delivered, late, cold, unhappy, service
```

### Syntactic analysis

The system identifies the subject, verb, and object in each sentence.

```text
Rahul ordered a pizza.
It was delivered late.
The pizza was cold.
Rahul was unhappy.
```

### Semantic analysis

```text
delivered late → Negative
cold pizza     → Negative
unhappy        → Negative
```

### Discourse integration

```text
It → pizza
```

### Pragmatic analysis

The customer is complaining about poor delivery service and food quality.

### Output

```text
Rahul was unhappy because his pizza was delivered late and cold.
```

### Example 5: Question answering

Input:

```text
Who wrote the blog?
```

Context:

```text
Seema wrote a blog. She published it yesterday.
```

### Lexical analysis

```text
Who → Question word
wrote → Verb
the blog → Object
```

### Syntactic analysis

```text
Who → Subject being asked
wrote → Action
the blog → Object
```

### Semantic analysis

```text
Question type → Person
Expected answer → Name of the writer
```

### Discourse integration

From the context:

```text
Seema wrote a blog.
```

The system identifies:

```text
Writer → Seema
```

### Output

```text
Seema wrote the blog.
```

---

## Applications of NLP

NLP is used wherever computers need to work with human language. The main applications are:

### 1. Chatbots and virtual assistants

- Customer-support bots on websites and apps
- Voice assistants such as Alexa, Siri, and Google Assistant
- Understand user questions and give a suitable reply or action

### 2. Machine translation

- Converts text or speech from one language to another
- Example: Google Translate, translating English to Hindi
- Useful for communication across languages

### 3. Sentiment analysis (opinion mining)

- Finds whether text is positive, negative, or neutral
- Used for product reviews, social media monitoring, and brand reputation
- Example: `The camera is amazing but battery is poor` → Mixed sentiment

### 4. Search engines and information retrieval

- Understands the meaning of the search query, not only keywords
- Example: Google search, e-commerce product search, voice search
- Gives auto-suggestions and "did you mean...?" corrections

### 5. Text summarization

- Converts long documents into short summaries
- Used for news articles, research papers, legal documents, and meeting notes

### 6. Question answering systems

- Gives direct answers to questions instead of a list of documents
- Example: `Who wrote the blog?` → `Seema`
- Used in help desks, education, and search engines

### 7. Speech recognition and text-to-speech

- Converts speech into text (voice typing, subtitles, call transcription)
- Converts text into speech (screen readers, navigation voice)

### 8. Named Entity Recognition and information extraction

- Finds names of people, organizations, places, dates, and amounts in text
- Used for resume parsing, news analysis, and building databases from documents

### 9. Spam detection and email filtering

- Classifies emails and messages as spam or not spam
- Also sorts emails into categories such as Primary, Promotions, and Social

### 10. Text classification and topic modelling

- Puts documents into categories (sports, politics, finance)
- Used for news tagging, ticket routing, and document organization

### 11. Grammar checking, spell checking, and autocomplete

- Tools such as Grammarly and MS Word spell-check
- Next-word prediction on mobile keyboards and Gmail Smart Compose

### 12. Content moderation

- Detects hate speech, abusive language, and fake news on social media
- Helps make online platforms safer

### 13. Text generation and writing assistance

- Writes emails, articles, code, and stories using language models (for example, ChatGPT and Claude)
- Used for content creation, report writing, and paraphrasing

### 14. Domain-specific applications

| Domain              | Use of NLP                                                                              |
| ------------------- | --------------------------------------------------------------------------------------- |
| **Healthcare**      | Extracting information from patient records and doctor's notes, medical chatbots        |
| **Legal**           | Contract analysis, legal document search and summarization                              |
| **Finance**         | News and market sentiment analysis, fraud detection, automated report reading           |
| **Education**       | Automatic essay grading, language-learning apps, intelligent tutors                     |
| **Human Resources** | Resume screening, matching candidates to jobs                                           |
| **E-commerce**      | Product recommendations, review analysis, customer-query bots                           |
| **Accessibility**   | Speech-to-text for hearing-impaired people, text-to-speech for visually impaired people |

### Application vs NLP stage used (quick view)

| Application | Most important NLP stages |
|---|---|
| Spell checker | Lexical |
| Grammar checker | Lexical + Syntactic |
| Machine translation | Lexical + Syntactic + Semantic + Pragmatic |
| Information extraction / NER | Lexical + Syntactic + Semantic |
| Text summarization | Semantic + Discourse |
| Question answering | Semantic + Discourse |
| Chatbots / voice assistants | All five stages |
| Sarcasm / intent detection | Pragmatic |

---

## Quick Revision

- **NLP** = making machines understand, interpret, and generate human language.
- **Five stages:** Lexical → Syntactic → Semantic → Discourse → Pragmatic.
- **Lexical:** tokens, morphology, POS. Ambiguity: `book` (noun/verb).
- **Syntactic:** grammar and parse tree. Ambiguity: `I saw the man with the telescope.`
- **Semantic:** literal meaning. Ambiguity: `bank` (money / river).
- **Discourse:** links to surrounding sentences. Ambiguity: `Rahul told Amit that he won.`
- **Pragmatic:** real intention. Ambiguity: `Can you open the window?`
- **Applications:** chatbots, translation, sentiment analysis, search engines, summarization, question answering, speech recognition, spam detection, NER, content moderation, and text generation.
