# Meeting Processing Pipeline: Step 1 & Step 2 Technical Specification

This document provides a comprehensive, code-verified technical breakdown of **Step 1 (Audio Standardization & Preprocessing)** and **Step 2 (Speech-to-Text with Speaker Diarization & Language Model Specialization)** in the VoiceSense backend.

All specifications, logic flows, parameters, and design decisions documented below are derived directly from the source code in `src/api/` as the single source of truth.

---

## Pipeline Ingestion & Context

Before Step 1 executes, audio ingestion and meeting initialization occur across `src/api/controllers/meetingController.ts` and `src/api/services/meetingProcessingService.ts`:

1. **HTTP Ingestion Endpoint (`POST /api/meetings/upload`)**:
   - Controller: `MeetingController.uploadMeeting(req, res)`
   - Middleware: `express-fileupload` (`useTempFiles: true`, `tempFileDir: ./uploads/temp`, max `200MB`).
   - Validates existence of `userId` and checks database via `databaseService.getUserById(userId)`.
   - Validates optional `language` payload parameter: normalized to `'english'`, `'hindi'`, or `'bilingual'`. Returns HTTP `400` if invalid.
   - Moves uploaded audio file to `./uploads/audio/meeting_${userId}_${Date.now()}${ext}` via `audioFile.mv(filePath)`.
   - Computes default meeting title from file base name or fallback `Meeting YYYY-MM-DD`.
   - Inserts record into database via `databaseService.createMeeting(userId, meetingTitle, filePath, description, fileSize)` with initial status `'uploaded'`.
   - Returns immediate HTTP `201 Accepted` response with meeting metadata.
   - Dispatches background pipeline asynchronously using `setImmediate(() => meetingProcessingService.processMeeting(...))`.

2. **Pipeline State Initialization (`MeetingProcessingService.processMeeting`)**:
   - Updates meeting status in PostgreSQL: `databaseService.updateMeeting(meetingId, { status: 'processing' })`.
   - Starts global and granular execution timers using `performance.now()`.
   - **Language Selection Resolution Hierarchy**:
     1. `requestedLanguage` passed explicitly in upload request body.
     2. Enrolled voice profile language from `(userProfiles[0] as any)?.language` via `databaseService.getVoiceProfilesByUserId(userId)`.
     3. Default fallback: `'bilingual'`.

---

## Step 1: Layer 1 — Audio Standardization & Preprocessing

- **Primary Source File**: `src/api/services/audioProcessingService.ts`
- **Orchestration Caller**: `src/api/services/meetingProcessingService.ts` (lines 62–73)
- **Database Service**: `src/api/services/databaseService.ts` (`updateMeeting`)

### 1.1 Objective
Convert arbitrary audio containers and codecs (e.g., iPhone M4A/AAC, Android 3GP/AMR, browser WebM/Opus, MP3) into a standardized, uncompressed linear PCM WAV file. This file serves as the single immutable acoustic baseline for all subsequent pipeline stages (Deepgram STT, ECAPA-TDNN ONNX embeddings, and segmentation).

### 1.2 Underlying Dependencies & System Tools
- `fluent-ffmpeg` (v2.1.3): Node.js wrapper for FFmpeg process control.
- `@ffmpeg-installer/ffmpeg` (v1.1.0): Static FFmpeg binary installer. Configured via `ffmpeg.setFfmpegPath(ffmpegInstaller.path)`.
- `@ffprobe-installer/ffprobe` (v2.1.2): Static FFprobe binary installer. Configured via `ffmpeg.setFfprobePath(ffprobeInstaller.path)`.
- Native Node.js `fs` and `path`: Directory creation, file metadata inspection via `promisify(fs.stat)`, and cleanup via `promisify(fs.unlink)`.

### 1.3 Execution Flow & Internal Logic

```
Input File (Any audio format/codec)
        │
        ├── 1. Ensure output directory exists (fs.mkdirSync('./uploads/processed', { recursive: true }))
        ├── 2. Generate target path: processed_${Date.now()}_${basename}.wav
        ├── 3. Read input file size via fs.stat(inputPath)
        ├── 4. Probe duration and format via FFprobe (ffmpeg.ffprobe)
        │
        ▼
FFmpeg Conversion Pipeline
  ├── Frequency: 16000 Hz (16kHz)
  ├── Channels: 1 (Mono)
  ├── Codec: pcm_s16le (16-bit linear PCM Little-Endian)
  └── Format: wav (RIFF header)
        │
        ▼
Post-Conversion Verification
  ├── Check output file size via fs.stat(outputPath)
  ├── Guard: If processedSize > 500 bytes -> use outputPath; Else fallback to inputPath
  ├── Calculate compressionRatio = ((originalSize - finalSize) / originalSize) * 100
  └── Return AudioProcessingResult
```

### 1.4 Detailed FFmpeg Parameter Specification

| Parameter | Value in Code | Technical Purpose & Code Justification |
| :--- | :--- | :--- |
| `audioFrequency` | `16000` (16kHz) | Calibrated Nyquist frequency (8kHz bandwidth) matching Deepgram acoustic models and ECAPA-TDNN Mel filterbank extraction. |
| `audioChannels` | `1` (Mono) | Eliminates stereo panning artifacts and collapses multi-mic tracks into a single coherent mono stream for diarization. |
| `audioCodec` | `'pcm_s16le'` | 16-bit signed integer Linear PCM, Little-Endian byte order. Raw uncompressed audio frames. |
| `format` | `'wav'` | Standard RIFF WAVE container format. |

### 1.5 Critical Code-Level Design Choices

1. **Why Linear PCM WAV instead of Compressed Opus/OGG?**
   - Source code explicit comment (`audioProcessingService.ts`, lines 44–46):
     > *"WAV (`pcm_s16le`) preserves every frequency bin — Opus at 32kbps drops high-frequency consonants that distinguish Hindi phonemes (श/स, ट/त, ड/द) and causes wrong words."*
   - Lossy transform codecs apply psychoacoustic masking that quantizes or discards 4kHz–8kHz high-frequency consonant bands critical for Indian language phonetics.

2. **Deliberate Absence of Denoise and Loudness Normalization Filters:**
   - Source code explicit comment (`audioProcessingService.ts`, lines 47–53):
     > *"IMPORTANT: no extra filters (denoise/loudness) here on purpose. This exact signal is ALSO the input to ECAPA-TDNN voice embeddings and Deepgram's utterance segmentation. Adding cleanup filters measurably perturbs both — embeddings shift, cluster thresholds flip, and short utterances get inherited by the WRONG speaker. The raw 16kHz mono signal is the baseline the speaker-recognition layer is calibrated against; STT accuracy is handled at the model level (nova-3 multi / keyterms / Gemini refinement)."*
   - DSP filters (e.g., `afftdn`, `arnndn`, `dynaudnorm`) alter spectral formant shapes and non-linearly perturb phase, directly degrading cosine distance accuracy in downstream neural embeddings.

### 1.6 Size Verification & Fault Tolerance
- Once FFmpeg process emits `'end'`, output file size is measured with `await stat(outputPath)`.
- **Integrity Guard**:
  ```typescript
  const finalPath = processedSize > 500 ? outputPath : inputPath;
  const finalSize = processedSize > 500 ? processedSize : originalSize;
  ```
  If FFmpeg outputs an empty or corrupted header (`<= 500` bytes), pipeline falls back safely to `inputPath` rather than failing outright.

### 1.7 Return Interface & Database State Mutation

- **Interface Output (`AudioProcessingResult`)**:
  ```typescript
  export interface AudioProcessingResult {
    processedFilePath: string;
    originalSize: number;
    processedSize: number;
    compressionRatio: number;
    duration: number;
  }
  ```

- **Database State Update**:
  `meetingProcessingService.ts` records the standardized output:
  ```typescript
  await databaseService.updateMeeting(meetingId, {
    processedAudioUrl: audioResult.processedFilePath,
    durationSeconds: audioResult.duration,
  });
  ```
- **Performance Metric**:
  Elapsed time recorded in `timings.audioProcessing = performance.now() - audioStart`.

---

## Step 2: Layer 2 — Speech-to-Text with Speaker Diarization & Model Specialization

- **Primary Source File**: `src/api/services/deepgramService.ts`
- **Orchestration Caller**: `src/api/services/meetingProcessingService.ts` (lines 74–95)
- **External Engine**: Deepgram Pre-recorded Speech-to-Text API

### 2.1 Objective
Process the standardized 16kHz mono WAV file through Deepgram's neural STT engine to generate time-aligned, punctuated, formatted transcript segments with initial integer speaker diarization labels (`0`, `1`, `2`, ...), word-level timestamps, and confidence scores.

### 2.2 Underlying Dependencies & Authentication
- `@deepgram/sdk` (v3.4.0): Initialized via `createClient(process.env.DEEPGRAM_API_KEY || '')`.
- Environment Variables:
  - `DEEPGRAM_API_KEY`: Authentication secret for Deepgram Cloud.
  - `DEEPGRAM_KEYTERMS`: Comma-separated list of rare domain vocabulary to boost in transcription.

### 2.3 Input Verification & Memory Buffering
Before network invocation, `deepgramService.transcribeWithDiarization` verifies file existence and contents:
1. `fs.existsSync(audioFilePath)`: Throws error if file missing.
2. `fs.readFileSync(audioFilePath)`: Loads audio into an in-memory `Buffer`.
3. Validates buffer: `if (audioBuffer.length < 500) throw new Error('Audio file is empty or contains no detectable audio frames.')`.

### 2.4 Domain Vocabulary & Keyterm Boosting (`DEEPGRAM_KEYTERMS`)
Vocabulary boosting is config-driven without hardcoded terms in application code:
- Extracted via `getConfiguredKeyterms()`:
  ```typescript
  const configuredKeyterms = (process.env.DEEPGRAM_KEYTERMS || '')
    .split(',')
    .map((t) => t.trim())
    .filter((t) => t.length > 0);
  ```
- **Generation-Specific Parameter Syntax**:
  Deepgram SDK syntax differs fundamentally across model generations:
  - **Nova-2**: Expects `keywords: string[]` where each string supports boost weighting (syntax: `"term:3"`).
  - **Nova-3**: Expects `keyterm: string[]` where each string is an unweighted plain text term.
- **Accuracy Rule in Code**: Only rare, domain-specific terminology is boosted (e.g., product names). Boosting common conversational words (e.g., `meeting`, `review`) harms accuracy and induces false-positive phonetic substitutions.

### 2.5 Base Diarization & Transcription Parameters
Shared base configuration applied to all language requests:
```typescript
const baseConfig: any = {
  smart_format: true,   // Formats dates, numbers, currency, phone numbers
  punctuate: true,      // Inserts commas, periods, question marks
  diarize: true,        // Detects speaker turns; assigns integer speaker labels
  utterances: true,     // Groups words into continuous conversational turns
  paragraphs: false,    // Disables paragraph blocks to retain raw utterance boundaries
  filler_words: false,  // Strips um/uh filler tokens
};
```

### 2.6 Language Model Selection & Ordered Fallback Matrix

The pipeline constructs an ordered array `configs: any[]` based on normalized `selectedLanguage`. If Deepgram rejects a model (e.g., tier permissions or temporary capacity), the system loops and falls back automatically.

```
Language Selection Logic
           │
           ├── "english" | "en"
           │     ├── 1. Primary:  model: 'nova-3', language: 'en-IN', keyterm: configuredKeyterms
           │     └── 2. Fallback: model: 'nova-2', language: 'en-IN', keywords: ["term:3"]
           │
           ├── "hindi" | "hi"
           │     └── 1. Primary:  model: 'nova-2', language: 'hi',    keywords: ["term:3"]
           │
           └── "bilingual" (Default / Hinglish)
                 ├── 1. Primary:  model: 'nova-3', language: 'multi', keyterm: configuredKeyterms
                 └── 2. Fallback: model: 'nova-2', language: 'hi',    keywords: ["term:3"]
```

#### Detailed Model Specification:
1. **`english` / `en`**:
   - Primary: `nova-3` with `language: 'en-IN'`. Specialized for Indian English phonetics, syllable timing, and cadence.
   - Fallback: `nova-2` with `language: 'en-IN'`.
2. **`hindi` / `hi`**:
   - Primary: `nova-2` with `language: 'hi'`. Dedicated deterministic Hindi acoustic model.
3. **`bilingual` / Hinglish (Default)**:
   - Primary: `nova-3` with `language: 'multi'`. Deepgram's dedicated multilingual code-switching model.
   - **Crucial Code Distinction**: `language: 'multi'` performs **per-utterance** language evaluation. Code comments explicitly note that `detect_language` is invalid because it forces a single global language across the whole file (corrupting mixed Hindi/English dialogues), while `multi_language` is an invalid Deepgram parameter name.
   - Fallback: `nova-2` with `language: 'hi'`.

### 2.7 API Execution & Fallback Retry Loop
Execution proceeds iteratively across `configs`:
```typescript
let result: any = null;
let lastError: any = null;
for (const cfg of configs) {
  try {
    const response = await deepgram.listen.prerecorded.transcribeFile(audioBuffer, cfg);
    if (response.error) {
      throw new Error(`Deepgram transcription failed: ${response.error.message}`);
    }
    if (!response.result) {
      throw new Error('No transcription result received from Deepgram');
    }
    result = response.result;
    break;
  } catch (err: any) {
    lastError = err;
    console.warn(`[Deepgram] Config model=${cfg.model} language=${cfg.language} failed (${err.message}); trying next fallback...`);
  }
}
if (!result) {
  throw lastError || new Error('Deepgram transcription failed for all configurations');
}
```

### 2.8 Utterance Parsing & Data Normalization
The raw Deepgram response is parsed from `result.results.utterances`. Each utterance is transformed into a `TranscriptionSegment`:

```typescript
const segments: TranscriptionSegment[] = [];
const utterances = result.results?.utterances || [];

for (const utterance of utterances) {
  segments.push({
    text: utterance.transcript || '',
    startTime: utterance.start || 0,
    endTime: utterance.end || 0,
    speaker: utterance.speaker ?? 0,        // Deepgram integer speaker ID (0, 1, 2, ...)
    confidence: utterance.confidence || 0,
    words: (utterance.words || []).map((word: any) => ({
      word: word.word || '',
      start: word.start || 0,
      end: word.end || 0,
      confidence: word.confidence || 0,
      speaker: word.speaker ?? utterance.speaker ?? 0,
    })),
  });
}
```

Metadata aggregated:
- `speakers`: Count of unique speaker IDs: `new Set(segments.map((s) => s.speaker)).size`.
- `duration`: Audio duration reported by Deepgram: `result.metadata?.duration || 0`.

### 2.9 Handoff to Layer 2.1 & Deepgram Speaker Snapshot
Immediately after Step 2 completes in `meetingProcessingService.ts`:
1. **Diarization Snapshot**:
   ```typescript
   for (const seg of transcriptionResult.segments) {
     seg.deepgramSpeakerLabel = seg.speaker;
   }
   ```
   *Reason*: In Step 2.1, local acoustic re-diarization (`voiceRecognitionService.reDiarizeSegments`) will re-evaluate and overwrite `seg.speaker` using ECAPA-TDNN neural embeddings to fix Deepgram speaker collapse. The original Deepgram assignment is snapshot into `deepgramSpeakerLabel` so both raw and refined speaker assignments persist into the `Transcript` database table.
2. **Performance Metric**:
   Step duration recorded in `timings.deepgramSTT = performance.now() - sttStart`.

---

## Step 1 & Step 2 Data Interface Reference

All data structures shared between Step 1 and Step 2 directly defined in source code:

```typescript
// From src/api/services/audioProcessingService.ts
export interface AudioProcessingResult {
  processedFilePath: string;
  originalSize: number;
  processedSize: number;
  compressionRatio: number;
  duration: number;
}

// From src/api/services/deepgramService.ts
export interface TranscriptionWord {
  word: string;
  start: number;
  end: number;
  confidence: number;
  speaker?: number;
}

export interface TranscriptionSegment {
  text: string;
  startTime: number;
  endTime: number;
  speaker: number;
  confidence: number;
  words?: TranscriptionWord[];
  /** Deepgram's original speaker index, preserved before acoustic re-diarization */
  deepgramSpeakerLabel?: number;
  /** Best cosine similarity score vs the matched enrolled profile */
  similarityScore?: number;
}

export interface DeepgramTranscriptionResult {
  segments: TranscriptionSegment[];
  speakers: number;
  duration: number;
}
```

---

## Summary Matrix: Step 1 vs Step 2

| Dimension | Step 1: Layer 1 (Audio Processing) | Step 2: Layer 2 (Speech-to-Text & Diarization) |
| :--- | :--- | :--- |
| **Executing Service** | `AudioProcessingService` (`audioProcessingService.ts`) | `DeepgramService` (`deepgramService.ts`) |
| **Primary Tool/Library** | `fluent-ffmpeg`, `@ffmpeg-installer/ffmpeg`, `@ffprobe-installer/ffprobe` | `@deepgram/sdk` (`listen.prerecorded.transcribeFile`) |
| **Input** | Original uploaded audio file (`./uploads/audio/...`) | 16kHz Mono WAV (`./uploads/processed/...`) |
| **Output** | `AudioProcessingResult` (processed file path, sizes, duration) | `DeepgramTranscriptionResult` (`segments`, `speakers`, `duration`) |
| **Target Specs** | 16,000 Hz, 1 Channel Mono, `pcm_s16le`, WAV container | Transcribed text, timestamps, word alignment, Deepgram speaker IDs |
| **Model Selection** | Deterministic DSP / Linear PCM transcode (no filters) | `nova-3` multi / `nova-2` hi / `nova-3` en-IN with fallback chains |
| **Config Flags** | No denoise, no loudness normalization | `smart_format`, `punctuate`, `diarize`, `utterances`, `keyterm`/`keywords` |
| **DB Updates** | `Meeting.processedAudioUrl`, `Meeting.durationSeconds` | In-memory segment enhancement (`deepgramSpeakerLabel` snapshot) |
| **Timing Metric** | `timings.audioProcessing` | `timings.deepgramSTT` |
