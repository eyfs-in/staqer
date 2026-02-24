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

### 4.3 Cost Optimization

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

## 5. Quota Management

### 5.1 Data Model

```
UserQuota {
    userId: String
    monthYear: String          // "2026-02"
    deepExtractsUsed: Int      // Current count
    deepExtractsLimit: Int     // 10 for free, MAX_INT for paid
    resetDate: Date            // First of next month
}
```

### 5.2 Quota Check Flow

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

### 5.3 Quota Display

Show remaining count in three places:
1. On the "Get Full Details" button: `"7/10 free extracts left"`
2. In Settings > Account: full quota breakdown
3. In the confirmation dialog before extraction

---

## 6. Edge Cases

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

## 7. Development Plan

### 7.1 Tasks & Estimates

| # | Task | Layer | Effort | Dependencies |
|---|------|-------|--------|-------------|
| 1 | Build extract API endpoint with SSE streaming | Server | 6h | — |
| 2 | Integrate Apify scraper API | Server | 4h | Task 1 |
| 3 | Integrate fallback scrapers (ScrapeCreators, Bright Data) | Server | 4h | Task 2 |
| 4 | Build server-side Whisper transcription pipeline | Server | 4h | Task 1 |
| 5 | Build server-side LLM extraction (Claude Haiku / GPT-4o-mini) | Server | 4h | Task 4 |
| 6 | Implement URL deduplication cache (Redis) | Server | 3h | Task 1 |
| 7 | Build on-device transcription pipeline (SpeechAnalyzer) | iOS | 6h | — |
| 8 | Build on-device transcription pipeline (ML Kit Speech) | Android | 6h | — |
| 9 | Build on-device video download + AVFoundation audio extraction | iOS | 4h | Task 7 |
| 10 | Build on-device video download + MediaExtractor audio extraction | Android | 4h | Task 8 |
| 11 | Build on-device AI extraction from transcript | Both | 4h | Tasks 7–10 |
| 12 | Build SSE client + progress UI | Both | 4h | Task 1 |
| 13 | Build "Get Full Details" button + confirmation dialog | Both | 3h | — |
| 14 | Implement quota management (check, increment, reset) | Server + Client | 4h | — |
| 15 | Build quota exhausted paywall UI | Both | 3h | Task 14 |
| 16 | Build error states + retry logic | Both | 3h | Tasks 1–11 |
| 17 | Build background processing (app closed during extract) | Both | 4h | — |
| 18 | Implement card transition animation (generic → structured) | Both | 3h | F4 |
| 19 | QA: full pipeline testing across platforms and scraper states | QA | 8h | All |

**Total: ~85 hours (~3.5 weeks)**

### 7.2 Definition of Done

- [ ] "Get Full Details" triggers server scrape + on-device transcription pipeline
- [ ] SSE progress updates display meaningful stages on client
- [ ] On-device transcription works on Tier 1 devices (SpeechAnalyzer / ML Kit)
- [ ] Server fallback transcription works for Tier 2 devices
- [ ] URL dedup cache prevents reprocessing same URL
- [ ] Free user quota enforced correctly (10/month, resets on 1st)
- [ ] Failed extracts don't count against quota
- [ ] Paywall shown when quota exhausted
- [ ] Background processing completes even if app is closed
- [ ] Card animates from generic to structured template on completion
- [ ] Processing completes in < 30 seconds for typical 60-sec Reel
- [ ] Fallback scraper chain works (primary → secondary → tertiary)
