# F5: Deep Extract (On-Demand) — Design & Development Guide

**Priority:** P0 (MVP)
**Phase:** 1
**Platforms:** iOS (Swift) + Android (Kotlin)
**Dependencies:** F1, F2, F4
**Estimated Effort:** 3–4 weeks

---

## 1. Feature Overview

Deep Extract is the server-assisted pipeline that processes the full video content — downloading the video, transcribing the audio, OCR-ing key frames, and running AI extraction on the combined text. This triggers when the lightweight on-device analysis didn't capture enough detail (e.g., the caption was just "😍🔥 link in bio").

This is also the core monetization gate: free users get 10 deep extracts/month, paid users get unlimited.

---

## 2. User Stories

| ID | Story | Acceptance Criteria |
|----|-------|-------------------|
| US-5.1 | As a user, I want to get full recipe details from a vague Reel caption | Tap "Get Full Details" → full ingredients + steps appear within 30 sec |
| US-5.2 | As a user, I want to see progress while deep extract runs | Progress indicator with meaningful stages |
| US-5.3 | As a free user, I want to know how many deep extracts I have left | Visible counter before triggering extract |
| US-5.4 | As a user, I want deep extract to work even if I close the app | Background processing completes the job |
| US-5.5 | As a paid user, I want deep extracts to run automatically for vague saves | Auto-trigger option in settings |

---

## 3. UX Design

### 3.1 Trigger Points

Deep Extract is triggered from four places:

**A) "Get Full Details" button on cards (Primary)**
```
┌───────────────────────────────────────┐
│ [Thumbnail]                    📌     │
│ "This is incredible!! 😍"            │
│                                       │
│ ┌─────────────────────────────────┐   │
│ │  🔍 Get Full Details            │   │
│ │  7 of 10 free extracts left     │   │
│ └─────────────────────────────────┘   │
└───────────────────────────────────────┘
```

**B) Batch extract from library (Secondary)**
Multi-select cards in library → "🔍 Extract" button in batch toolbar.

**C) Auto-extract on save (Paid users, optional)**
Settings toggle: "Auto-extract when details are incomplete."

**D) From Share Extension (Paid users with auto-extract on)**
Low-richness saves trigger background extraction immediately after share extension capture.

### 3.2 Deep Extract Flow — Free User

```
User taps "Get Full Details"
         │
         ▼
┌─────────────────────────────────────┐
│                                     │
│  🔍 Get Full Details               │
│                                     │
│  We'll analyze the video to pull    │
│  out the important stuff — like     │
│  ingredients, locations, or prices. │
│                                     │
│  You have 7 of 10 free extracts    │
│  remaining this month.              │
│                                     │
│  ┌───────────────────────────┐     │
│  │     Extract Now           │     │
│  └───────────────────────────┘     │
│                                     │
│  ┌───────────────────────────┐     │
│  │     Cancel                │     │
│  └───────────────────────────┘     │
│                                     │
└─────────────────────────────────────┘
```

**Why a confirmation dialog for free users:** Each extract costs one of their 10 monthly credits. They deserve to make an informed choice.

**Paid users skip the dialog entirely** — tapping "Get Full Details" starts extraction immediately. No friction for paying customers.

### 3.3 Progress States

Progress is shown inline on the card itself (not as a separate screen or modal). The card content updates live as extraction progresses.

```
┌───────────────────────────────────────┐
│ [Thumbnail]                    📌     │
│ "This is incredible!! 😍"            │
│                                       │
│ ┌─────────────────────────────────┐   │
│ │                                 │   │
│ │  🌐 Fetching video...          │   │
│ │  ━━━━━━━━━━░░░░░░░░░░░        │   │
│ │                                 │   │
│ │               Cancel            │   │
│ └─────────────────────────────────┘   │
└───────────────────────────────────────┘
         ↓ (3-5 sec later)
┌───────────────────────────────────────┐
│ [Thumbnail]                    📌     │
│ "This is incredible!! 😍"            │
│                                       │
│ ┌─────────────────────────────────┐   │
│ │                                 │   │
│ │  🎙️ Listening to audio...     │   │
│ │  ━━━━━━━━━━━━━━━░░░░░░        │   │
│ │                                 │   │
│ │               Cancel            │   │
│ └─────────────────────────────────┘   │
└───────────────────────────────────────┘
         ↓ (10-15 sec later)
┌───────────────────────────────────────┐
│ [Thumbnail]                    📌     │
│ "This is incredible!! 😍"            │
│                                       │
│ ┌─────────────────────────────────┐   │
│ │                                 │   │
│ │  🧠 Pulling out the details... │   │
│ │  ━━━━━━━━━━━━━━━━━━━━░        │   │
│ │                                 │   │
│ │               Cancel            │   │
│ └─────────────────────────────────┘   │
└───────────────────────────────────────┘
         ↓ (5 sec later)
  Card type transitions to Recipe/Travel/Product
  with extracted data populating live
```

**Progress bar is stage-based, not percentage-based:** Users don't care about 37% vs 42%. They care about which stage they're in. Each stage advances the bar by roughly equal increments. The bar moves smoothly between stages (not in jumps) to feel alive.

**Stage labels are human-friendly:**
- "Fetching video..." (not "Calling scraper API")
- "Listening to audio..." (not "Transcribing via Whisper")
- "Pulling out the details..." (not "Running LLM extraction")

**Cancel button available at every stage.** If cancelled:
- No quota deducted for free users
- Card returns to its pre-extraction state
- Partially fetched data is discarded (clean rollback)

### 3.4 Completion — Card Transitions In-Place

When extraction completes, the card doesn't show a summary dialog. Instead, the generic card smoothly morphs into the structured card:

1. Progress bar fills to 100% → brief checkmark animation (200ms)
2. "Get Full Details" section fades out
3. Structured sections (Ingredients, Steps, Map, etc.) fade in sequentially with 100ms stagger
4. Category badge updates if category changed
5. Brief haptic pulse on completion

**Why no summary popup:** Showing "Found 8 ingredients" as an intermediary step adds a tap before the user sees the actual data. The data itself is the payoff — let them see it immediately.

**Notification if app is backgrounded:** If the user closed the app during extraction, send a local notification: "✅ Your Garlic Pasta save is ready with full details." Tapping the notification deep-links to the card.

### 3.5 Batch Extract

When multiple cards are selected in multi-select mode (F3) and user taps "🔍 Extract":

```
┌───────────────────────────────────────┐
│ Extracting 3 of 5 saves              │
│                                       │
│ ✅ Garlic Pasta — Done (Recipe)      │
│ ✅ Bali Beaches — Done (Travel)      │
│ 🔄 Amazon Find — Analyzing...        │
│ ⏳ Morning Routine — Queued          │
│ ⏳ HIIT Workout — Queued             │
│                                       │
│ ┌─────────────────────────────────┐   │
│ │         Cancel Remaining        │   │
│ └─────────────────────────────────┘   │
└───────────────────────────────────────┘
```

- Extracts run **sequentially** (not in parallel) to control server costs and maintain processing quality
- Each completed save gets a checkmark + the discovered category
- "Cancel Remaining" stops unstarted extracts. Already-completed ones keep their results.
- Free users: batch is limited by remaining quota. "You have 3 extracts left. Only the first 3 of 5 selected saves will be extracted."

### 3.6 Auto-Extract (Paid Users)

Settings → Extraction → "Auto-extract when details are incomplete"

```
┌───────────────────────────────────────┐
│ Auto-Extract                    [ON]  │
│                                       │
│ When you save content and we can't   │
│ find enough details from the caption  │
│ alone, we'll automatically analyze    │
│ the video in the background.          │
│                                       │
│ Only runs on Wi-Fi by default.        │
│                                       │
│ ☐ Also run on cellular data          │
└───────────────────────────────────────┘
```

**Wi-Fi default:** Auto-extract downloads video which consumes data. Defaulting to Wi-Fi-only is respectful of mobile data caps, especially in India.

When auto-extract runs in background:
- Card shows a subtle animated "sparkle" indicator in the library grid (not a spinner — too heavy)
- When complete, card updates silently. No notification (would be too noisy for power users saving 10+/day). The "NEW" badge from F3 changes to a "✨ Updated" badge.

### 3.7 Quota Exhausted — Free User

```
┌─────────────────────────────────────┐
│                                     │
│  You've used all 10 free extracts   │
│  for February.                      │
│                                     │
│  Resets March 1.                    │
│                                     │
│  ┌───────────────────────────┐     │
│  │  ⭐ Unlock Unlimited      │     │
│  │  Staq Pro · $4.99/month   │     │
│  └───────────────────────────┘     │
│                                     │
│  ┌───────────────────────────┐     │
│  │     Not Now               │     │
│  └───────────────────────────┘     │
│                                     │
│  Your saves still work — you can   │
│  browse, search, and view original │
│  content anytime.                  │
│                                     │
└─────────────────────────────────────┘
```

**Key UX detail:** Reassurance text at the bottom ("Your saves still work") prevents the user from feeling locked out of the app. Only deep extract is gated, not the core functionality.

### 3.8 Error States

```
Scraper Failed:
┌───────────────────────────────────────┐
│                                       │
│  ⚠️ Couldn't analyze this video      │
│                                       │
│  This sometimes happens when the      │
│  platform restricts access. This      │
│  doesn't count against your quota.    │
│                                       │
│  ┌─────────────────────────────┐     │
│  │     Try Again               │     │
│  └─────────────────────────────┘     │
│  ┌─────────────────────────────┐     │
│  │     View Original Instead   │     │
│  └─────────────────────────────┘     │
└───────────────────────────────────────┘

Partial Success:
┌───────────────────────────────────────┐
│                                       │
│  ⚠️ Partial results                  │
│                                       │
│  We found some details but couldn't  │
│  get everything. The video might     │
│  not have had clear audio.           │
│                                       │
│  ┌─────────────────────────────┐     │
│  │     Keep What We Found      │     │
│  └─────────────────────────────┘     │
│  ┌─────────────────────────────┐     │
│  │     Try Again               │     │
│  └─────────────────────────────┘     │
└───────────────────────────────────────┘
```

**Key UX principle:** Failed extracts do NOT count against free quota. Only fully successful completions decrement the counter. Partial successes also don't count — user can choose to keep partial results and try again later.

### 3.9 Accessibility

- **VoiceOver / TalkBack:** Progress stages announced dynamically: "Fetching video. Stage 1 of 3." → "Listening to audio. Stage 2 of 3." → "Complete. Card updated with recipe details."
- **Cancel button:** Large (full-width), accessible label: "Cancel extraction."
- **Batch extract list:** Each item announces status: "Garlic Pasta. Completed. Recipe."
- **Reduce Motion:** Progress bar animation replaced with static stage labels that swap in-place. Card transition uses crossfade instead of morphing.

---

## 4. Technical Pipeline

### 4.1 Full Processing Flow

```
Client                          Server                    Client (on-device)
  │                                │                            │
  │  POST /extract {url}           │                            │
  │ ─────────────────────────────► │                            │
  │                                │                            │
  │                          Scrape API call                    │
  │                          (Apify/ScrapeCreators)             │
  │                          Fetch video + metadata             │
  │                                │                            │
  │  SSE: {stage: "fetching", %}   │                            │
  │ ◄───────────────────────────── │                            │
  │                                │                            │
  │  Response: {video_url,         │                            │
  │   caption, thumbnail,          │                            │
  │   metadata}                    │                            │
  │ ◄───────────────────────────── │                            │
  │                                                             │
  │  Download video from URL ─────────────────────────────────► │
  │                                                             │
  │                                          AVFoundation extract audio
  │                                          SpeechAnalyzer transcribe
  │                                          Vision OCR key frames
  │                                          Foundation Models extract
  │                                                             │
  │  ◄─────────────────────────── Structured data complete ──── │
  │                                                             │
  │  POST /extract/complete                                     │
  │  {save_id, extracted_data}     │                            │
  │ ─────────────────────────────► │                            │
  │                          Cache result for URL dedup         │
```

### 4.2 Server API Design

**Endpoint:** `POST /api/v1/extract`

```json
// Request
{
  "url": "https://www.instagram.com/reel/ABC123/",
  "platform": "instagram",
  "save_id": "uuid-of-save",
  "device_can_transcribe": true
}

// Response (streamed via SSE)
event: progress
data: {"stage": "fetching", "percent": 10}

event: progress
data: {"stage": "fetching", "percent": 50}

event: video_ready
data: {
  "video_url": "https://cdn.../video.mp4",
  "full_caption": "Full caption text from scrape...",
  "creator": "@cookingwithsara",
  "hashtags": ["#recipe", "#pasta", "#easy"],
  "thumbnail_url": "https://..."
}

// If device can't transcribe, server does it:
event: progress
data: {"stage": "transcribing", "percent": 70}

event: transcript_ready
data: {"transcript": "Today I'm going to show you my 15-minute garlic pasta..."}

event: progress
data: {"stage": "extracting", "percent": 90}

event: complete
data: {
  "category": "recipe",
  "extracted_data": { ... },
  "richness_score": 0.92
}
```

#### 4.2.1 SSE Client Implementation

SSE (Server-Sent Events) keeps the client informed of progress in real time without polling.

**iOS:**
```swift
// URLSession-based EventSource pattern
let request = URLRequest(url: extractURL)
let session = URLSession(configuration: .default, delegate: sseDelegate, delegateQueue: nil)
let task = session.dataTask(with: request)

// SSEDelegate parses the text/event-stream response incrementally:
// - Buffers incoming bytes, splits on double-newline boundaries
// - Parses "event:" and "data:" fields per the SSE spec
// - Dispatches parsed events to the UI via MainActor
// - Handles connection drops with automatic reconnect (3 retries, 2s backoff)
// - Sets Last-Event-ID header on reconnect so server can resume from last event
```

**Android:**
```kotlin
// OkHttp SSE support via EventSource
val request = Request.Builder()
    .url(extractUrl)
    .header("Accept", "text/event-stream")
    .build()

val eventSource = EventSources.createFactory(okHttpClient)
    .newEventSource(request, object : EventSourceListener() {
        override fun onEvent(es: EventSource, id: String?, type: String?, data: String) {
            // Parse stage/percent from JSON data
            // Update UI on Main dispatcher
        }
        override fun onFailure(es: EventSource, t: Throwable?, response: Response?) {
            // Reconnect with exponential backoff (max 3 retries)
        }
    })
```

**Key SSE behaviors:**
- Connection timeout: 90 seconds (longer than the 60s job timeout to allow for network variance)
- If SSE connection drops but job is still running, client polls `GET /api/v1/extract/{job_id}/status` as fallback
- Client sends `Last-Event-ID` on reconnect so server can replay missed events from its in-memory buffer (last 50 events per job)

#### 4.2.2 Scraper Service Layer

The scraper is the most fragile part of the pipeline — platforms change their anti-bot measures frequently. Abstract it behind a strategy pattern so swapping providers is a config change, not a code change.

**Scraper Strategy Interface:**
```typescript
interface ScraperProvider {
  name: string;
  scrape(url: string, platform: Platform): Promise<ScrapeResult>;
  healthCheck(): Promise<boolean>;
  costPerResult: number;
}
```

**Provider Chain (ordered by cost/reliability):**

| Priority | Provider | Cost | Strengths |
|----------|----------|------|-----------|
| Primary | Apify Instagram/TikTok scraper | $0.0026/result | Most reliable, best metadata, supports both platforms |
| Secondary | ScrapeCreators | $0.002/credit | Cheapest, good for Instagram, weaker TikTok support |
| Tertiary | Bright Data | $0.003/result | Most resilient to anti-bot, but slowest and most expensive |

**Circuit Breaker Pattern:**
```
Each provider has a circuit breaker with three states:
  CLOSED (healthy) → requests flow through normally
  OPEN (tripped) → all requests skip this provider
  HALF-OPEN (testing) → one test request allowed through

Trip conditions:
  - 3 failures within a 5-minute sliding window → trip to OPEN
  - OPEN state lasts 15 minutes, then transitions to HALF-OPEN
  - 1 successful request in HALF-OPEN → reset to CLOSED
  - 1 failure in HALF-OPEN → back to OPEN for another 15 minutes

Failure = HTTP 5xx, timeout (>15s), or scraper-specific error codes
Rate limits (HTTP 429) count as failures.
```

**Health Check Endpoint:**
```
GET /api/v1/internal/scrapers/health

Response:
{
  "apify": { "status": "healthy", "circuit": "closed", "avg_latency_ms": 2800, "success_rate_1h": 0.97 },
  "scrapecreators": { "status": "healthy", "circuit": "closed", "avg_latency_ms": 3400, "success_rate_1h": 0.94 },
  "brightdata": { "status": "degraded", "circuit": "half-open", "avg_latency_ms": 5200, "success_rate_1h": 0.82 }
}
```

This endpoint is internal only (not exposed to clients). Wire it into your monitoring dashboard and set alerts when all three providers are degraded simultaneously — that means the platform changed something and we need to act.

#### 4.2.3 Job Queue Architecture

Extract requests are processed asynchronously via a job queue. This decouples the API layer from the processing layer and gives us retries, priority, and observability for free.

**Stack:** BullMQ (Node.js) backed by Redis. If the server is Python, use Celery with Redis broker instead — the semantics are identical.

**Job Lifecycle:**
```
pending → fetching → transcribing → extracting → complete
                                                 └→ failed
```

| State | Description | Typical Duration |
|-------|-------------|-----------------|
| `pending` | Job enqueued, waiting for a worker | 0–5s (free), 0–1s (paid) |
| `fetching` | Scraper is downloading video + metadata | 3–8s |
| `transcribing` | Server-side Whisper running (only if device can't transcribe) | 5–15s |
| `extracting` | LLM extraction on combined text | 3–8s |
| `complete` | Result cached and returned to client | — |
| `failed` | All retries exhausted | — |

**Priority Queue:**
```
Two priority levels:
  - Priority 1 (high): Paid users, auto-extract jobs
  - Priority 2 (normal): Free users

Paid users' jobs are always dequeued before free users' jobs.
Within the same priority, FIFO ordering.
```

**Timeouts and Retries:**
```
Job timeout: 60 seconds max (hard kill after 60s)
Retry policy:
  - Max 2 retries per job
  - Retry delay: 5s (first), 15s (second)
  - Only retry on transient errors (scraper timeout, network error)
  - Do NOT retry on permanent errors (video deleted, private account)

Dead letter queue (DLQ):
  - Jobs that fail all retries land in the DLQ
  - DLQ is drained daily — aggregate failure reasons for debugging
  - Alert if DLQ depth exceeds 50 jobs in 1 hour (signals a systemic issue)
```

**Concurrency Limits:**
```
Per-worker: 3 concurrent jobs (limited by Whisper GPU memory)
Per-user: max 5 concurrent extract jobs (prevents one user from starving others)
Global: max 100 jobs/minute enqueued (rate limit at API layer to prevent abuse)
```

#### 4.2.4 Server-Side Rate Limiting

Rate limits protect against abuse, runaway clients, and cost spikes.

```
Per-user limits (keyed by user ID):
  - Free users: 10 extracts/month (quota, not rate limit — handled by quota system)
  - Free users: max 3 extracts/minute (burst protection)
  - Paid users: max 10 extracts/minute (generous but bounded)
  - All users: max 5 concurrent in-flight extracts

Global limits:
  - max 100 extract jobs enqueued/minute across all users
  - If global limit hit, return HTTP 503 with Retry-After header

Rate limit headers returned on every response:
  X-RateLimit-Limit: 10
  X-RateLimit-Remaining: 7
  X-RateLimit-Reset: 1709251200
```

### 4.3 On-Device Processing Pipeline

This is where the real magic happens for Tier 1 devices. By doing transcription and extraction on-device, we keep costs at the absolute minimum (just the scrape) and avoid sending user content to our servers.

#### 4.3.1 Full On-Device Pipeline

```
Step 1: Download video to temp directory
         │
Step 2: Extract audio track only (discard video track)
         │
Step 3: Run speech-to-text on audio
         │
Step 4: Extract key frames at 1fps
         │
Step 5: Run OCR on each key frame
         │
Step 6: Deduplicate + combine all text sources
         │
Step 7: Run on-device LLM extraction
         │
Step 8: Clean up all temp files
```

**Step 1 — Video Download:**
Stream the video URL to a temp file. Don't load the whole thing into memory.
```
iOS:  URLSession.shared.download(from: videoURL) → writes to tmp/
Android: OkHttp streaming download → context.cacheDir

Max file size: 100MB (reject anything larger — it's not a typical Reel/TikTok)
Validate MIME type before processing: must be video/mp4, video/quicktime, or video/webm
```

**Step 2 — Audio Extraction (discard video track to save storage):**
```
iOS:
  let asset = AVURLAsset(url: tempVideoURL)
  let audioTrack = asset.tracks(withMediaType: .audio).first
  let exportSession = AVAssetExportSession(asset: asset, presetName: AVAssetExportPresetAppleM4A)
  exportSession.outputFileType = .m4a
  // Exports audio-only file (~1/10th the size of the video)
  // Delete the video file immediately after audio extraction

Android:
  val extractor = MediaExtractor()
  extractor.setDataSource(tempVideoPath)
  // Select audio track, write to temp .m4a file via MediaMuxer
  // Delete video file immediately after extraction
```

**Why discard the video track:** A 60-second Reel at 1080p is ~15-30MB. The audio track alone is ~1-3MB. We only need the video file briefly for key frame extraction (Step 4), then we can delete it. Sequence: extract audio → extract key frames → delete video file → process audio and frames.

**Step 3 — Speech-to-Text:**

*iOS (two paths):*
```
iOS 26+ (SpeechAnalyzer — preferred):
  let analyzer = SpeechAnalyzer()
  let result = try await analyzer.transcribe(audioURL)
  // SpeechAnalyzer automatically detects language (no need to specify locale)
  // Returns timestamped segments — useful for syncing with key frames later
  // Runs fully on-device with Apple's latest speech model

iOS 17–25 (SFSpeechRecognizer — fallback):
  let recognizer = SFSpeechRecognizer()  // defaults to device locale
  let request = SFSpeechURLRecognitionRequest(url: audioURL)
  request.requiresOnDeviceRecognition = true  // CRITICAL: stay on-device
  request.shouldReportPartialResults = false
  // If on-device model not available for this language, fall back to server transcription
  // SFSpeechRecognizer has a 1-minute audio limit per request — for longer videos,
  // split audio into 55-second chunks with 5-second overlap, then merge transcripts
```

*Android:*
```
ML Kit Speech Recognition:
  val options = SpeechRecognizerOptions.Builder()
      .setResultType(SpeechRecognizerOptions.RESULT_TYPE_FINAL)
      .build()
  val recognizer = SpeechRecognition.getClient(options)
  // Process audio file through recognizer
  // ML Kit handles language detection automatically
  // For videos > 1 minute, chunk audio the same way as iOS SFSpeechRecognizer
```

**Step 4 — Key Frame Extraction:**
```
Extract frames at 1fps (one frame per second of video).
A 60-second Reel = 60 frames to process.

iOS:
  let generator = AVAssetImageGenerator(asset: asset)
  generator.requestedTimeToleranceBefore = .zero
  generator.requestedTimeToleranceAfter = .zero
  generator.maximumSize = CGSize(width: 1280, height: 1280)  // downscale for OCR
  // Generate CGImage at each 1-second interval

Android:
  val retriever = MediaMetadataRetriever()
  retriever.setDataSource(videoPath)
  // getFrameAtTime(timeUs, OPTION_CLOSEST_SYNC) at each 1-second interval
  // Scale to max 1280px on the long edge
```

**Why 1fps:** Most recipe/travel/product Reels show each text overlay for 2-5 seconds. 1fps captures every overlay at least twice, which is enough for deduplication. Going higher (e.g., 5fps) would 5x the OCR work for negligible quality improvement.

**Step 5 — OCR on Key Frames:**
```
iOS:
  let request = VNRecognizeTextRequest()
  request.recognitionLevel = .accurate  // not .fast — we want quality
  request.usesLanguageCorrection = true
  request.recognitionLanguages = ["en", "es", "fr", "de", "it", "pt", "ja", "ko", "zh-Hans"]
  let handler = VNImageRequestHandler(cgImage: frame)
  try handler.perform([request])

Android:
  val recognizer = TextRecognition.getClient(TextRecognizerOptions.DEFAULT_OPTIONS)
  // For non-Latin scripts: use TextRecognition.getClient(ChineseTextRecognizerOptions...)
  val result = recognizer.process(InputImage.fromBitmap(frame, 0))
```

**Text Deduplication:**
OCR across 60 frames will produce massive duplication (same overlay captured 3-5 times). Deduplicate:
```
1. Normalize each frame's OCR output (lowercase, strip whitespace)
2. Compare each frame's text to previous frame using Levenshtein distance
3. If similarity > 80%, discard the duplicate
4. Keep only unique text blocks in chronological order
```

**Step 6 — Combine All Text Sources:**
```
combinedText = """
[CAPTION]
{original caption from scrape}

[TRANSCRIPT]
{speech-to-text output}

[ON-SCREEN TEXT]
{deduplicated OCR text from key frames, in chronological order}
"""
```

This combined text is what gets fed into the LLM extraction step. Three complementary signals: what the creator wrote (caption), what they said (transcript), and what they showed on screen (OCR).

**Step 7 — On-Device LLM Extraction:**
```
iOS:  Foundation Models framework (Apple Intelligence) or CoreML model
Android: Gemini Nano via Google AI Edge SDK

Both: Run the same extraction prompt used on the server side (see F4),
      but adapted for the smaller on-device model's context window.
```

**Step 8 — Temp File Cleanup:**
```
Delete ALL temp files immediately after processing:
  - Downloaded video file (should already be deleted after Step 2/4)
  - Extracted audio file
  - Key frame images (delete each after OCR, don't accumulate all 60 in memory)
  - Any intermediate processing artifacts

Use a finally/defer block to guarantee cleanup even if processing crashes.
Files must never persist beyond 60 seconds of the processing completing or failing.
```

#### 4.3.2 Memory Management

On-device processing is memory-intensive. Key frames, audio buffers, and ML models all compete for RAM.

```
iOS Share Extension: hard memory limit of 120MB
  - Share extension should NOT run the full pipeline
  - Share extension only captures the URL and metadata
  - Full pipeline runs in the main app or via background processing extension

Main App Processing: target < 300MB peak memory
  - Process key frames one at a time (don't load all 60 into memory)
  - Release each CGImage/Bitmap immediately after OCR
  - Use autoreleasepool (iOS) / explicit bitmap.recycle() (Android) aggressively
  - Monitor memory pressure: if approaching limit, stop key frame OCR early
    and work with whatever frames were already processed
  - Audio file is streamed to disk, not held in memory
  - LLM inference: Foundation Models manages its own memory budget;
    Gemini Nano has a ~50MB runtime footprint
```

**Memory Monitoring:**
```swift
// iOS — check available memory before starting pipeline
let availableMemory = os_proc_available_memory()
if availableMemory < 200_000_000 { // < 200MB available
    // Fall back to server-side processing instead of risking OOM
    return .serverFallback(reason: .lowMemory)
}
```

#### 4.3.3 Battery Considerations

Deep extract is a heavy on-device workload: video download, audio decoding, speech recognition, OCR (60 frames), and LLM inference. On a low battery, this could push the device into low-power mode mid-processing.

```
If device battery < 20%:
  Show warning: "This will use significant battery. Extract now or wait until charged?"
  Options: "Extract Now" / "Remind Me Later"

If device battery < 5%:
  Block on-device processing entirely
  Fall back to full server-side processing (if user has quota)
  Message: "Battery too low for on-device analysis. We'll process this on our servers instead."

If device enters Low Power Mode mid-processing:
  Continue current extract (don't abort mid-stream)
  But don't start new batch extracts
```

### 4.4 Cost Optimization

**URL Deduplication Cache:**
```
Before processing, check: has this URL been deep-extracted before?
  - By this user → return cached result instantly ($0.00)
  - By any user → return cached result ($0.00)
  - Never → process and cache result
```

Viral Reels (saved by hundreds of users) are processed once. Cache TTL: 30 days.

**Conditional server transcription:**
```
if device_can_transcribe (Tier 1 device):
    Server returns video_url only → device transcribes
    Cost: $0.002-0.003 (scrape only)
else:
    Server transcribes + extracts
    Cost: $0.006-0.011 (scrape + whisper + LLM)
```

---

## 5. Cost Analysis

### 5.1 Per-Extract Cost Breakdown

Each extract follows one of three processing paths depending on device capability. Here's the cost breakdown for each:

| Component | Path A (on-device everything) | Path B (server transcription, on-device extraction) | Path C (full server) |
|-----------|-------------------------------|-----------------------------------------------------|----------------------|
| Scrape API | $0.002–0.003 | $0.002–0.003 | $0.002–0.003 |
| Whisper transcription | $0.000 (on-device) | $0.002–0.003 (server Whisper) | $0.002–0.003 |
| LLM extraction | $0.000 (on-device) | $0.000 (on-device) | $0.002–0.003 |
| Server compute (CPU/GPU) | negligible | $0.001–0.002 | $0.002–0.003 |
| **Total per extract** | **$0.002–0.003** | **$0.005–0.008** | **$0.008–0.012** |

**Path A** is the goal for the majority of users. Modern iPhones (A15+) and flagship Androids (Tensor G2+, Snapdragon 8 Gen 2+) can handle the full pipeline. Path C is the fallback for older devices and the server-side safety net.

### 5.2 Monthly Cost Projections

Assumptions:
- Average 8 extracts/user/month (free users average 5, paid users average 15)
- Device capability split: 60% Path A, 25% Path B, 15% Path C
- Redis cache hit rate: 20% at 1K users, scaling to 30% at 100K users (viral content reuse)
- Paid user ratio: 5% at 1K users, 8% at 100K users

| Scale | Total Extracts/Month | Cache Hits | Billable Extracts | Weighted Avg Cost | Monthly Server Cost | Revenue (Pro subs) |
|-------|---------------------|------------|-------------------|-------------------|--------------------|--------------------|
| 1K users | 8,000 | 1,600 (20%) | 6,400 | ~$0.004 | **$26** | $250 (50 subs) |
| 10K users | 80,000 | 20,000 (25%) | 60,000 | ~$0.004 | **$240** | $4,000 (800 subs) |
| 100K users | 800,000 | 240,000 (30%) | 560,000 | ~$0.0038 | **$2,128** | $40,000 (8,000 subs) |

**Key takeaway:** Even at 100K users, deep extract server costs are ~$2.1K/month against ~$40K/month in subscription revenue. The on-device-first strategy keeps the cost ratio extremely healthy. The biggest cost risk is if Path A device adoption is lower than 60% — monitor the `processing_path` analytics event closely.

### 5.3 Cache Economics

The URL deduplication cache is the single biggest cost saver at scale.

```
Cache implementation: Redis with URL hash as key
Cache TTL: 30 days
Cache value: full extracted result JSON (typically 2-5KB)
Cache storage at 100K users: ~500K unique URLs × 4KB avg = ~2GB Redis (easily fits in memory)

Viral content effect:
  - A single viral Reel saved by 1,000 users = 1 scrape + 999 cache hits
  - At $0.003/scrape, that's $0.003 instead of $3.00
  - 1000x cost reduction on popular content
```

---

## 6. Quota Management

### 6.1 Data Model

```
UserQuota {
    userId: String
    monthYear: String          // "2026-02"
    deepExtractsUsed: Int      // Current count
    deepExtractsLimit: Int     // 10 for free, MAX_INT for paid
    resetDate: Date            // First of next month
}
```

### 6.2 Quota Check Flow

```swift
func canPerformDeepExtract(user: User) -> QuotaResult {
    if user.isPro { return .allowed }

    let quota = getQuota(user: user, month: currentMonth)
    if quota.deepExtractsUsed < quota.deepExtractsLimit {
        return .allowed(remaining: quota.deepExtractsLimit - quota.deepExtractsUsed)
    }
    return .limitReached(resetsOn: quota.resetDate)
}
```

### 6.3 Quota Display

Show remaining count in three places:
1. On the "Get Full Details" button: `"7/10 free extracts left"`
2. In Settings > Account: full quota breakdown
3. In the confirmation dialog before extraction

---

## 7. Security

### 7.1 Server-Side URL Validation (SSRF Prevention)

Before passing any URL to the scraper, validate it server-side to prevent SSRF (Server-Side Request Forgery) attacks. A malicious client could submit `http://169.254.169.254/latest/meta-data/` or `http://localhost:6379/` instead of an Instagram URL.

```
Allowed domains (allowlist — reject everything else):
  - instagram.com, www.instagram.com
  - tiktok.com, www.tiktok.com, vm.tiktok.com
  - youtube.com, www.youtube.com, youtu.be (future)

Validation steps:
  1. Parse URL — reject if malformed
  2. Resolve hostname — reject if it resolves to private IP ranges
     (10.x.x.x, 172.16-31.x.x, 192.168.x.x, 127.x.x.x, 169.254.x.x)
  3. Check against domain allowlist — reject if not on list
  4. Reject URLs with non-standard ports (only 80 and 443 allowed)
  5. Reject URLs with authentication credentials in the URL (user:pass@host)
```

### 7.2 Rate Limiting & Quota Integrity

```
Server-side quota enforcement (don't trust the client):
  - Quota check happens server-side before enqueuing the job
  - Quota decrement happens atomically via Redis INCR after successful completion
  - Race condition prevention: use Redis WATCH/MULTI for check-and-increment
  - Client quota display is informational only — server is the source of truth

Per-user rate limiting (prevent quota manipulation):
  - Keyed by authenticated user ID (not IP, which can be shared)
  - Free users: max 3 requests/minute, max 10/month
  - Paid users: max 10 requests/minute, no monthly cap
  - Return HTTP 429 with Retry-After header when exceeded
```

### 7.3 Video File Validation

Before processing any downloaded video file (on-device or server-side):

```
1. Check MIME type from HTTP Content-Type header:
   Allowed: video/mp4, video/quicktime, video/webm, video/3gpp
   Reject anything else (prevents downloading executables, zip bombs, etc.)

2. Check file size:
   Max: 100MB
   Typical Reel/TikTok: 5-30MB
   If Content-Length header exceeds 100MB, abort download before it starts

3. Validate video container:
   After download, verify the file is actually a valid video container
   iOS: AVAsset.load(.isPlayable) — must return true
   Android: MediaMetadataRetriever.setDataSource() — must not throw

4. Check video duration:
   Max: 10 minutes (600 seconds)
   Process first 3 minutes only for videos > 3 minutes (see Edge Cases)
```

### 7.4 Temp File Security

```
All temp files follow strict lifecycle rules:

1. Created in platform-specific temp directories only:
   iOS: FileManager.default.temporaryDirectory (sandboxed, auto-cleaned by OS)
   Android: context.cacheDir (sandboxed, can be cleared by OS under pressure)

2. Named with UUID to prevent path traversal attacks:
   Bad:  /tmp/{user-provided-filename}.mp4
   Good: /tmp/extract-{UUID}.mp4

3. Deleted within 60 seconds of processing completion or failure
   - Use defer/finally blocks to guarantee cleanup
   - Register a background cleanup task that runs every 5 minutes
     and deletes any extract temp files older than 2 minutes (safety net)

4. Never persisted to permanent storage
   - No temp file should survive an app restart
   - On app launch, sweep temp directory for stale extract files and delete them

5. Server temp files follow the same rules:
   - Stored in /tmp with restrictive permissions (0600)
   - Cleaned up in the job's finally block
   - Cron job sweeps /tmp/extract-* files older than 5 minutes
```

---

## 8. Quality Assurance

### 8.1 Extraction Quality Measurement

The extraction is only as good as the structured data it produces. A transcript is useless if the LLM hallucinates ingredients or misidentifies a travel destination as a product.

**Human Evaluation (weekly):**
```
Sample: 100 random extracts per week, stratified by category:
  - 30 recipes, 25 travel, 20 products, 15 fitness, 10 other

Evaluators score each extract on:
  1. Category correctness: Did it pick the right category? (binary)
  2. Field completeness: What % of relevant fields were populated? (0-100%)
  3. Field accuracy: Of populated fields, what % are factually correct? (0-100%)
  4. Hallucination rate: Did the LLM invent data not present in the source? (binary per field)

Target quality bar:
  - Category correctness: > 95%
  - Field completeness: > 80%
  - Field accuracy: > 90%
  - Hallucination rate: < 5% of fields
```

### 8.2 Automated Quality Metrics

Track these continuously (not just in weekly reviews):

| Metric | Definition | Target | Alert Threshold |
|--------|-----------|--------|-----------------|
| Field completeness rate | Avg % of non-null fields in extracted data | > 80% | < 70% (7-day rolling) |
| Field accuracy rate | Measured via human eval sample | > 90% | < 85% |
| Category change rate | % of extracts where category changed from initial save | 30–50% (healthy) | > 70% (model may be over-classifying) |
| Category agreement rate | % where deep extract category = initial lightweight category | 50–70% | < 40% (models disagree too much) |
| Empty extraction rate | % of extracts returning zero useful fields | < 5% | > 10% |
| Processing success rate | % of extracts that complete without error | > 95% | < 90% |

### 8.3 A/B Testing LLM Providers

Run continuous A/B tests on the server-side extraction LLM to find the best quality/cost tradeoff.

```
A/B test setup:
  - Control: Claude Haiku 4.5 (current default)
  - Variant: GPT-4o-mini
  - Split: 50/50 by user ID hash (deterministic — same user always gets same provider)
  - Duration: 2 weeks minimum, 1,000+ extracts per arm minimum

Metrics to compare:
  - Field completeness rate (primary)
  - Field accuracy rate (via human eval subsample)
  - Hallucination rate
  - Latency (p50, p95)
  - Cost per extract
  - User re-extraction rate (proxy for dissatisfaction: user triggered "Try Again")

Decision criteria:
  Winner must be statistically significant (p < 0.05) on field completeness
  AND not regress on hallucination rate
  AND cost difference < 2x
```

### 8.4 Prompt Versioning & Rollback

The extraction prompt is effectively a core piece of business logic. Treat it like code.

```
Prompt versioning:
  - Each prompt version gets a semantic version: v1.0, v1.1, v2.0
  - Stored in a dedicated prompt registry (database table or config file)
  - Every extraction result records the prompt_version used
  - This lets us compare quality across prompt versions retroactively

Rollback strategy:
  - Keep the previous 3 prompt versions available for instant rollback
  - If quality metrics drop after a prompt update:
    1. Automated alert fires (see 8.2 thresholds)
    2. On-call engineer reviews dashboard
    3. Rollback is a config change (set active_prompt_version = previous version)
    4. No code deploy required — prompt version is read from config at runtime

Prompt change process:
  1. Draft new prompt version
  2. Run against 50 held-out test cases (golden set with known-correct outputs)
  3. Compare field completeness and accuracy vs current prompt
  4. If improvement > 2% on completeness with no accuracy regression, deploy to 10% traffic
  5. Monitor for 48 hours, then roll out to 100%
```

---

## 9. Analytics Events

Track every meaningful moment in the deep extract lifecycle. These events power the quality dashboard, cost monitoring, and conversion funnels.

### 9.1 Event Definitions

**`deep_extract_initiated`**
Fired when user triggers a deep extract (tap, batch, or auto).
```json
{
  "event": "deep_extract_initiated",
  "properties": {
    "trigger_point": "get_full_details" | "batch" | "auto_save" | "share_extension",
    "is_batch": false,
    "batch_size": 1,
    "is_auto": false,
    "platform": "instagram" | "tiktok",
    "is_pro": true,
    "quota_remaining": null,
    "device_can_transcribe": true
  }
}
```

**`deep_extract_stage_reached`**
Fired each time a new processing stage begins. One event per stage transition.
```json
{
  "event": "deep_extract_stage_reached",
  "properties": {
    "stage": "fetching" | "transcribing" | "ocr" | "extracting" | "complete",
    "duration_ms": 3200,
    "cumulative_duration_ms": 8400,
    "processing_path": "on_device" | "server" | "hybrid"
  }
}
```

**`deep_extract_completed`**
Fired when an extract finishes successfully.
```json
{
  "event": "deep_extract_completed",
  "properties": {
    "total_duration_ms": 18500,
    "category_before": "uncategorized",
    "category_after": "recipe",
    "richness_before": 0.15,
    "richness_after": 0.92,
    "processing_path": "on_device" | "server" | "hybrid",
    "cache_hit": false,
    "scraper_provider": "apify",
    "prompt_version": "v1.2",
    "llm_provider": "claude_haiku_4_5",
    "fields_populated": 12,
    "transcript_length_chars": 850,
    "ocr_frames_processed": 58,
    "ocr_unique_texts": 14
  }
}
```

**`deep_extract_failed`**
Fired when an extract fails after all retries.
```json
{
  "event": "deep_extract_failed",
  "properties": {
    "stage": "fetching",
    "error_type": "scraper_timeout" | "video_private" | "video_deleted" | "transcription_failed" | "llm_error" | "timeout" | "oom",
    "scraper_provider": "apify",
    "retry_count": 2,
    "duration_ms": 62000
  }
}
```

**`deep_extract_cancelled`**
Fired when user cancels an in-progress extract.
```json
{
  "event": "deep_extract_cancelled",
  "properties": {
    "stage_when_cancelled": "transcribing",
    "duration_ms": 8200,
    "is_batch": false
  }
}
```

**`quota_checked`**
Fired when the app checks the user's remaining quota (on app launch, before extract, etc.).
```json
{
  "event": "quota_checked",
  "properties": {
    "remaining": 7,
    "used": 3,
    "limit": 10,
    "is_pro": false,
    "days_remaining_in_month": 12
  }
}
```

**`quota_exhausted_shown`**
Fired when the quota exhausted dialog is displayed to a free user.
```json
{
  "event": "quota_exhausted_shown",
  "properties": {
    "remaining_days_in_month": 12,
    "extracts_used_this_month": 10
  }
}
```

**`paywall_upgrade_tapped`**
Fired when user taps the upgrade/unlock button from any deep extract context.
```json
{
  "event": "paywall_upgrade_tapped",
  "properties": {
    "source": "quota_exhausted" | "get_full_details" | "batch_limit" | "settings"
  }
}
```

### 9.2 Key Dashboards to Build

These events feed three critical dashboards:

1. **Extract Funnel:** `initiated` → `stage_reached(fetching)` → `stage_reached(complete)` → `completed`. Where are users dropping off? What's the completion rate?
2. **Cost Monitor:** `completed` grouped by `processing_path` and `scraper_provider`. Track weighted average cost per extract, daily and monthly.
3. **Monetization Funnel:** `quota_exhausted_shown` → `paywall_upgrade_tapped` → (in-app purchase event from billing system). What's the conversion rate from exhaustion to upgrade?

---

## 10. Edge Cases

| Scenario | Handling |
|----------|----------|
| Video is private or deleted | Return error, don't count against quota, offer "Try Again" |
| Video is very long (5+ minutes) | Transcribe first 3 minutes only. Note: "Analyzed first 3 minutes" |
| Video has no speech (music only) | Rely on OCR text from key frames + full caption from scrape |
| Scraper API is down | Retry with backup provider (Apify → ScrapeCreators → Bright Data) |
| User closes app during processing | Continue via background task. Card updates when user returns. |
| Network drops mid-process | Resume from last completed stage. Don't re-download video if already transcribed. |
| Deep extract returns same category but better data | Merge new data into existing card (don't overwrite user edits) |
| User triggers extract on already-complete card | Show "This save already has full details. Extract again?" |
| Batch extract with 5+ items | Queue sequentially, show batch progress: "Extracting 3 of 5..." |

---

## 11. Development Plan

### 11.1 Tasks & Estimates

| # | Task | Layer | Effort | Dependencies |
|---|------|-------|--------|-------------|
| 1 | Build extract API endpoint with SSE streaming | Server | 6h | — |
| 2 | Integrate Apify scraper API | Server | 4h | Task 1 |
| 3 | Integrate fallback scrapers (ScrapeCreators, Bright Data) | Server | 4h | Task 2 |
| 4 | Implement scraper circuit breaker + health checks | Server | 4h | Task 3 |
| 5 | Build job queue (BullMQ) with priority and DLQ | Server | 6h | Task 1 |
| 6 | Build server-side Whisper transcription pipeline | Server | 4h | Task 5 |
| 7 | Build server-side LLM extraction (Claude Haiku / GPT-4o-mini) | Server | 4h | Task 6 |
| 8 | Implement URL deduplication cache (Redis) | Server | 3h | Task 1 |
| 9 | Implement server-side rate limiting + URL validation | Server | 3h | Task 1 |
| 10 | Build on-device transcription pipeline (SpeechAnalyzer + SFSpeechRecognizer fallback) | iOS | 6h | — |
| 11 | Build on-device transcription pipeline (ML Kit Speech) | Android | 6h | — |
| 12 | Build on-device video download + AVFoundation audio extraction | iOS | 4h | Task 10 |
| 13 | Build on-device video download + MediaExtractor audio extraction | Android | 4h | Task 11 |
| 14 | Build key frame extraction + OCR pipeline (Vision) | iOS | 5h | Task 12 |
| 15 | Build key frame extraction + OCR pipeline (ML Kit TextRecognition) | Android | 5h | Task 13 |
| 16 | Build text deduplication + combination logic | Both | 3h | Tasks 14–15 |
| 17 | Build on-device AI extraction from combined text | Both | 4h | Task 16 |
| 18 | Implement memory management + battery checks | Both | 3h | Tasks 10–17 |
| 19 | Build SSE client + progress UI | Both | 4h | Task 1 |
| 20 | Build "Get Full Details" button + confirmation dialog | Both | 3h | — |
| 21 | Implement quota management (check, increment, reset) | Server + Client | 4h | — |
| 22 | Build quota exhausted paywall UI | Both | 3h | Task 21 |
| 23 | Build error states + retry logic | Both | 3h | Tasks 1–17 |
| 24 | Build background processing (app closed during extract) | Both | 4h | — |
| 25 | Implement card transition animation (generic → structured) | Both | 3h | F4 |
| 26 | Implement analytics events | Both + Server | 4h | All |
| 27 | Build prompt versioning + A/B test infrastructure | Server | 4h | Task 7 |
| 28 | QA: full pipeline testing across platforms and scraper states | QA | 10h | All |

**Total: ~127 hours (~5 weeks)**

### 11.2 Definition of Done

- [ ] "Get Full Details" triggers server scrape + on-device transcription pipeline
- [ ] SSE progress updates display meaningful stages on client
- [ ] On-device transcription works on Tier 1 devices (SpeechAnalyzer / ML Kit)
- [ ] SFSpeechRecognizer fallback works on iOS 17–25 devices
- [ ] Server fallback transcription works for Tier 2 devices
- [ ] Key frame extraction at 1fps with OCR and text deduplication working
- [ ] Combined text (caption + transcript + OCR) fed into extraction
- [ ] URL dedup cache prevents reprocessing same URL
- [ ] Free user quota enforced correctly (10/month, resets on 1st)
- [ ] Failed extracts don't count against quota
- [ ] Paywall shown when quota exhausted
- [ ] Background processing completes even if app is closed
- [ ] Card animates from generic to structured template on completion
- [ ] Processing completes in < 30 seconds for typical 60-sec Reel
- [ ] Fallback scraper chain works (primary → secondary → tertiary)
- [ ] Circuit breaker trips after 3 failures and recovers after 15 minutes
- [ ] Job queue processes with correct priority ordering (paid > free)
- [ ] Rate limiting prevents abuse (per-user and global)
- [ ] URL validation blocks SSRF attempts
- [ ] Video file validation enforces MIME type and 100MB size limit
- [ ] Temp files cleaned up within 60 seconds of processing
- [ ] Memory stays under 300MB peak during on-device processing
- [ ] Battery warning shown when device < 20%
- [ ] All analytics events firing correctly
- [ ] Prompt versioning allows rollback without code deploy
