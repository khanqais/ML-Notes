# Porter Stemming Algorithm

## Definition
Porter Stemming is a rule-based algorithm used in NLP to reduce English words to their root/stem form by removing common suffixes.

Example:  
- running → run  
- connected → connect  
- studies → studi  

It was developed by Martin Porter in 1980.

## Prerequisites

### 1. Vowels and Consonants
- Vowels: a, e, i, o, u
- Consonants: all other letters
- `y` is a vowel if the previous letter is a consonant.  
  Example: In `happy`, `y` is a vowel.

### 2. Word Form
Every word can be written as:

`[C](VC)^m[V]`

Where:
- `C` = consonant group
- `V` = vowel group
- `m` = measure

### 3. Measure (m)
`m` is the number of `VC` groups in a word.

Examples:
- `tree` → m = 0
- `trouble` → m = 1
- `private` → m = 2

The measure helps decide whether a suffix can be removed.

## Step 1a: Plural Removal

Rules:
- `sses → ss`
- `ies → i`
- `ss → ss`
- `s → ""`

Examples:
- caresses → caress
- ponies → poni
- cats → cat

## Step 1b: Remove -ed and -ing

Rules:
- If `m > 0`: `eed → ee`
- If stem contains a vowel:
  - `ed → ""`
  - `ing → ""`

Examples:
- agreed → agree
- plastered → plaster
- running → runn → run

After removing `ed` or `ing`:
- `at → ate`
- `bl → ble`
- `iz → ize`

Examples:
- conflated → conflate
- troubled → trouble

## Step 1c: Replace y with i

Rule:
- If stem contains a vowel: `y → i`

Examples:
- happy → happi
- enjoy → enjoi
- sky → sky (no change, because stem has no vowel)

## Step 2: Derivational Suffixes

Important rules:
- `ational → ate`
- `tional → tion`
- `izer → ize`
- `fulness → ful`
- `ousness → ous`
- `biliti → ble`

Examples:
- relational → relate
- conditional → condition
- hopefulness → hopeful

## Step 3: Remove More Suffixes

Important rules:
- `icate → ic`
- `ative → ""`
- `alize → al`
- `ical → ic`
- `ful → ""`
- `ness → ""`

Examples:
- hopeful → hope
- goodness → good
- electrical → electric

## Step 4: Remove Remaining Suffixes

Most suffixes are removed only if `m > 1`.

Important rules:
- `al → ""`
- `ance → ""`
- `ence → ""`
- `er → ""`
- `ment → ""`
- `ion → ""` only if stem ends with `s` or `t`
- `ate → ""`
- `ous → ""`
- `ive → ""`

Examples:
- revival → reviv
- adjustment → adjust
- adoption → adopt

## Step 5: Final Cleanup

### Step 5a: Remove final e
- If `m > 1`: remove final `e`
- If `m = 1` and stem does not end in CVC: remove final `e`

Examples:
- probate → probat
- rate → rate

### Step 5b: Remove double l
- If `m > 1` and word ends with `ll`: remove one `l`

Example:
- controll → control

## Example
`relational`

Step 2: `ational → ate`  
So, `relational → relate`

Step 5a: remove final `e`  
Final stem: `relat`

## Advantages
- Simple and fast
- Useful for search engines
- Reduces different forms of a word to one stem

## Disadvantages
- Output may not be a real word
- Example: `studies → studi`
- Less accurate than lemmatization