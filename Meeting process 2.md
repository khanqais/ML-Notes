# VoiceSense Meeting Processing Pipeline: Complete End-to-End Technical Specification

This document provides a comprehensive, code-verified technical breakdown of the **entire Meeting Processing Pipeline** in the VoiceSense backend, covering **all stages from initial audio ingestion through Google Gemini AI analysis and final database persistence**.

All specifications, algorithms, parameter matrices, mathematical formulas, prompt structures, and design decisions documented herein are extracted directly from the actual implementation in `src/api/`.

---

## 1. High-Level Architecture & End-to-End Data Flow

The VoiceSense meeting processing pipeline operates as an asynchronous, multi-layered processing workflow designed for noisy, conversational, bilingual (Hindi/English/Hinglish) audio recordings.

```
Incoming Audio (M4A / WAV / MP3 / WebM / OGG up to 200MB)
                           │
                           ▼
  [HTTP Controller] POST /api/meetings/upload
    ├── Temporary File Storage (express-fileupload)
    ├── User & Language Override Validation
    ├── Initial DB Record: Meeting.status = 'uploaded'
    └── Immediate HTTP 201 Accepted Response
                           │
                           ▼ (setImmediate async background worker)
  ┌─────────────────────────────────────────────────────────────┐
  │ Step 1: Layer 1 — Audio Standardization (FFmpeg)            │
  │   - 16,000 Hz, 1 Channel Mono, 16-bit PCM (pcm_s16le) WAV   │
  │   - No DSP filters (preserves acoustic embedding baseline)  │
  │   - DB Update: Meeting.processedAudioUrl, durationSeconds   │
  └──────────────────────────────┬──────────────────────────────┘
                                 │
                                 ▼
  ┌─────────────────────────────────────────────────────────────┐
  │ Step 2: Layer 2 — Speech-to-Text & Diarization (Deepgram)   │
  │   - Language Model Matrix: Nova-3 multi / Nova-2 hi / en-IN │
  │   - Configurable Domain Vocabulary Boosting (keyterm)       │
  │   - Utterance tokenization with word-level timestamps       │
  │   - Snapshot raw Deepgram speaker labels                    │
  └──────────────────────────────┬──────────────────────────────┘
                                 │
                                 ▼
  ┌─────────────────────────────────────────────────────────────┐
  │ Step 3: Layer 2.1 — Acoustic Re-Diarization (ECAPA-TDNN)    │
  │   - ONNX Runtime 192-dimensional speaker embeddings         │
  │   - Agglomerative Hierarchical Clustering (average-linkage) │
  │   - Centroid matching vs enrolled user VoiceProfiles        │
  │   - Fixes Deepgram speaker collapse & assigns Person 1..N   │
  └──────────────────────────────┬──────────────────────────────┘
                                 │
                                 ▼
  ┌─────────────────────────────────────────────────────────────┐
  │ Step 4: Layer 2.3 — Phonetic & Name Refinement (Gemini AI)  │
  │   - Algorithmic Consonant Skeleton & Levenshtein matching   │
  │   - Uncertainty extraction (confidence < 0.72)              │
  │   - Gemini prompt-based phonetic error correction           │
  │   - Anti-fabrication guard: reverts hallucinated names      │
  │   - Deterministic utterance-initial name-call correction    │
  └──────────────────────────────┬──────────────────────────────┘
                                 │
                                 ▼
  ┌─────────────────────────────────────────────────────────────┐
  │ Step 5: Layer 2.4 — Transcript Formatting & DB Storage      │
  │   - Speaker ID mapping (Enrolled User vs Unknown Person N)  │
  │   - UnknownSpeaker centroids & metrics stored in DB         │
  │   - Transcripts persisted with timestamps and similarity    │
  └──────────────────────────────┬──────────────────────────────┘
                                 │
                                 ▼
  ┌─────────────────────────────────────────────────────────────┐
  │ Step 6: Layer 2.5 — System 1 Decision Extraction (Laya)     │
  │   - Zero-dependency multilingual regex dialogue act parsing │
  │   - Fast proposal-affirmation consensus extraction          │
  │   - Produces pre-verified ground-truth decisions            │
  └──────────────────────────────┬──────────────────────────────┘
                                 │
                                 ▼
  ┌─────────────────────────────────────────────────────────────┐
  │ Step 7: Layer 3 — Conversational Intelligence (Gemini AI)   │
  │   - Full transcript analysis grounded in System 1 decisions │
  │   - Structured JSON generation (gemini-3.5-flash-lite)      │
  │   - Executive Summary, Key Points, Action Items, Events,    │
  │     Participants, Sentiment, Topics, Decisions              │
  └──────────────────────────────┬──────────────────────────────┘
                                 │
                                 ▼
  ┌─────────────────────────────────────────────────────────────┐
  │ Step 8: Layer 4 — Deduplication & Persistence Finalization  │
  │   - Reconciles & deduplicates Laya vs Gemini decisions      │
  │   - ISO 8601 Event scheduling parsing & storage             │
  │   - Summary, key points & action items DB persistence       │
  │   - Status update: Meeting.status = 'completed'             │
  │   - Execution timing metrics aggregation                    │
  └─────────────────────────────────────────────────────────────┘
```

---

## 2. Ingestion & Pre-Pipeline Orchestration

- **Controller**: `src/api/controllers/meetingController.ts` (`uploadMeeting`)
- **Orchestrator**: `src/api/services/meetingProcessingService.ts` (`processMeeting`)
- **Database Service**: `src/api/services/databaseService.ts`

### 2.1 HTTP Request Handling (`POST /api/meetings/upload`)
1. **Multipart Upload**: Managed by `express-fileupload` (`useTempFiles: true`, `tempFileDir: ./uploads/temp`, max size `200MB`).
2. **User Authentication & Validation**: Validates `userId` against `databaseService.getUserById(userId)`.
3. **Language Selection Validation**: Accepts optional `language` in request body. Validates and normalizes to:
   - `'english'` / `'en'`
   - `'hindi'` / `'hi'`
   - `'bilingual'` (Default)
   - Rejects unapproved values with HTTP `400 Bad Request`.
4. **Filesystem Relocation**: Relocates uploaded temporary file into persistent audio store:
   `./uploads/audio/meeting_${userId}_${Date.now()}${extension}`.
5. **Initial Database Record**:
   `databaseService.createMeeting(userId, title, audioFileUrl, description, fileSize)` sets status to `'uploaded'`.
6. **Asynchronous Non-Blocking Dispatch**:
   Returns immediate HTTP `201 Accepted` with meeting ID. Dispatches background execution using Node.js `setImmediate()`:
   ```typescript
   setImmediate(async () => {
     try {
       await meetingProcessingService.processMeeting(meeting.id, filePath, userId, languageOverride);
     } catch (err) {
       console.error(`[MeetingController] Background processing failed for meeting ${meeting.id}:`, err);
     }
   });
   ```

### 2.2 Pipeline Initialization (`processMeeting`)
1. **Status Transition**: Updates `Meeting.status` to `'processing'`.
2. **Timing Registry**: Initializes high-precision timers using `performance.now()` across:
   - `audioProcessing`
   - `deepgramSTT`
   - `ecapaReDiarization`
   - `geminiPhoneticRefinement`
   - `layaSystem1`
   - `geminiLLMAnalysis`
   - `databasePersistence`
   - `total`
3. **Language Resolution Hierarchy**:
   ```typescript
   const selectedLanguage =
     requestedLanguage ||
     (userProfiles && userProfiles.length > 0 ? (userProfiles[0] as any)?.language : undefined) ||
     'bilingual';
   ```

---

## 3. Step 1: Layer 1 — Audio Standardization & Preprocessing

- **Source File**: `src/api/services/audioProcessingService.ts` (`processAudio`)
- **Underlying Engines**: `fluent-ffmpeg`, `@ffmpeg-installer/ffmpeg`, `@ffprobe-installer/ffprobe`

### 3.1 Objective & Rationale
Convert diverse, arbitrary input formats (AAC in M4A, Opus in WebM, MP3, OGG) into an uncompressed, uniform acoustic baseline: **16,000 Hz, 1-channel Mono, 16-bit Linear PCM Little-Endian WAV**.

### 3.2 Detailed Conversion Specification

```typescript
ffmpeg(inputPath)
  .audioFrequency(16000)
  .audioChannels(1)
  .audioCodec('pcm_s16le')
  .format('wav')
  .output(outputPath)
```

| Parameter | Exact Value | Code Justification & Acoustic Purpose |
| :--- | :--- | :--- |
| **Container** | `'wav'` (RIFF Header) | Header format easily parsed by both native C++ libraries (ONNX) and remote APIs. |
| **Codec** | `'pcm_s16le'` | Raw 16-bit signed integer linear PCM. Avoids lossy psychoacoustic compression. |
| **Sample Rate** | `16000` (16 kHz) | Exactly matches Deepgram's internal model frequency and ECAPA-TDNN 80-channel filterbank extraction. |
| **Channels** | `1` (Mono) | Eliminates stereo panning phase issues, ensuring uniform speaker energy. |

### 3.3 Critical Architectural Decisions Documented in Code
1. **Linear PCM vs Opus/OGG**:
   - `audioProcessingService.ts` lines 44–46: Linear PCM preserves high-frequency spectral components (4kHz–8kHz) essential for Indian language consonant discrimination (e.g., distinguishing Hindi dental vs retroflex consonants: श/स, ट/त, ड/द).
2. **Zero DSP Cleanup Filters (No Denoise, No Normalization)**:
   - `audioProcessingService.ts` lines 47–53: Deliberately omitted. The exact standardized audio signal is fed directly into ECAPA-TDNN neural embedding models. Denoising (`arnndn`, `afftdn`) or non-linear dynamic range compression alter spectral formant envelopes and introduce non-linear phase distortion, which causes severe acoustic clustering drift and misattribution.
3. **Integrity Guard**:
   - If converted output is `<= 500` bytes (corrupted/truncated transcode), the pipeline safely falls back to the original input file.

### 3.4 Database Update
Persists standardized audio file reference:
```typescript
await databaseService.updateMeeting(meetingId, {
  processedAudioUrl: audioResult.processedFilePath,
  durationSeconds: audioResult.duration,
});
```

---

## 4. Step 2: Layer 2 — Speech-to-Text with Speaker Diarization

- **Source File**: `src/api/services/deepgramService.ts` (`transcribeWithDiarization`)
- **Underlying Engine**: `@deepgram/sdk` (v3.4.0)

### 4.1 Input Validation & Vocabulary Setup
1. Validates audio buffer: `if (audioBuffer.length < 500) throw new Error(...)`.
2. Extracts domain vocabulary from environment variable `process.env.DEEPGRAM_KEYTERMS`.
3. Sets up base Deepgram configuration:
   - `smart_format: true` (formats dates, numerals, currencies).
   - `punctuate: true` (inserts punctuation).
   - `diarize: true` (enables initial turn detection).
   - `utterances: true` (groups words into conversational turns).
   - `paragraphs: false` (retains raw conversational turn boundaries).
   - `filler_words: false` (filters speech disfluencies).

### 4.2 Language Model Selection & Fallback Hierarchy

The system attempts primary models and automatically cascades to fallbacks on rejection:

```
┌─────────────────┬──────────────────────────────────┬─────────────────────────────────┐
│ Language        │ Primary Configuration            │ Fallback Configuration          │
├─────────────────┼──────────────────────────────────┼─────────────────────────────────┤
│ english / en    │ model: 'nova-3', lang: 'en-IN',  │ model: 'nova-2', lang: 'en-IN', │
│                 │ keyterm: configuredKeyterms      │ keywords: ["term:3"]            │
├─────────────────┼──────────────────────────────────┼─────────────────────────────────┤
│ hindi / hi      │ model: 'nova-2', lang: 'hi',     │ (none - deterministic)          │
│                 │ keywords: ["term:3"]             │                                 │
├─────────────────┼──────────────────────────────────┼─────────────────────────────────┤
│ bilingual       │ model: 'nova-3', lang: 'multi',  │ model: 'nova-2', lang: 'hi',    │
│ (Default)       │ keyterm: configuredKeyterms      │ keywords: ["term:3"]            │
└─────────────────┴──────────────────────────────────┴─────────────────────────────────┘
```

> **Key Code Rule (`deepgramService.ts` lines 86–92)**:
> In bilingual mode, `language: 'multi'` performs **per-utterance** language evaluation. Deepgram's `detect_language` parameter is explicitly forbidden because it calculates a single global language across the entire audio file, breaking code-switched dialogues. Furthermore, Nova-3 expects unweighted `keyterm: string[]`, whereas Nova-2 requires weighted `keywords: string[]` (e.g., `["term:3"]`).

### 4.3 Utterance Extraction & Speaker Label Snapshot
1. Iterates over `result.results.utterances`.
2. Extracts word-level timestamps (`start`, `end`, `confidence`, `speaker`).
3. Saves original Deepgram speaker labels before downstream re-diarization:
   ```typescript
   for (const seg of transcriptionResult.segments) {
     seg.deepgramSpeakerLabel = seg.speaker;
   }
   ```

---

## 5. Step 3: Layer 2.1 — Acoustic Speaker Re-Diarization (ECAPA-TDNN)

- **Source Files**:
  - `src/api/services/voiceRecognitionService.ts` (`reDiarizeSegments`)
  - `src/api/services/ecapaTdnnService.ts` (`generateEcapaEmbedding`)
- **Underlying Engine**: `onnxruntime-node` running SpeechBrain ECAPA-TDNN

### 5.1 Objective
Deepgram diarization frequently suffers from **speaker collapse** (merging multiple distinct voices into a single speaker label when accents or recording conditions are similar). Layer 2.1 overrides Deepgram's speaker assignments using local acoustic neural embeddings.

### 5.2 Mathematical & DSP Feature Extraction
1. **Mel Filterbank Extraction**:
   - 16 kHz audio sliced to exact utterance timestamps `[startTime, endTime]`.
   - Windowing: 25ms frame length, 10ms frame shift (160 samples hop).
   - 80 Mel frequency filterbank bins spanning 20 Hz to 8,000 Hz.
2. **ECAPA-TDNN ONNX Inference**:
   - Passes 80-channel Mel features through `ecapa_tdnn.onnx`.
   - Generates a **192-dimensional acoustic embedding vector**.
   - Applies L2 Normalization:
     $$\mathbf{e} = \frac{\mathbf{v}}{\|\mathbf{v}\|_2}$$

### 5.3 Clustering & Attribution Pipeline
1. **Filtering**: Segments $< 1.5\text{s}$ skip direct embedding (insufficient acoustic information); segments $\le 2.0\text{s}$ are held for acoustic inheritance.
2. **Agglomerative Hierarchical Clustering (AHC)**:
   - Groups segments using average-linkage cosine similarity.
   - Merging threshold: `CLUSTER_MERGE_THRESHOLD = 0.50` (adaptive floor `0.35`).
3. **Duration-Weighted Cluster Centroid**:
   $$\mathbf{c}_k = \frac{\sum_{i \in C_k} d_i \mathbf{e}_i}{\|\sum_{i \in C_k} d_i \mathbf{e}_i\|_2}$$
4. **Enrolled Profile Matching**:
   - Compares cluster centroids against enrolled user voice embeddings in PostgreSQL.
   - Enforces a multi-level matching gate:
     - Centroid similarity $\ge \text{CENTROID\_MATCH\_THRESHOLD } (0.55)$.
     - Quorum requirement: at least $50\%$ (`PROFILE_QUORUM_FRAC`) of cluster utterances must individually exceed similarity floor $0.45$ (`PROFILE_MEMBER_THRESHOLD`).
5. **Speaker Number Assignment**:
   - Enrolled User: Assigned index `0` (or matching profile index).
   - Unknown Speakers: Numbered sequentially (`Person 1`, `Person 2`, ...).
   - Short segments inherit the speaker label of the acoustically nearest assigned segment if similarity $\ge 0.30$.

---

## 6. Step 4: Layer 2.3 — Phonetic & Name Refinement with Google Gemini

- **Source File**: `src/api/services/geminiService.ts` (`refineTranscriptionSegments`)
- **Integration Caller**: `src/api/services/meetingProcessingService.ts` (lines 107–133)
- **Model Client**: `@google/genai` (v0.1.1+)
- **Active Model**: `gemini-3.5-flash-lite` (configurable via `process.env.GEMINI_MODEL`)

### 6.1 The Core Problem & Hybrid Design
ASR engines (including Deepgram Nova-3) struggle with:
1. **Indian-accented English technical terms** (e.g., *EchoSense* transcribed as *egoscence*, *scheduler* transcribed without *silent*).
2. **Proper Indian names** (e.g., *Khizar* transcribed as *Kashif* or *Kaisar*).
3. **Hinglish mid-sentence code-switching** (words randomly swapped between Devanagari and Latin script).

However, pure LLM rewriting is inherently dangerous: LLMs tend to invent names, fix colloquial grammar, or hallucinate words from conversational context.

**VoiceSense's Solution**: A strict, three-tier hybrid architecture:
1. **Deterministic Phonetic Pre-Analysis (TypeScript)**.
2. **Context-Bounded Constrained Refinement (Gemini)**.
3. **Deterministic Algorithmic Post-Guards (TypeScript)**.

```
Raw Diarized Segments
         │
         ├── 1. Find Low-Confidence Words (< 0.72)
         ├── 2. Compute Consonant Skeletons & Weighted Levenshtein vs Known Names
         │
         ▼
Gemini LLM Refinement Prompt
  (Injected: uncertain_words, possible_name_tokens, known_names, domain_vocab)
  (Enforced: Edit scope restriction, script consistency, zero grammar fixes)
         │
         ▼
Raw Gemini Refined Array
         │
         ├── 3. Array Length & Character Ratio Guard (50% <= len <= 200% + 20)
         ├── 4. Anti-Fabrication Guard (revertInsertedNames)
         └── 5. Deterministic Name-Call Correction (applyDeterministicNameCorrections)
         │
         ▼
Phonetically Perfected Segments
```

### 6.2 Deterministic Phonetic Name Matching Algorithm
Implemented in `src/api/services/geminiService.ts` lines 26–161:

#### A. Devanagari Transliteration & Nukta Normalization
Normalizes Urdu/Persian loan consonant dots (Nukta) before mapping characters to Latin equivalents:
```typescript
const NUKTA_MAP: Array<[RegExp, string]> = [
  [/क\u093C/g, 'q'], [/ख\u093C/g, 'kh'], [/ग\u093C/g, 'g'],
  [/ज\u093C/g, 'z'], [/ड\u093C/g, 'r'],  [/ढ\u093C/g, 'rh'],
  [/फ\u093C/g, 'f'], [/य\u093C/g, 'y'],
];
```
Full character mapping (`DEVANAGARI_TRANSLIT`) maps all independent vowels (अ, आ...), consonants (क, ख...), and matras (ा, ि...).

#### B. Consonant Skeleton Extraction
The phonetic core of each token is extracted:
1. Converts Devanagari to Latin representation.
2. Strips all non-alphabetical characters.
3. Collapses multi-character digraphs into single phonetic units:
   - `chh` $\rightarrow$ `c`, `sh` $\rightarrow$ `s`, `kh` $\rightarrow$ `k`, `gh` $\rightarrow$ `g`
   - `ch` $\rightarrow$ `c`, `th` $\rightarrow$ `t`, `dh` $\rightarrow$ `d`, `ph` $\rightarrow$ `f`, `bh` $\rightarrow$ `b`
4. Normalizes hard `c` to `k` (e.g., English *cash* aligns with *Qais*).
5. Discards all vowels (`[aeiou]`) as they carry minimal discriminative weight in Indic name recognition.

#### C. Weighted Levenshtein with Acoustic Confusion Cost Matrix
ASR confusion groups (sounds easily swapped by speech models) are penalized at half cost ($0.5$ instead of $1.0$):
```typescript
const CONFUSION_GROUPS: string[][] = [
  ['z', 'j'], ['k', 'q'], ['b', 'v'], ['s', 'z'],
];

function substitutionCost(a: string, b: string): number {
  if (a === b) return 0;
  for (const grp of CONFUSION_GROUPS) {
    if (grp.includes(a) && grp.includes(b)) return 0.5;
  }
  return 1;
}
```

The normalized similarity score is computed:
$$\text{Sim}(a, b) = 1 - \frac{\text{Levenshtein}(a, b)}{\max(|a|, |b|)}$$

#### D. Name Candidate Filtering (`findNameCandidates`)
To be considered a candidate match:
1. First consonant sound must match: `substitutionCost(sk[0], nameSk[0]) < 1`.
2. Consonant similarity must exceed threshold: $\text{Sim} \ge 0.60$ (`NAME_MATCH_THRESHOLD`).
3. Disambiguation Margin: $\text{Sim}_{\text{best}} - \text{Sim}_{\text{second}} \ge 0.10$.

### 6.3 Prompt Construction & Gemini Instructions
`geminiService.refineTranscriptionSegments` structures an exhaustive JSON prompt:
- **Payload**:
  - `index`: Segment order index.
  - `text`: Utterance text.
  - `uncertain_words`: Low-confidence words ($\text{confidence} < 0.72$).
  - `possible_name_tokens`: Pre-calculated candidate hints from phonetic analyzer.
- **Injected Context**:
  - `KNOWN_PARTICIPANT_NAMES`: Derived from active `VoiceProfile` records in PostgreSQL + `process.env.KNOWN_PARTICIPANT_NAMES`.
  - `DOMAIN_VOCABULARY`: Derived from `process.env.DEEPGRAM_KEYTERMS`.
- **Strict Guardrail Rules Injected into Prompt**:
  1. **Edit Scope**: Change ONLY tokens appearing in `uncertain_words`, `possible_name_tokens`, or domain vocabulary sound-alikes. All other words MUST be copied verbatim.
  2. **No Rewriting or Summarizing**: Never fix grammar, alter dialect, or rephrase sentences. Output must remain a verbatim spoken transcript.
  3. **Script Consistency**: Keep words in the script they were spoken (Devanagari for Hindi words, Latin for English technical terms).
  4. **Strict Name Rule**: Match names strictly by sound. Never assign or insert a name based on conversational context (e.g., who replied or who owns the task).

### 6.4 Gemini Execution & Transient Retry Loop
Gemini API call execution includes built-in retry handling:
```typescript
const maxAttempts = 3;
for (let attempt = 1; attempt <= maxAttempts; attempt++) {
  try {
    const response = await ai.models.generateContent({
      model: modelId(),
      contents: prompt,
      config: { responseMimeType: 'application/json' },
    });
    return response.text ?? null;
  } catch (err: any) {
    const status = /503|429|UNAVAILABLE|overloaded/i.test(err?.message || '') ? 'transient' : 'fatal';
    if (attempt < maxAttempts && status === 'transient') {
      await new Promise((r) => setTimeout(r, 1500 * attempt));
    } else {
      throw err;
    }
  }
}
```

### 6.5 Post-Gemini Algorithmic Safety Guards

#### Guard 1: Segment Character Length Ratio Check
A candidate correction is accepted only if:
$$\text{length}(\text{candidate}) \ge \max(1, \lfloor \text{length}(\text{original}) \times 0.5 \rfloor)$$
$$\text{length}(\text{candidate}) \le \lceil \text{length}(\text{original}) \times 2.0 + 20 \rceil$$
Prevents hallucinations or accidental truncation from degrading the transcript.

#### Guard 2: Anti-Fabrication Name Guard (`revertInsertedNames`)
If a known participant name appears in the refined text, but was **not** present in the original text, the engine checks whether *any* token in the original utterance phonetically resembles the name:
```typescript
const hasSoundAlike = originalSkeletons.some(
  (sk) => skeletonSimilarity(sk, consonantSkeleton(name)) >= NAME_MATCH_THRESHOLD
);
if (!hasSoundAlike) {
  // Revert entire utterance to original unrefined text
  segments[i].text = original;
}
```
If no original token sounds like the name, Gemini hallucinated it from context, and the segment is immediately reverted.

#### Guard 3: Deterministic Name-Call Correction (`applyDeterministicNameCorrections`)
If an utterance begins with a token that phonetically matches a known name ($\text{Sim} \ge 0.60$), but Gemini either left the token uncorrected or substituted the wrong name, TypeScript deterministically overwrites the initial token with the correct known name.

---

## 7. Step 5: Layer 2.4 — Speaker Attribution & Transcript Persistence

- **Source File**: `src/api/services/meetingProcessingService.ts` (`processTranscriptionSegments`)
- **Database Service**: `src/api/services/databaseService.ts`

### 7.1 Speaker Name Mapping Logic
1. Distinct speaker indices from segments are sorted:
   - Index `< voiceProfiles.length`: Enrolled profile user (`displayName` or `user.name`).
   - Unmatched Index: Labeled as `Person ${spkNum}`.
   - Index `-1`: Labeled as `Unknown`.

### 7.2 Database Persistence Operations
1. **Unknown Speakers (`UnknownSpeaker` Table)**:
   For every non-enrolled speaker, aggregates:
   - `speakerLabel`: (e.g., `Person 1`).
   - `voiceEmbedding`: 192-dimensional cluster centroid vector from `voiceRecognitionService.getSpeakerCentroid(spkNum)`.
   - `utteranceCount` & `totalDuration`.
   Saved via `databaseService.createUnknownSpeaker()`.
2. **Utterance Transcripts (`Transcript` Table)**:
   Batched insert via `databaseService.createTranscripts()` storing:
   - `meetingId`, `speakerId` (UUID if enrolled, otherwise null).
   - `speakerType`: `'known'` | `'unknown'`.
   - `speakerName`: (e.g., `Ayaz` or `Person 1`).
   - `deepgramSpeakerLabel`: Preserved raw Deepgram integer.
   - `similarityScore`: Cosine similarity to enrolled voice profile.
   - `text`: Phonetically refined transcript utterance.
   - `startTime`, `endTime`, `confidence`.

---

## 8. Step 6: Layer 2.5 — Fast System 1 Classification (Multilingual Laya)

- **Source File**: `src/api/services/layaMultilingualService.ts` (`processMeeting`)

### 8.1 Objective
Zero-dependency, high-speed heuristic classification running in $< 50\text{ms}$. Extracts dialogue acts, consensus decisions, and task delegations across Hindi, Hinglish, and English using deterministic regex state machines.

### 8.2 Dialogue Act & Decision Heuristics
- Categorizes utterances into `PROPOSAL`, `AFFIRMATION`, `REJECTION`, `ACTION_ITEM`, `QUESTION`, `STATEMENT`.
- **Consensus Decision Protocol**:
  When Speaker A makes a `PROPOSAL` and Speaker B immediately follows with an `AFFIRMATION` (e.g., *theek hai*, *done*, *agreed*, *let's do it*), Laya registers a high-confidence consensus decision.
- **Output**: Returns `LayaAnalysisResult` containing pre-verified decisions, which act as ground-truth anchor constraints for the subsequent Gemini LLM stage.

---

## 9. Step 7: Layer 3 — Conversational Intelligence & Meeting Analysis (Google Gemini)

- **Source File**: `src/api/services/geminiService.ts` (`analyzeMeeting`)
- **Integration Caller**: `src/api/services/meetingProcessingService.ts` (lines 166–183)
- **Active Model**: `gemini-3.5-flash-lite`

### 9.1 Transcript Representation
The transcript is serialized into human-readable, timestamped conversational blocks via `deepgramService.formatTranscript`:
```text
[00:04] Ayaz: Kal tak deployment complete karna hai.
[00:08] Person 1: Done, I will handle the Docker build.
```

### 9.2 Ground Truth Anchoring
The System 1 decisions extracted by Laya are explicitly injected into the Gemini prompt:
```text
PRE-VERIFIED SYSTEM 1 GROUND-TRUTH DECISIONS:
[
  {
    "title": "Complete deployment by tomorrow",
    "decidedBy": "Ayaz",
    "timestamp": 4.2
  }
]
Ensure your summary and key points align with these confirmed decisions, and include them in your decisions list.
```
This dual-process approach (System 1 fast heuristic + System 2 deep generative reasoning) prevents Gemini from omitting critical operational decisions.

### 9.3 Structured JSON Analysis Schema
Gemini is instructed with `config: { responseMimeType: 'application/json' }` to return:

```json
{
  "summary": "A concise 2-3 sentence summary of the meeting",
  "keyPoints": [
    "Discussion point 1",
    "Discussion point 2"
  ],
  "decisions": [
    {
      "title": "Short decision title",
      "description": "Detailed description of what was agreed",
      "decidedBy": "Speaker name or Person X",
      "timestamp": 4.2,
      "context": "Verbatim conversational context",
      "confidence": 0.95
    }
  ],
  "events": [
    {
      "title": "Event or milestone title",
      "description": "Event description",
      "eventType": "scheduled_meeting | deadline | milestone | appointment | other",
      "scheduledAt": "2026-11-29T10:00:00Z or null",
      "mentionedBy": "Speaker name or Person X",
      "timestamp": 12.5,
      "context": "Context from dialogue",
      "confidence": 0.9
    }
  ],
  "actionItems": [
    {
      "task": "Task description",
      "assignedTo": "Person name or null",
      "dueDate": "ISO 8601 date string or null",
      "priority": "high | medium | low",
      "timestamp": 14.0
    }
  ],
  "participants": [
    {
      "name": "Speaker name or Person X",
      "role": "Inferred role (e.g., engineer, manager)",
      "utterances": 14
    }
  ],
  "sentiment": "positive | neutral | negative | mixed",
  "topics": ["Docker", "Deployment", "Deadlines"]
}
```

### 9.4 Specific Extraction Directives
1. **Timestamp Conversion**: Timestamps in `[MM:SS]` are converted into floating-point seconds.
2. **Event Chronology Rule**: **Events must be ordered by discussion timestamp (when spoken in audio)**, NOT by future calendar dates.
3. **Exact Speaker Labels**: Must strictly preserve speaker names assigned during Layer 2.1/2.4 (e.g., `Ayaz` or `Person 1`).

---

## 10. Step 8: Layer 4 — Deduplication & Final Persistence

- **Source File**: `src/api/services/meetingProcessingService.ts` (lines 185–264)
- **Database Service**: `src/api/services/databaseService.ts`

### 10.1 Decision Merging & Deduplication Algorithm
The pipeline combines System 1 (Laya) decisions and System 2 (Gemini) decisions without duplication:
```typescript
const combinedDecisions = [...layaResult.decisions];

for (const geminiDec of analysis.decisions) {
  const isDuplicate = combinedDecisions.some(
    (d) =>
      Math.abs(d.timestamp - geminiDec.timestamp) < 5 ||
      d.title.toLowerCase().includes(geminiDec.title.toLowerCase()) ||
      geminiDec.title.toLowerCase().includes(d.title.toLowerCase())
  );
  if (!isDuplicate) {
    combinedDecisions.push(geminiDec);
  }
}
```
Two decisions are considered duplicates if their audio timestamps occur within **5 seconds** of each other or if one decision title is a substring of the other.

### 10.2 Database Operations
1. **Summary Persistence (`MeetingSummary` Table)**:
   ```typescript
   await databaseService.createMeetingSummary(
     meetingId,
     analysis.summary,
     analysis.keyPoints,
     analysis.actionItems,
     analysis.participants
   );
   ```
2. **Decisions Persistence (`Decision` Table)**:
   Persists all unified decisions linked to `meetingId`.
3. **Events Persistence (`Event` Table)**:
   Parses `scheduledAt` strings into native PostgreSQL `DateTime` objects (or `null` if unscheduled).
4. **Meeting Finalization (`Meeting` Table)**:
   Updates status to `'completed'`:
   ```typescript
   await databaseService.updateMeeting(meetingId, { status: 'completed' });
   ```

### 10.3 Failure Handling
If any fatal error occurs at any point in the pipeline:
1. Captures total execution time up to the failure point.
2. Logs error to standard error with `[meetingId]` prefix.
3. Updates meeting status in database:
   ```typescript
   await databaseService.updateMeeting(meetingId, {
     status: 'failed',
     processingError: error.message,
   });
   ```

---

## 11. Pipeline Execution Timings & Return Schema

The pipeline resolves with a `ProcessingResult` object tracking execution duration across every layer:

```typescript
export interface ProcessingResult {
  meetingId: string;
  status: 'completed' | 'failed';
  transcriptCount: number;
  decisionsCount: number;
  eventsCount: number;
  error?: string;
  timings?: {
    audioProcessing: number;          // Step 1: FFmpeg transcoding
    deepgramSTT: number;              // Step 2: Deepgram STT
    ecapaReDiarization: number;       // Step 3: ECAPA-TDNN ONNX embeddings & clustering
    geminiPhoneticRefinement: number; // Step 4: Gemini transcript refinement
    layaSystem1: number;              // Step 6: Multilingual Laya dialogue classification
    geminiLLMAnalysis: number;        // Step 7: Gemini meeting intelligence analysis
    databasePersistence: number;      // Steps 5 & 8: PostgreSQL writes
    total: number;                    // End-to-end processing duration
  };
}
```

---

## 12. Component & Model Reference Summary

| Layer | Component Name | Technology / Library | Model / Algorithm Used | Primary Purpose |
| :--- | :--- | :--- | :--- | :--- |
| **Ingestion** | `MeetingController` | Express, `express-fileupload` | Multipart stream, `setImmediate` | Audio ingestion & async dispatch |
| **Layer 1** | `AudioProcessingService` | `fluent-ffmpeg` | 16kHz Mono `pcm_s16le` transcode | Standardized acoustic baseline |
| **Layer 2** | `DeepgramService` | `@deepgram/sdk` | Nova-3 (`multi`/`en-IN`), Nova-2 (`hi`) | STT, initial turns, word timestamps |
| **Layer 2.1**| `VoiceRecognitionService` | `onnxruntime-node` | ECAPA-TDNN (192-dim) + AHC | Fixes speaker collapse, attributes users |
| **Layer 2.3**| `GeminiService` | `@google/genai` | `gemini-3.5-flash-lite` + Consonant Skeletons | Phonetic term & name correction |
| **Layer 2.4**| `MeetingProcessingService`| Prisma ORM (PostgreSQL) | Dynamic Centroid Serialization | Transcripts & UnknownSpeakers DB write |
| **Layer 2.5**| `LayaMultilingualService`| TypeScript native | State-machine regex dialogue rules | Fast System 1 consensus decisions |
| **Layer 3** | `GeminiService` | `@google/genai` | `gemini-3.5-flash-lite` (Structured JSON) | Executive summary, events, action items |
| **Layer 4** | `MeetingProcessingService`| Prisma ORM (PostgreSQL) | Substring & 5s delta deduplication | Final DB persistence & completion status |
