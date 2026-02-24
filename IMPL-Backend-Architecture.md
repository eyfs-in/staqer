
# Staqer Backend Architecture Plan

**Version:** 1.0  
**Date:** February 24, 2026  
**Status:** Implementation Blueprint  
**Target:** Senior Backend Developer  

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Tech Stack Decision](#2-tech-stack-decision)
3. [Service Architecture](#3-service-architecture)
4. [API Design](#4-api-design)
5. [Deep Extract Pipeline](#5-deep-extract-pipeline)
6. [Scraper Service Layer](#6-scraper-service-layer)
7. [Job Queue Architecture](#7-job-queue-architecture)
8. [SSE Implementation](#8-sse-implementation)
9. [Database Design](#9-database-design)
10. [Caching Strategy](#10-caching-strategy)
11. [Authentication & Authorization](#11-authentication--authorization)
12. [Subscription & Quota Management](#12-subscription--quota-management)
13. [Infrastructure](#13-infrastructure)
14. [Monitoring & Observability](#14-monitoring--observability)
15. [Security](#15-security)
16. [Cost Analysis](#16-cost-analysis)
17. [Sprint-by-Sprint Breakdown](#17-sprint-by-sprint-breakdown)

---

## 1. Executive Summary

Staqer's backend is designed as a **monolith-first, API-driven Node.js/TypeScript service** optimized for cost efficiency and rapid iteration. The architecture prioritizes on-device processing (60-70% of saves cost $0.00 server-side) while providing robust server-assisted pipelines for deep content extraction.

### Core Design Principles

1. **Cost-First Architecture**: Target <$0.05/free user/month, <$0.15/paid user/month
2. **On-Device Maximization**: Server only handles what clients can't (video scraping, fallback transcription)
3. **Monolith with Service Boundaries**: Clean internal modules ready for microservice extraction if needed
4. **Async Everything**: Job queues for heavy work, SSE for real-time progress
5. **Cache-Aggressive**: Redis deduplication cache saves 10-35x on viral content

### Key Metrics

| Metric | Target | Critical Threshold |
|--------|--------|-------------------|
| Deep Extract Latency | < 30s (p95) | > 45s |
| API Response Time | < 200ms (p95) | > 500ms |
| Job Queue Throughput | 100 jobs/min | < 50 jobs/min |
| Cache Hit Rate | > 25% | < 15% |
| Server Cost per Extract | ~$0.004 | > $0.01 |
| Uptime | > 99.5% | < 99% |

---

## 2. Tech Stack Decision

### 2.1 Runtime: Node.js + TypeScript

**Decision: Node.js 20 LTS + TypeScript 5.3**

**Rationale:**
- **NPM Ecosystem**: Best-in-class libraries for video processing (fluent-ffmpeg), scraping (Apify SDK), job queues (BullMQ)
- **TypeScript Safety**: Strong typing prevents entire classes of bugs at compile time, critical for data transformation pipelines
- **Async-Native**: Event loop model fits the I/O-bound workload (API calls, database queries, scraper integration)
- **Team Velocity**: Single language for mobile (Kotlin/Swift) + shared KMP logic + backend reduces context switching
- **Supabase Integration**: First-class support via `@supabase/supabase-js` SDK

**Alternatives Considered:**
- **Python + FastAPI**: Excellent ML/data ecosystem, but slower cold starts, weaker type safety, fewer video processing libraries
- **Go**: Fastest runtime, lowest memory, but smaller ecosystem for video/scraping, steeper learning curve
- **Rust**: Maximum performance, but development velocity would be 2-3x slower for an MVP

### 2.2 Framework: Express.js

**Decision: Express.js 4.x**

**Rationale:**
- **Minimalist**: Doesn't impose structure, easy to wrap with our own abstractions
- **Middleware Ecosystem**: Rich library of logging, auth, validation, rate limiting middleware
- **SSE Support**: Native support for Server-Sent Events via `res.write()` streaming
- **Battle-Tested**: 10+ years of production hardening

**Alternatives Considered:**
- **Fastify**: 2x faster, but smaller ecosystem, not worth the migration cost from Express
- **NestJS**: Opinionated structure good for large teams, overkill for MVP, longer onboarding

### 2.3 Database: Supabase (PostgreSQL)

**Decision: Supabase PostgreSQL with Row-Level Security (RLS)**

**Rationale:**
- **Managed Postgres**: Production-grade Postgres without ops overhead
- **Built-in Auth**: Supabase Auth integrates seamlessly with Apple/Google Sign-In
- **Realtime Subscriptions**: Supabase Realtime uses Postgres logical replication for live data sync (future multi-device support)
- **Storage**: Supabase Storage for future thumbnail caching, backed by S3
- **Free Tier**: Generous limits for development and early users (500MB database, 50K monthly active users)

**Schema Design Philosophy:**
- **Normalized**: Avoid duplication, use foreign keys, enforce referential integrity
- **JSONB for Flexibility**: `extracted_data` as JSONB allows schema evolution without migrations
- **Indexes**: B-tree on lookup keys, GIN indexes on JSONB fields for fast queries
- **RLS Policies**: Enforce data isolation at database level (users can only read/write their own saves)

### 2.4 Job Queue: BullMQ + Redis

**Decision: BullMQ 5.x backed by Redis 7.x**

**Rationale:**
- **Persistent**: Jobs survive crashes via Redis AOF persistence
- **Priority Queues**: Paid users' extracts jump ahead of free users
- **Dead Letter Queue (DLQ)**: Failed jobs don't disappear, can be retried or debugged
- **Observability**: Bull Board provides web UI for queue monitoring
- **Retries & Backoff**: Exponential backoff for transient failures
- **Rate Limiting**: Per-queue rate limits prevent scraper API abuse

**Redis Use Cases:**
1. BullMQ job queue storage
2. URL deduplication cache (extraction results)
3. Rate limiting state (user quotas, per-endpoint limits)
4. Session storage (future multi-device support)

### 2.5 Serverless Compute: Supabase Edge Functions

**Decision: Supabase Edge Functions (Deno runtime) for lightweight tasks**

**Use Cases:**
- **Webhook Handlers**: RevenueCat subscription events, push notification dispatch
- **URL Metadata Enrichment**: Fetch Open Graph tags, expand short URLs
- **Cron Jobs**: Daily quota resets, monthly usage reports
- **Push Notification Dispatch**: Trigger FCM/APNS after async job completion

**Why Edge Functions over always-on server:**
- **Cost**: $0.00 at low volume, $2/million invocations vs. $15-50/month for always-on Heroku dyno
- **Cold Start**: <200ms, acceptable for non-critical paths
- **Global Distribution**: Deno Deploy runs on Cloudflare edge, low latency worldwide

**What stays in the main Node.js service:**
- Deep extract pipeline (CPU/memory intensive, needs long-running workers)
- Core API endpoints (auth, saves CRUD, search)
- Job queue workers

---

## 3. Service Architecture

### 3.1 Monolith-First Approach

**Decision: Single Node.js service with clean module boundaries, NOT microservices**

**Rationale:**
- **Velocity**: Microservices add coordination overhead, slow down early iteration
- **Shared Code**: Scraper abstraction, URL normalization, extraction prompts are shared across endpoints
- **Deployment Simplicity**: Single Docker image, single CI/CD pipeline
- **Cost**: One small server < 5 VMs with orchestration overhead
- **Data Consistency**: Single database, no distributed transaction complexity

**Module Structure (logical services within the monolith):**

```
src/
├── api/                    # HTTP layer
│   ├── routes/
│   │   ├── auth.routes.ts      # /auth/* endpoints
│   │   ├── saves.routes.ts     # /saves/* endpoints
│   │   ├── extract.routes.ts   # /extract/* endpoints
│   │   ├── users.routes.ts     # /users/* quota, profile
│   │   └── webhooks.routes.ts  # /webhooks/* RevenueCat, etc.
│   ├── middlewares/
│   │   ├── auth.middleware.ts  # JWT validation
│   │   ├── rateLimit.middleware.ts
│   │   └── errorHandler.middleware.ts
│   └── validators/         # Request schema validation (Zod)
├── services/               # Business logic layer
│   ├── extraction/
│   │   ├── ExtractionService.ts
│   │   ├── OnDeviceCoordinator.ts
│   │   └── PromptManager.ts
│   ├── scrapers/
│   │   ├── ScraperFactory.ts
│   │   ├── ApifyScraper.ts
│   │   ├── ScrapeCreatorsScraper.ts
│   │   └── BrightDataScraper.ts
│   ├── processing/
│   │   ├── TranscriptionService.ts  # Server Whisper fallback
│   │   ├── LLMService.ts            # Claude/GPT extraction
│   │   └── OCRService.ts            # (future) server OCR
│   ├── quota/
│   │   ├── QuotaService.ts
│   │   └── SubscriptionService.ts
│   └── notifications/
│       └── PushService.ts
├── workers/                # Job queue processors
│   ├── extraction.worker.ts
│   ├── cleanup.worker.ts       # Temp file cleanup
│   └── quota-reset.worker.ts   # Monthly quota reset
├── database/               # Data layer
│   ├── models/
│   │   ├── User.model.ts
│   │   ├── SavedContent.model.ts
│   │   ├── Extraction.model.ts
│   │   └── Subscription.model.ts
│   ├── migrations/
│   └── supabase.client.ts
├── cache/
│   ├── redis.client.ts
│   └── CacheService.ts
├── queue/
│   ├── bullmq.client.ts
│   └── QueueManager.ts
├── utils/
│   ├── url.utils.ts        # Normalization, validation
│   ├── logger.ts
│   └── metrics.ts
└── config/
    ├── env.ts              # Environment variable validation
    └── constants.ts
```

### 3.2 Service Boundaries (Future Microservice Extraction Points)

If the monolith becomes a bottleneck (unlikely at <100K users), these modules can be extracted:

| Module | Extraction Trigger | Communication | Data Ownership |
|--------|-------------------|---------------|----------------|
| **Scraper Service** | Scraper costs dominate ($500+/month), need independent scaling | gRPC or HTTP | Scrape logs, circuit breaker state |
| **Extraction Workers** | Extract queue depth consistently >1000, need dedicated GPU workers | Redis queue (BullMQ) | Extraction job state |
| **Notification Service** | >10K push notifications/day | Redis pub/sub + webhook | Notification delivery state |

**Don't extract unless:**
- Service has clear bounded context
- Performance/cost problems proven via metrics
- Team size allows independent service ownership (2+ engineers)

### 3.3 Scaling Strategy

**Vertical Scaling First (0-10K users):**
- Start with 1 server: 2 vCPU, 4GB RAM ($20-40/month Railway)
- Bottleneck will be scraper API latency (3-8s), not CPU

**Horizontal Scaling Next (10K-100K users):**
- Add 2 more API servers behind load balancer (Railway auto-scaling or manual)
- Separate worker servers for job queue (scale workers independently from API)
- Redis Cluster (3 nodes) for cache and queue reliability

**Database Scaling (>50K users):**
- Supabase Pro plan ($25/month) with read replicas for queries
- Connection pooling via PgBouncer (already included in Supabase)
- Partition `saved_content` table by `user_id` if >1M rows

---

## 4. API Design

### 4.1 REST Principles

**Base URL:** `https://api.staq.app/v1`

**Versioning:** URL path versioning (`/v1`, `/v2`), not headers (simpler caching, easier debugging)

**Authentication:** Bearer token in `Authorization` header (JWT from Supabase Auth)

**Response Format:** JSON with consistent envelope:

```json
{
  "success": true,
  "data": { ... },
  "error": null,
  "meta": {
    "timestamp": "2026-02-24T10:30:00Z",
    "request_id": "req_abc123"
  }
}
```

**Error Format:**

```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "QUOTA_EXCEEDED",
    "message": "You've used all 10 free extracts for February.",
    "details": {
      "quota_limit": 10,
      "quota_used": 10,
      "reset_date": "2026-03-01T00:00:00Z"
    }
  },
  "meta": { ... }
}
```

### 4.2 Core Endpoints

#### 4.2.1 Authentication

```
POST /auth/signup
POST /auth/signin
POST /auth/signout
POST /auth/refresh
GET  /auth/me
```

**Implementation:** Supabase Auth handles all auth logic. Backend validates JWTs and extracts `user_id`.

#### 4.2.2 Saves Management

```
GET    /saves                    # List user's saves (paginated, filtered)
GET    /saves/:id                # Get single save with extracted data
POST   /saves                    # Create save from share extension
PATCH  /saves/:id                # Update user note, category override
DELETE /saves/:id                # Soft delete save
POST   /saves/batch              # Batch operations (delete, recategorize)
GET    /saves/search?q=pasta     # Full-text search via FTS5
```

**Example: Create Save**

```http
POST /v1/saves
Authorization: Bearer <jwt>
Content-Type: application/json

{
  "url": "https://instagram.com/reel/ABC123/",
  "platform": "instagram",
  "caption_text": "try this recipe 🔥",
  "thumbnail_url": "https://...",
  "user_note": "make for dinner",
  "processing_tier": "tier1_ios"
}

Response 201:
{
  "success": true,
  "data": {
    "id": "save_xyz789",
    "url": "https://instagram.com/reel/ABC123/",
    "category": "recipe",
    "confidence": 0.85,
    "richness_score": 0.65,
    "extracted_data": { "dish_name": "Garlic Pasta", ... },
    "status": "complete",
    "created_at": "2026-02-24T10:30:00Z"
  }
}
```

#### 4.2.3 Deep Extract

```
POST /extract                    # Trigger deep extract (returns job_id)
GET  /extract/:job_id/status     # Poll job status (fallback to SSE)
GET  /extract/:job_id/stream     # SSE stream for progress updates
POST /extract/:job_id/cancel     # Cancel in-progress extraction
POST /extract/batch              # Batch extract (returns job_ids array)
```

**Example: Trigger Extract**

```http
POST /v1/extract
Authorization: Bearer <jwt>
Content-Type: application/json

{
  "save_id": "save_xyz789",
  "device_can_transcribe": true,
  "device_os": "ios",
  "device_tier": "tier1"
}

Response 202:
{
  "success": true,
  "data": {
    "job_id": "extract_job_abc123",
    "save_id": "save_xyz789",
    "status": "pending",
    "estimated_duration_sec": 20,
    "sse_stream_url": "/v1/extract/extract_job_abc123/stream"
  }
}
```

**SSE Stream (see Section 8 for full spec):**

```http
GET /v1/extract/extract_job_abc123/stream
Authorization: Bearer <jwt>
Accept: text/event-stream

event: progress
data: {"stage":"fetching","percent":10}

event: video_ready
data: {"video_url":"https://cdn.../video.mp4","caption":"..."}

event: complete
data: {"category":"recipe","extracted_data":{...}}
```

#### 4.2.4 User & Quota

```
GET  /users/me                   # Profile + quota info
GET  /users/me/quota             # Detailed quota breakdown
PATCH /users/me                  # Update profile (name, preferences)
GET  /users/me/stats             # Usage stats (saves count, extracts used, etc.)
```

**Example: Get Quota**

```http
GET /v1/users/me/quota
Authorization: Bearer <jwt>

Response 200:
{
  "success": true,
  "data": {
    "user_id": "user_123",
    "is_pro": false,
    "month_year": "2026-02",
    "deep_extracts_used": 7,
    "deep_extracts_limit": 10,
    "deep_extracts_remaining": 3,
    "reset_date": "2026-03-01T00:00:00Z",
    "days_until_reset": 5
  }
}
```

#### 4.2.5 Webhooks (Internal)

```
POST /webhooks/revenuecat        # Subscription lifecycle events
POST /webhooks/apns-feedback     # APNS delivery failures (future)
```

### 4.3 Rate Limiting

**Strategy:** Token bucket algorithm per-user, per-endpoint

**Limits:**

| Endpoint | Free User | Paid User | Burst | Window |
|----------|-----------|-----------|-------|--------|
| `POST /saves` | 60/min | 120/min | 20 | 1 min |
| `POST /extract` | 3/min, 10/month | 10/min, unlimited | 5 | 1 min |
| `GET /saves*` | 300/min | 600/min | 50 | 1 min |
| `GET /extract/:id/stream` | 5 concurrent | 10 concurrent | — | — |

**Implementation:** `express-rate-limit` + Redis store

**Response Headers:**

```http
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 42
X-RateLimit-Reset: 1709251260
Retry-After: 45  (if 429)
```

### 4.4 API Versioning Strategy

**Current: v1 (MVP)**

**Future: v2 (breaking changes only)**

**Migration Path:**
- v1 and v2 coexist for 6 months
- Mobile apps declare API version in `Accept` header: `Accept: application/json; version=1`
- Sunset v1 after 95% of active users upgraded to app version supporting v2
- Breaking changes: field renames, response structure changes, removed endpoints

**Non-Breaking Changes (deploy without version bump):**
- Add new optional fields to requests
- Add new fields to responses
- Add new endpoints
- Relax validation rules

---

## 5. Deep Extract Pipeline

### 5.1 Processing Modes

The pipeline has three modes based on device capability:

| Mode | Scraper | Video Download | Transcription | Extraction | Cost |
|------|---------|----------------|---------------|-----------|------|
| **Mode A: Full On-Device** | Server | Client | Client (on-device) | Client (on-device) | $0.002-0.003 |
| **Mode B: Hybrid** | Server | Client | Server (Whisper) | Client (on-device) | $0.005-0.008 |
| **Mode C: Full Server** | Server | Server | Server (Whisper) | Server (LLM) | $0.008-0.012 |

**Device Decision Logic (determined by client, sent in request):**

```typescript
// Client determines mode based on device capability
if (device.hasFoundationModels || device.hasGeminiNano) {
  if (device.hasSpeechAnalyzer || device.hasMLKitSpeech) {
    mode = 'full_on_device';  // Mode A
  } else {
    mode = 'hybrid';  // Mode B
  }
} else {
  mode = 'full_server';  // Mode C
}
```

### 5.2 Step-by-Step Pipeline (Mode A - Target Path)

```
Client Request ──► Server API ──► BullMQ Job Enqueued
                                          │
                                          ▼
                            Worker picks job (priority: paid > free)
                                          │
                                          ▼
                            ┌─────────────────────────────┐
                            │ Step 1: Check Cache         │
                            │ Redis: URL -> extracted_data│
                            │ If HIT: return cached, done │
                            └─────────────────────────────┘
                                          │ MISS
                                          ▼
                            ┌─────────────────────────────┐
                            │ Step 2: Validate URL        │
                            │ - Parse URL                 │
                            │ - Check allowlist           │
                            │ - Resolve to public IP      │
                            │ - Reject private ranges     │
                            └─────────────────────────────┘
                                          │
                                          ▼
                            ┌─────────────────────────────┐
                            │ Step 3: Call Scraper API    │
                            │ Apify (primary)             │
                            │ → ScrapeCreators (fallback) │
                            │ → Bright Data (last resort) │
                            │ Circuit breaker per provider│
                            └─────────────────────────────┘
                                          │
                                          ▼
                            ┌─────────────────────────────┐
                            │ Step 4: Return Video URL    │
                            │ SSE: video_ready event      │
                            │ {video_url, full_caption,   │
                            │  metadata, thumbnail_url}   │
                            └─────────────────────────────┘
                                          │
                                          ▼
                            Client downloads video ────────┐
                            Client extracts audio          │
                            Client transcribes (on-device) │
                            Client OCRs key frames         │
                            Client runs LLM extraction     │
                                          │                │
                                          ▼                │
                            ┌─────────────────────────────┐│
                            │ Step 5: Client POSTs Result ││
                            │ /extract/:job_id/complete   ││
                            │ {extracted_data, transcript}││
                            └─────────────────────────────┘│
                                          │                │
                                          ▼                │
                            ┌─────────────────────────────┐│
                            │ Step 6: Save to Database    ││
                            │ Update saved_content row    ││
                            │ Cache result in Redis       ││
                            └─────────────────────────────┘│
                                          │                │
                                          ▼                │
                            ┌─────────────────────────────┐│
                            │ Step 7: Send SSE Complete   ││
                            │ Increment user quota        ││
                            │ Trigger push notification   ││
                            └─────────────────────────────┘│
                                                           │
                            Client Cleanup ◄───────────────┘
                            Delete temp files
                            Update local DB
```

### 5.3 Mode B & C (Server-Side Transcription/Extraction)

**Mode B: Server transcribes, client extracts**

```
After Step 4 (video_url returned):
  ▼
Server downloads video
Server extracts audio
Server calls Whisper API (Groq or OpenAI)
Server returns transcript via SSE
  ▼
Client runs on-device extraction from transcript
Client POSTs result to /extract/:job_id/complete
```

**Mode C: Fully server-side**

```
After Step 4 (video_url):
  ▼
Server downloads video
Server transcribes (Whisper)
Server OCRs key frames (future, not MVP)
Server runs LLM extraction (Claude Haiku / GPT-4o-mini)
Server POSTs result directly to database
Server sends SSE complete event
```

### 5.4 Extraction Prompt Engineering

**Prompt Structure (sent to on-device LLM or server LLM):**

```typescript
const EXTRACTION_PROMPT_V1 = `
You are a content analysis assistant. Your job is to extract structured information from social media content.

INPUT:
- Caption: {caption}
- Transcript: {transcript}
- On-screen text: {ocr_text}

TASK:
1. Determine the content category from this list:
   recipe, travel, product, tech_gadgets, fitness, beauty, fashion, diy, finance, entertainment, education, parenting, inspiration, general

2. Extract relevant structured fields based on the category. For each field, only include data explicitly present in the input. Do not infer, guess, or hallucinate.

3. Assign a confidence score (0.0-1.0) for the category choice.

OUTPUT FORMAT (JSON only, no other text):
{
  "category": "recipe",
  "confidence": 0.92,
  "title": "15-Minute Garlic Pasta",
  "extracted_data": {
    "recipe": {
      "dish_name": "Garlic Pasta",
      "ingredients": [
        {"item": "spaghetti", "quantity": "200g"},
        {"item": "garlic", "quantity": "4 cloves", "prep": "minced"}
      ],
      "steps": [
        {"text": "Boil pasta until al dente", "time_seconds": null},
        {"text": "Saute garlic in olive oil for 3 minutes", "time_seconds": 180}
      ],
      "prep_time": "15 minutes",
      "servings": 2,
      "cuisine": "Italian",
      "difficulty": "Easy"
    }
  }
}

RULES:
- If a field cannot be extracted, omit it entirely (do not use null or "unknown")
- Preserve exact quantities and measurements from the source
- Extract time durations in consistent units (convert "3 min" to 180 seconds)
- For recipes: prioritize ingredient list accuracy over everything else
- For travel: extract coordinates if location name is specific
- For products: extract numeric price only if explicitly stated
`.trim();
```

**Prompt Versioning:**

- Each prompt gets a version ID: `v1.0`, `v1.1`, `v2.0`
- Stored in database: `extraction_prompts` table
- Worker reads `active_prompt_version` from config at job start
- Each extraction result records `prompt_version` used
- Allows A/B testing and instant rollback (change config, no code deploy)

### 5.5 Quality Assurance Hooks

**Post-Extraction Validation:**

```typescript
function validateExtraction(result: ExtractionResult): ValidationResult {
  const errors: string[] = [];
  
  // Rule 1: Category must be from allowed list
  if (!ALLOWED_CATEGORIES.includes(result.category)) {
    errors.push(`Invalid category: ${result.category}`);
  }
  
  // Rule 2: Confidence must be 0.0-1.0
  if (result.confidence < 0 || result.confidence > 1) {
    errors.push(`Confidence out of range: ${result.confidence}`);
  }
  
  // Rule 3: Recipe must have ingredients or steps
  if (result.category === 'recipe') {
    const recipe = result.extracted_data.recipe;
    if (!recipe?.ingredients?.length && !recipe?.steps?.length) {
      errors.push('Recipe has no ingredients or steps');
    }
  }
  
  // Rule 4: Travel must have location_name
  if (result.category === 'travel') {
    if (!result.extracted_data.travel?.location_name) {
      errors.push('Travel content has no location');
    }
  }
  
  // Rule 5: Check for common hallucination patterns
  const text = JSON.stringify(result.extracted_data);
  if (text.includes('as an AI') || text.includes('I cannot') || text.includes('I apologize')) {
    errors.push('LLM response contamination detected');
  }
  
  return { valid: errors.length === 0, errors };
}
```

If validation fails, mark extraction as `partial_success` and log for manual review.

---

## 6. Scraper Service Layer

### 6.1 Strategy Pattern Implementation

**Interface:**

```typescript
interface ScraperProvider {
  name: string;
  scrape(url: string, platform: Platform): Promise<ScrapeResult>;
  healthCheck(): Promise<boolean>;
  getCostPerResult(): number;
}

interface ScrapeResult {
  success: boolean;
  video_url?: string;
  thumbnail_url?: string;
  full_caption?: string;
  creator_handle?: string;
  hashtags?: string[];
  view_count?: number;
  error?: string;
}
```

**Concrete Implementations:**

```typescript
// Apify (Primary)
class ApifyScraper implements ScraperProvider {
  name = 'apify';
  private apiToken: string;
  
  async scrape(url: string, platform: Platform): Promise<ScrapeResult> {
    const actorId = platform === 'instagram' 
      ? 'apify/instagram-reel-scraper' 
      : 'apify/tiktok-scraper';
    
    const run = await ApifyClient.actor(actorId).call({
      startUrls: [url],
      resultsLimit: 1
    });
    
    // Transform Apify output to ScrapeResult format
    return this.transformResult(run);
  }
  
  getCostPerResult() { return 0.0026; }
}

// ScrapeCreators (Secondary)
class ScrapeCreatorsScraper implements ScraperProvider {
  name = 'scrapecreators';
  
  async scrape(url: string, platform: Platform): Promise<ScrapeResult> {
    const response = await fetch('https://api.scrapecreators.com/v1/scrape', {
      method: 'POST',
      headers: { 'X-API-Key': this.apiKey },
      body: JSON.stringify({ url, platform })
    });
    
    return this.transformResult(await response.json());
  }
  
  getCostPerResult() { return 0.002; }
}

// Bright Data (Tertiary)
class BrightDataScraper implements ScraperProvider {
  name = 'brightdata';
  
  async scrape(url: string): Promise<ScrapeResult> {
    // Bright Data Scraping Browser or Unblocker
    const response = await fetch(`http://brd-customer-${customerId}.zproxy.lum-superproxy.io:22225`, {
      method: 'GET',
      headers: {
        'X-Target-URL': url,
        'Authorization': `Basic ${Buffer.from(`${username}:${password}`).toString('base64')}`
      }
    });
    
    return this.transformResult(await response.text());
  }
  
  getCostPerResult() { return 0.003; }
}
```

### 6.2 Circuit Breaker Implementation

**Circuit States:**

```typescript
enum CircuitState {
  CLOSED = 'closed',    // Healthy, requests flow through
  OPEN = 'open',        // Unhealthy, skip provider
  HALF_OPEN = 'half_open' // Testing, allow one request
}

class CircuitBreaker {
  private state: CircuitState = CircuitState.CLOSED;
  private failureCount = 0;
  private lastFailureTime: Date | null = null;
  private readonly failureThreshold = 3;
  private readonly timeoutMs = 15 * 60 * 1000; // 15 minutes
  private readonly testInterval = 15 * 60 * 1000; // 15 minutes
  
  async execute<T>(fn: () => Promise<T>): Promise<T> {
    if (this.state === CircuitState.OPEN) {
      if (Date.now() - this.lastFailureTime.getTime() > this.testInterval) {
        this.state = CircuitState.HALF_OPEN;
      } else {
        throw new Error('Circuit breaker OPEN');
      }
    }
    
    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }
  
  private onSuccess() {
    if (this.state === CircuitState.HALF_OPEN) {
      this.state = CircuitState.CLOSED;
      this.failureCount = 0;
    }
  }
  
  private onFailure() {
    this.failureCount++;
    this.lastFailureTime = new Date();
    
    if (this.failureCount >= this.failureThreshold) {
      this.state = CircuitState.OPEN;
      logger.error(`Circuit breaker OPEN after ${this.failureCount} failures`);
    }
  }
}
```

**Scraper Manager with Circuit Breakers:**

```typescript
class ScraperManager {
  private providers: ScraperProvider[];
  private breakers: Map<string, CircuitBreaker>;
  
  constructor() {
    this.providers = [
      new ApifyScraper(),
      new ScrapeCreatorsScraper(),
      new BrightDataScraper()
    ];
    
    this.breakers = new Map(
      this.providers.map(p => [p.name, new CircuitBreaker()])
    );
  }
  
  async scrape(url: string, platform: Platform): Promise<ScrapeResult> {
    for (const provider of this.providers) {
      const breaker = this.breakers.get(provider.name);
      
      try {
        const result = await breaker.execute(() => 
          provider.scrape(url, platform)
        );
        
        if (result.success) {
          // Log provider success for cost tracking
          logger.info('Scrape success', { provider: provider.name, cost: provider.getCostPerResult() });
          return result;
        }
      } catch (error) {
        logger.warn('Scrape failed, trying next provider', { 
          provider: provider.name, 
          error: error.message 
        });
        continue; // Try next provider
      }
    }
    
    throw new Error('All scraper providers failed');
  }
}
```

### 6.3 Health Check Endpoint

```typescript
// GET /api/v1/internal/scrapers/health
router.get('/internal/scrapers/health', async (req, res) => {
  const manager = new ScraperManager();
  const health = await Promise.all(
    manager.providers.map(async provider => {
      const breaker = manager.breakers.get(provider.name);
      const healthCheckOk = await provider.healthCheck();
      
      return {
        name: provider.name,
        status: healthCheckOk ? 'healthy' : 'degraded',
        circuit: breaker.state,
        cost_per_result: provider.getCostPerResult(),
        // Fetch from metrics store (future)
        avg_latency_ms: 0,
        success_rate_1h: 0
      };
    })
  );
  
  res.json({ providers: health });
});
```

---

## 7. Job Queue Architecture

### 7.1 BullMQ Configuration

**Queue Definition:**

```typescript
import { Queue, Worker, QueueScheduler } from 'bullmq';
import Redis from 'ioredis';

const redisConnection = new Redis(process.env.REDIS_URL);

// Queue for deep extract jobs
const extractQueue = new Queue('extract', { connection: redisConnection });

// Priority levels (lower number = higher priority)
enum JobPriority {
  HIGH = 1,   // Paid users, auto-extract
  NORMAL = 5  // Free users
}

// Add job to queue
async function enqueueExtractJob(data: ExtractJobData) {
  const priority = data.user_is_pro ? JobPriority.HIGH : JobPriority.NORMAL;
  
  const job = await extractQueue.add('extract', data, {
    priority,
    attempts: 2,  // Max 2 retries
    backoff: { type: 'exponential', delay: 5000 }, // 5s, 15s
    removeOnComplete: 100,  // Keep last 100 completed jobs
    removeOnFail: false     // Keep failed jobs for debugging
  });
  
  return job.id;
}
```

**Worker Implementation:**

```typescript
const extractWorker = new Worker('extract', async (job) => {
  const { save_id, url, platform, device_can_transcribe, user_id } = job.data;
  
  try {
    // Step 1: Check cache
    const cached = await cache.get(`extract:${url}`);
    if (cached) {
      await job.updateProgress({ stage: 'complete', percent: 100 });
      return cached;
    }
    
    // Step 2: Scrape video
    await job.updateProgress({ stage: 'fetching', percent: 10 });
    const scrapeResult = await scraperManager.scrape(url, platform);
    
    // Step 3: If device can transcribe, return video URL and wait for client
    if (device_can_transcribe) {
      await job.updateProgress({ stage: 'video_ready', percent: 50 });
      
      // Send SSE event (see Section 8)
      sseManager.sendEvent(job.id, 'video_ready', scrapeResult);
      
      // Wait for client to POST completion
      return { status: 'awaiting_client', video_url: scrapeResult.video_url };
    }
    
    // Step 4: Server-side transcription (Mode B/C)
    await job.updateProgress({ stage: 'transcribing', percent: 60 });
    const transcript = await transcriptionService.transcribe(scrapeResult.video_url);
    
    // Step 5: Server-side extraction
    await job.updateProgress({ stage: 'extracting', percent: 80 });
    const extracted = await llmService.extract({
      caption: scrapeResult.full_caption,
      transcript,
      ocr_text: '' // Future: server OCR
    });
    
    // Step 6: Save to database
    await db.savedContent.update(save_id, { extracted_data: extracted });
    
    // Step 7: Cache result
    await cache.set(`extract:${url}`, extracted, 30 * 24 * 60 * 60); // 30 days
    
    // Step 8: Increment quota
    await quotaService.incrementUsage(user_id, 'deep_extracts');
    
    await job.updateProgress({ stage: 'complete', percent: 100 });
    return extracted;
    
  } catch (error) {
    logger.error('Extract job failed', { job_id: job.id, error });
    throw error; // Will trigger retry via BullMQ
  }
}, {
  connection: redisConnection,
  concurrency: 3,  // 3 jobs in parallel per worker process
  limiter: {
    max: 100,      // Max 100 jobs processed per minute
    duration: 60000
  }
});
```

### 7.2 Dead Letter Queue (DLQ)

**Failed Job Handling:**

```typescript
extractWorker.on('failed', async (job, error) => {
  if (job.attemptsMade >= job.opts.attempts) {
    // All retries exhausted, move to DLQ
    await dlqQueue.add('failed_extract', {
      original_job_id: job.id,
      save_id: job.data.save_id,
      url: job.data.url,
      error: error.message,
      failed_at: new Date()
    });
    
    logger.error('Job moved to DLQ', { job_id: job.id, error });
  }
});
```

**DLQ Monitoring:**

```typescript
// Cron job: every hour, check DLQ depth
setInterval(async () => {
  const dlqCount = await dlqQueue.count();
  
  if (dlqCount > 50) {
    // Alert: systemic issue
    await alertService.send({
      severity: 'critical',
      message: `DLQ depth: ${dlqCount}. Investigate scraper failures.`
    });
  }
}, 60 * 60 * 1000); // Every hour
```

### 7.3 Job Timeout & Cleanup

**Timeout Enforcement:**

```typescript
const extractWorker = new Worker('extract', handler, {
  ...workerConfig,
  lockDuration: 90000, // 90 seconds max execution time
  lockRenewTime: 30000 // Renew lock every 30s if still running
});
```

If a job doesn't complete within 90 seconds, BullMQ releases the lock and another worker can retry.

**Stale Job Cleanup (Cron):**

```typescript
// Daily cleanup: remove completed jobs older than 7 days
setInterval(async () => {
  await extractQueue.clean(7 * 24 * 60 * 60 * 1000, 'completed');
  await extractQueue.clean(7 * 24 * 60 * 60 * 1000, 'failed');
}, 24 * 60 * 60 * 1000); // Once per day
```

---

## 8. SSE Implementation

### 8.1 Server-Sent Events Specification

**Why SSE over WebSockets:**
- Simpler protocol (HTTP/1.1, no upgrade handshake)
- Auto-reconnect built into browser EventSource API
- Works through most corporate proxies (WebSockets often blocked)
- One-way communication (server → client) matches our use case

**SSE Endpoint:**

```typescript
// GET /api/v1/extract/:job_id/stream
router.get('/extract/:job_id/stream', authenticate, async (req, res) => {
  const { job_id } = req.params;
  const user_id = req.user.id;
  
  // Verify job belongs to user
  const job = await extractQueue.getJob(job_id);
  if (!job || job.data.user_id !== user_id) {
    return res.status(404).json({ error: 'Job not found' });
  }
  
  // Set SSE headers
  res.writeHead(200, {
    'Content-Type': 'text/event-stream',
    'Cache-Control': 'no-cache',
    'Connection': 'keep-alive',
    'X-Accel-Buffering': 'no' // Disable nginx buffering
  });
  
  // Send initial comment (keeps connection alive)
  res.write(': connected\n\n');
  
  // Register this connection in SSE manager
  sseManager.addClient(job_id, res);
  
  // Heartbeat every 30 seconds
  const heartbeat = setInterval(() => {
    res.write(': heartbeat\n\n');
  }, 30000);
  
  // Cleanup on disconnect
  req.on('close', () => {
    clearInterval(heartbeat);
    sseManager.removeClient(job_id, res);
  });
});
```

**SSE Manager (In-Memory Event Distribution):**

```typescript
class SSEManager {
  private clients: Map<string, Set<Response>>;  // job_id -> Set of connected responses
  
  constructor() {
    this.clients = new Map();
  }
  
  addClient(jobId: string, res: Response) {
    if (!this.clients.has(jobId)) {
      this.clients.set(jobId, new Set());
    }
    this.clients.get(jobId).add(res);
  }
  
  removeClient(jobId: string, res: Response) {
    const clients = this.clients.get(jobId);
    if (clients) {
      clients.delete(res);
      if (clients.size === 0) {
        this.clients.delete(jobId);
      }
    }
  }
  
  sendEvent(jobId: string, event: string, data: any) {
    const clients = this.clients.get(jobId);
    if (!clients) return;
    
    const payload = `event: ${event}\ndata: ${JSON.stringify(data)}\n\n`;
    
    for (const res of clients) {
      try {
        res.write(payload);
      } catch (error) {
        // Client disconnected, remove from set
        clients.delete(res);
      }
    }
  }
}

const sseManager = new SSEManager();
```

**Job Progress → SSE Events (in Worker):**

```typescript
// Inside extractWorker handler
await job.updateProgress({ stage: 'fetching', percent: 10 });
sseManager.sendEvent(job.id, 'progress', { stage: 'fetching', percent: 10 });

// When scrape complete
sseManager.sendEvent(job.id, 'video_ready', {
  video_url: scrapeResult.video_url,
  full_caption: scrapeResult.full_caption
});

// When fully complete
sseManager.sendEvent(job.id, 'complete', {
  category: extracted.category,
  extracted_data: extracted.extracted_data
});
```

### 8.2 SSE Client Reconnection

**Last-Event-ID Support:**

```typescript
router.get('/extract/:job_id/stream', authenticate, async (req, res) => {
  const lastEventId = req.headers['last-event-id'];
  
  // If client reconnects, replay missed events from in-memory buffer
  if (lastEventId) {
    const missedEvents = sseManager.getEventsSince(job_id, lastEventId);
    for (const event of missedEvents) {
      res.write(`id: ${event.id}\nevent: ${event.type}\ndata: ${JSON.stringify(event.data)}\n\n`);
    }
  }
  
  // Continue with normal SSE flow...
});
```

**Client Fallback to Polling:**

If SSE connection fails repeatedly (e.g., network drops, corporate proxy blocks), client should fall back to polling `GET /extract/:job_id/status` every 2 seconds.

---

## 9. Database Design

### 9.1 Schema (PostgreSQL via Supabase)

**Users Table:**

```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email TEXT UNIQUE NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  is_pro BOOLEAN DEFAULT FALSE,
  pro_expires_at TIMESTAMPTZ,
  apple_user_id TEXT UNIQUE,  -- Apple Sign-In identifier
  google_user_id TEXT UNIQUE, -- Google Sign-In identifier
  preferences JSONB DEFAULT '{}'::JSONB
);

-- Indexes
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_apple_id ON users(apple_user_id);
CREATE INDEX idx_users_google_id ON users(google_user_id);
```

**Saved Content Table:**

```sql
CREATE TABLE saved_content (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  url TEXT NOT NULL,
  canonical_url TEXT NOT NULL,  -- Normalized URL for deduplication
  platform TEXT NOT NULL CHECK (platform IN ('instagram', 'tiktok', 'youtube', 'twitter', 'pinterest', 'threads', 'other')),
  
  -- Metadata
  caption_text TEXT,
  thumbnail_url TEXT,
  thumbnail_local_path TEXT,
  creator_handle TEXT,
  
  -- Classification
  category TEXT NOT NULL DEFAULT 'general',
  category_confidence FLOAT,
  user_category_override TEXT,
  
  -- Extraction
  extracted_data JSONB DEFAULT '{}'::JSONB,
  richness_score FLOAT,
  processing_tier TEXT,  -- 'tier1_ios', 'tier1_android', 'tier2', 'tier2_5', 'server'
  
  -- User data
  user_note TEXT,
  
  -- Status
  status TEXT DEFAULT 'pending' CHECK (status IN ('pending', 'processing', 'complete', 'failed')),
  
  -- Timestamps
  created_at TIMESTAMPTZ DEFAULT NOW(),
  processed_at TIMESTAMPTZ,
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  
  -- Soft delete
  deleted_at TIMESTAMPTZ
);

-- Indexes
CREATE INDEX idx_saved_content_user_id ON saved_content(user_id) WHERE deleted_at IS NULL;
CREATE INDEX idx_saved_content_canonical_url ON saved_content(canonical_url);
CREATE INDEX idx_saved_content_category ON saved_content(category) WHERE deleted_at IS NULL;
CREATE INDEX idx_saved_content_created_at ON saved_content(created_at DESC) WHERE deleted_at IS NULL;

-- GIN index for JSONB extracted_data (allows querying nested fields)
CREATE INDEX idx_saved_content_extracted_data ON saved_content USING GIN (extracted_data);

-- Full-text search index (FTS5 via pg_trgm extension)
CREATE EXTENSION IF NOT EXISTS pg_trgm;
CREATE INDEX idx_saved_content_caption_trgm ON saved_content USING GIN (caption_text gin_trgm_ops);
```

**Extraction Jobs Table:**

```sql
CREATE TABLE extraction_jobs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  save_id UUID NOT NULL REFERENCES saved_content(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  
  -- Job metadata
  status TEXT DEFAULT 'pending' CHECK (status IN ('pending', 'processing', 'complete', 'failed', 'cancelled')),
  mode TEXT NOT NULL CHECK (mode IN ('full_on_device', 'hybrid', 'full_server')),
  
  -- Scraper details
  scraper_provider TEXT,
  scraper_cost DECIMAL(10, 6),
  
  -- Processing details
  video_url TEXT,
  transcript TEXT,
  transcript_source TEXT,  -- 'on_device', 'server_whisper'
  
  -- LLM extraction
  prompt_version TEXT,
  llm_provider TEXT,  -- 'on_device', 'claude_haiku', 'gpt4o_mini'
  llm_cost DECIMAL(10, 6),
  
  -- Results
  extracted_data JSONB,
  error_message TEXT,
  
  -- Timing
  created_at TIMESTAMPTZ DEFAULT NOW(),
  started_at TIMESTAMPTZ,
  completed_at TIMESTAMPTZ,
  duration_ms INT,
  
  -- Metadata
  cache_hit BOOLEAN DEFAULT FALSE
);

-- Indexes
CREATE INDEX idx_extraction_jobs_save_id ON extraction_jobs(save_id);
CREATE INDEX idx_extraction_jobs_user_id ON extraction_jobs(user_id);
CREATE INDEX idx_extraction_jobs_status ON extraction_jobs(status);
CREATE INDEX idx_extraction_jobs_created_at ON extraction_jobs(created_at DESC);
```

**Quotas Table:**

```sql
CREATE TABLE quotas (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  month_year TEXT NOT NULL,  -- '2026-02'
  
  deep_extracts_used INT DEFAULT 0,
  deep_extracts_limit INT DEFAULT 10,
  
  created_at TIMESTAMPTZ DEFAULT NOW(),
  reset_date TIMESTAMPTZ NOT NULL,
  
  UNIQUE(user_id, month_year)
);

-- Indexes
CREATE INDEX idx_quotas_user_month ON quotas(user_id, month_year);
CREATE INDEX idx_quotas_reset_date ON quotas(reset_date);
```

**Subscriptions Table:**

```sql
CREATE TABLE subscriptions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  
  -- RevenueCat data
  revenuecat_subscriber_id TEXT UNIQUE NOT NULL,
  revenuecat_product_id TEXT NOT NULL,
  revenuecat_entitlement TEXT NOT NULL,  -- 'pro'
  
  -- Status
  status TEXT NOT NULL CHECK (status IN ('active', 'cancelled', 'expired', 'in_trial')),
  
  -- Dates
  purchased_at TIMESTAMPTZ NOT NULL,
  expires_at TIMESTAMPTZ NOT NULL,
  cancelled_at TIMESTAMPTZ,
  
  -- Metadata
  platform TEXT CHECK (platform IN ('ios', 'android')),
  original_transaction_id TEXT,
  
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_subscriptions_user_id ON subscriptions(user_id);
CREATE INDEX idx_subscriptions_revenuecat_id ON subscriptions(revenuecat_subscriber_id);
CREATE INDEX idx_subscriptions_status ON subscriptions(status);
CREATE INDEX idx_subscriptions_expires_at ON subscriptions(expires_at);
```

### 9.2 Row-Level Security (RLS)

**Supabase RLS Policies:**

```sql
-- Users can only read their own data
ALTER TABLE saved_content ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can view own saves"
  ON saved_content FOR SELECT
  USING (auth.uid() = user_id);

CREATE POLICY "Users can insert own saves"
  ON saved_content FOR INSERT
  WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can update own saves"
  ON saved_content FOR UPDATE
  USING (auth.uid() = user_id);

CREATE POLICY "Users can delete own saves"
  ON saved_content FOR DELETE
  USING (auth.uid() = user_id);

-- Similar policies for quotas, extraction_jobs, subscriptions
```

### 9.3 Database Migrations

**Tool:** Supabase CLI + raw SQL migration files

**Migration Workflow:**

```bash
# Create new migration
supabase migration new add_education_category

# Writes: supabase/migrations/20260224103000_add_education_category.sql

# Apply locally
supabase db push

# Apply to production
supabase db push --project-ref <prod-project-id>
```

**Migration Best Practices:**
- Always include rollback SQL in comments
- Test migrations on staging database before production
- Use transactions for multi-statement migrations
- Version control all migrations in `supabase/migrations/`

---

## 10. Caching Strategy

### 10.1 Redis Cache Layers

**Layer 1: URL Deduplication Cache (PRIMARY)**

```typescript
// Cache structure:
// Key: extract:{sha256(canonical_url)}
// Value: JSON string of extracted_data
// TTL: 30 days

async function getCachedExtraction(url: string): Promise<ExtractionResult | null> {
  const key = `extract:${sha256(normalizeUrl(url))}`;
  const cached = await redis.get(key);
  return cached ? JSON.parse(cached) : null;
}

async function cacheExtraction(url: string, result: ExtractionResult) {
  const key = `extract:${sha256(normalizeUrl(url))}`;
  await redis.setex(key, 30 * 24 * 60 * 60, JSON.stringify(result));
}
```

**Why SHA256 hash:** URLs can exceed Redis key length limits (512MB), hashing ensures fixed key size.

**Layer 2: User Quota Cache (FAST ACCESS)**

```typescript
// Cache structure:
// Key: quota:{user_id}:{month_year}
// Value: {used: 7, limit: 10}
// TTL: Until end of month

async function getQuota(userId: string): Promise<Quota> {
  const monthYear = new Date().toISOString().slice(0, 7); // '2026-02'
  const key = `quota:${userId}:${monthYear}`;
  
  let cached = await redis.get(key);
  if (!cached) {
    // Cache miss, fetch from database
    const quota = await db.quotas.findOne({ user_id: userId, month_year: monthYear });
    if (!quota) {
      // Create new quota row
      quota = await db.quotas.create({
        user_id: userId,
        month_year: monthYear,
        deep_extracts_used: 0,
        deep_extracts_limit: 10,
        reset_date: getNextMonthStart()
      });
    }
    
    await redis.setex(key, getSecondsUntilMonthEnd(), JSON.stringify(quota));
    cached = JSON.stringify(quota);
  }
  
  return JSON.parse(cached);
}

async function incrementQuota(userId: string) {
  const monthYear = new Date().toISOString().slice(0, 7);
  const key = `quota:${userId}:${monthYear}`;
  
  // Atomic increment in Redis
  await redis.hincrby(key, 'used', 1);
  
  // Also update database (eventual consistency is fine here)
  await db.quotas.update(
    { user_id: userId, month_year: monthYear },
    { $inc: { deep_extracts_used: 1 } }
  );
}
```

**Layer 3: Rate Limiting State**

```typescript
// Cache structure:
// Key: ratelimit:{user_id}:{endpoint}
// Value: token count
// TTL: 60 seconds

// Implemented via express-rate-limit with Redis store
import rateLimit from 'express-rate-limit';
import RedisStore from 'rate-limit-redis';

const extractRateLimiter = rateLimit({
  store: new RedisStore({ client: redis }),
  windowMs: 60 * 1000,  // 1 minute
  max: (req) => req.user.is_pro ? 10 : 3,  // Dynamic limit
  keyGenerator: (req) => `${req.user.id}:extract`
});

router.post('/extract', extractRateLimiter, async (req, res) => { ... });
```

### 10.2 Cache Invalidation

**When to Invalidate:**

| Event | Cache Key(s) to Invalidate | Reason |
|-------|---------------------------|--------|
| User deletes save | `extract:{url}` | Only if this was the last user with this URL (check count) |
| User upgrades to Pro | `quota:{user_id}:{month}` | Quota limit changed from 10 to unlimited |
| Extraction fails and retries | None | Let cache TTL expire naturally |
| Admin forces re-extraction | `extract:{url}` | Rare, manual operation |

**Cache Warming (Optional):**

Proactively cache extractions for viral content:

```typescript
// Cron job: Identify viral URLs (saved by 10+ users) and pre-cache them
setInterval(async () => {
  const viralUrls = await db.savedContent.aggregate([
    { $group: { _id: '$canonical_url', count: { $sum: 1 } } },
    { $match: { count: { $gte: 10 } } },
    { $sort: { count: -1 } },
    { $limit: 100 }
  ]);
  
  for (const { _id: url } of viralUrls) {
    const cached = await getCachedExtraction(url);
    if (!cached) {
      // Cache miss on viral content — extract and cache proactively
      logger.info('Cache warming viral URL', { url });
      await extractAndCache(url);
    }
  }
}, 6 * 60 * 60 * 1000); // Every 6 hours
```

---

## 11. Authentication & Authorization

### 11.1 Supabase Auth Integration

**Flow: Apple Sign-In (Example)**

```
1. Client calls Sign In with Apple
2. Apple returns identity token
3. Client sends token to Supabase Auth:
   POST https://<project-ref>.supabase.co/auth/v1/token
   { grant_type: 'id_token', provider: 'apple', token: '<apple-token>' }
4. Supabase validates token with Apple, creates user, returns JWT
5. Client stores JWT and uses it for all API calls
```

**Backend JWT Validation:**

```typescript
import { createClient } from '@supabase/supabase-js';

const supabase = createClient(
  process.env.SUPABASE_URL,
  process.env.SUPABASE_ANON_KEY
);

// Middleware: Verify JWT and attach user to request
async function authenticate(req: Request, res: Response, next: NextFunction) {
  const authHeader = req.headers.authorization;
  if (!authHeader?.startsWith('Bearer ')) {
    return res.status(401).json({ error: 'Unauthorized' });
  }
  
  const token = authHeader.substring(7);
  
  const { data: { user }, error } = await supabase.auth.getUser(token);
  if (error || !user) {
    return res.status(401).json({ error: 'Invalid token' });
  }
  
  req.user = user;  // Attach user to request
  next();
}

// Apply to protected routes
router.get('/saves', authenticate, async (req, res) => {
  const saves = await db.savedContent.find({ user_id: req.user.id });
  res.json({ data: saves });
});
```

### 11.2 JWT Refresh Token Flow

**Access Token Expiry:** 1 hour  
**Refresh Token Expiry:** 30 days

**Client-Side Flow:**

```typescript
// On every API call
async function apiCall(url: string, options: RequestInit) {
  let accessToken = localStorage.getItem('access_token');
  
  let response = await fetch(url, {
    ...options,
    headers: {
      ...options.headers,
      'Authorization': `Bearer ${accessToken}`
    }
  });
  
  // If 401, try refreshing token
  if (response.status === 401) {
    const refreshToken = localStorage.getItem('refresh_token');
    const { data, error } = await supabase.auth.refreshSession({ refresh_token: refreshToken });
    
    if (error) {
      // Refresh failed, redirect to login
      window.location.href = '/login';
      return;
    }
    
    // Retry with new token
    accessToken = data.session.access_token;
    localStorage.setItem('access_token', accessToken);
    
    response = await fetch(url, {
      ...options,
      headers: {
        ...options.headers,
        'Authorization': `Bearer ${accessToken}`
      }
    });
  }
  
  return response.json();
}
```

### 11.3 API Key Authentication (Future: 3rd-party integrations)

For webhook endpoints that receive events from external services (RevenueCat), use shared secret validation:

```typescript
// Webhook signature verification
router.post('/webhooks/revenuecat', async (req, res) => {
  const signature = req.headers['x-revenuecat-signature'];
  const payload = req.body;
  
  const expectedSignature = crypto
    .createHmac('sha256', process.env.REVENUECAT_WEBHOOK_SECRET)
    .update(JSON.stringify(payload))
    .digest('hex');
  
  if (signature !== expectedSignature) {
    return res.status(401).json({ error: 'Invalid signature' });
  }
  
  // Process webhook...
});
```

---

## 12. Subscription & Quota Management

### 12.1 RevenueCat Integration

**Webhook Events:**

| Event Type | Trigger | Backend Action |
|------------|---------|----------------|
| `INITIAL_PURCHASE` | User completes first purchase | Create subscription row, set `users.is_pro = true`, reset quota to unlimited |
| `RENEWAL` | Subscription auto-renews | Update `subscriptions.expires_at`, ensure `is_pro = true` |
| `CANCELLATION` | User cancels (still active until expiry) | Set `subscriptions.cancelled_at`, no immediate action (still pro until expires) |
| `EXPIRATION` | Subscription expires without renewal | Set `users.is_pro = false`, reset quota to free tier limit (10/month) |
| `PRODUCT_CHANGE` | User upgrades/downgrades plan | Update `subscriptions.revenuecat_product_id` and quota limits |

**Webhook Handler:**

```typescript
router.post('/webhooks/revenuecat', verifyRevenueCatSignature, async (req, res) => {
  const { event_type, subscriber, product_id, expires_at } = req.body;
  
  const user = await db.users.findOne({ revenuecat_subscriber_id: subscriber.id });
  if (!user) {
    logger.warn('RevenueCat webhook for unknown user', { subscriber_id: subscriber.id });
    return res.status(404).json({ error: 'User not found' });
  }
  
  switch (event_type) {
    case 'INITIAL_PURCHASE':
    case 'RENEWAL':
      await db.users.update(user.id, { is_pro: true, pro_expires_at: new Date(expires_at) });
      await db.subscriptions.upsert({
        user_id: user.id,
        revenuecat_subscriber_id: subscriber.id,
        revenuecat_product_id: product_id,
        status: 'active',
        expires_at: new Date(expires_at),
        platform: subscriber.platform
      });
      break;
      
    case 'EXPIRATION':
      await db.users.update(user.id, { is_pro: false, pro_expires_at: null });
      await db.subscriptions.update({ user_id: user.id }, { status: 'expired' });
      break;
      
    case 'CANCELLATION':
      await db.subscriptions.update({ user_id: user.id }, { 
        status: 'cancelled',
        cancelled_at: new Date()
      });
      // Don't change is_pro yet — user stays pro until expiry
      break;
  }
  
  res.status(200).json({ success: true });
});
```

### 12.2 Quota Enforcement

**Check Quota Before Job:**

```typescript
async function canPerformDeepExtract(userId: string): Promise<QuotaCheckResult> {
  const user = await db.users.findById(userId);
  
  // Pro users have unlimited quota
  if (user.is_pro) {
    return { allowed: true, remaining: Infinity };
  }
  
  const quota = await getQuota(userId);  // From cache (see Section 10)
  
  if (quota.deep_extracts_used >= quota.deep_extracts_limit) {
    return {
      allowed: false,
      reason: 'QUOTA_EXCEEDED',
      reset_date: quota.reset_date
    };
  }
  
  return {
    allowed: true,
    remaining: quota.deep_extracts_limit - quota.deep_extracts_used
  };
}
```

**Increment After Success:**

```typescript
// In extraction worker, after successful completion
if (!user.is_pro) {
  await incrementQuota(user_id);
}
```

### 12.3 Monthly Quota Reset (Cron Job)

**Supabase Edge Function: Reset Quotas**

```typescript
// Deployed as Supabase Edge Function, triggered by cron
// Schedule: 0 0 1 * * (midnight on 1st of every month)

Deno.serve(async (req) => {
  const supabaseClient = createClient(
    Deno.env.get('SUPABASE_URL')!,
    Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')!  // Bypass RLS
  );
  
  const thisMonth = new Date().toISOString().slice(0, 7);
  const nextMonth = new Date(new Date().setMonth(new Date().getMonth() + 1)).toISOString().slice(0, 7);
  
  // Create new quota rows for all free users
  const { data: freeUsers } = await supabaseClient
    .from('users')
    .select('id')
    .eq('is_pro', false);
  
  for (const user of freeUsers) {
    await supabaseClient.from('quotas').insert({
      user_id: user.id,
      month_year: nextMonth,
      deep_extracts_used: 0,
      deep_extracts_limit: 10,
      reset_date: new Date(Date.UTC(new Date().getFullYear(), new Date().getMonth() + 2, 1))
    });
  }
  
  logger.info(`Reset quotas for ${freeUsers.length} users`);
  
  return new Response(JSON.stringify({ success: true }), {
    headers: { 'Content-Type': 'application/json' }
  });
});
```

---

## 13. Infrastructure

### 13.1 Cloud Provider: Railway (Recommended)

**Decision: Railway.app for MVP**

**Rationale:**
- **Simplicity**: Deploy from GitHub with zero config (Railway auto-detects Node.js)
- **Cost**: $5/month starter, $20/month for production (includes DB, Redis, bandwidth)
- **Performance**: Global CDN, auto-scaling, managed SSL
- **DX**: Instant preview deploys, environment variables, logs, metrics in web UI
- **Migration Path**: Railway uses Docker under the hood — easy to migrate to AWS/GCP later

**Alternatives Considered:**
- **AWS (ECS + RDS + ElastiCache)**: Most flexible, but requires DevOps expertise, $50-100/month minimum
- **GCP (Cloud Run + Cloud SQL)**: Good serverless option, but cold starts hurt API latency
- **Heroku**: Excellent DX, but 2-3x more expensive than Railway ($25/dyno + $20/Postgres)

### 13.2 Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                         Client (iOS/Android)                    │
│  Native App ──► KMP Shared Logic ──► Supabase Client SDK       │
└─────────────────────────────────────────────────────────────────┘
                                │
                                │ HTTPS (JWT in header)
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                         Railway (Load Balancer)                 │
└─────────────────────────────────────────────────────────────────┘
                                │
                 ┌──────────────┼──────────────┐
                 ▼              ▼              ▼
        ┌────────────┐  ┌────────────┐  ┌────────────┐
        │ API Server │  │ API Server │  │ API Server │
        │ (Node.js)  │  │ (Node.js)  │  │ (Node.js)  │
        └────────────┘  └────────────┘  └────────────┘
                 │              │              │
                 └──────────────┼──────────────┘
                                │
                 ┌──────────────┴──────────────┐
                 ▼                             ▼
        ┌────────────────┐           ┌────────────────┐
        │ BullMQ Workers │           │ Redis Cluster  │
        │ (Extract Jobs) │◄─────────►│ - Job Queue    │
        │                │           │ - Cache        │
        └────────────────┘           └────────────────┘
                 │                            │
                 │                            │
                 ▼                            │
        ┌────────────────┐                   │
        │ Scraper APIs   │                   │
        │ - Apify        │                   │
        │ - ScrapeCreators│                  │
        │ - Bright Data  │                   │
        └────────────────┘                   │
                 │                            │
                 └──────────────┬─────────────┘
                                │
                                ▼
                 ┌──────────────────────────────┐
                 │ Supabase (Managed Services)  │
                 │ - PostgreSQL (Primary DB)    │
                 │ - Auth (Apple/Google Sign-In)│
                 │ - Storage (Future: thumbs)   │
                 │ - Edge Functions (Webhooks)  │
                 └──────────────────────────────┘
```

### 13.3 Deployment Pipeline

**CI/CD: GitHub Actions + Railway**

```yaml
# .github/workflows/deploy.yml
name: Deploy to Railway

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: 20
      - run: npm ci
      - run: npm test
      - run: npm run lint
  
  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Deploy to Railway
        uses: bervProject/railway-deploy@v1
        with:
          railway_token: ${{ secrets.RAILWAY_TOKEN }}
          service: staqer-api
```

**Environment Variables (Railway Secrets):**

```bash
# Database
SUPABASE_URL=https://<project-ref>.supabase.co
SUPABASE_ANON_KEY=<anon-key>
SUPABASE_SERVICE_ROLE_KEY=<service-role-key>

# Redis
REDIS_URL=redis://default:<password>@<host>:6379

# Scraper APIs
APIFY_API_TOKEN=<token>
SCRAPECREATORS_API_KEY=<key>
BRIGHTDATA_USERNAME=<username>
BRIGHTDATA_PASSWORD=<password>

# LLM APIs
ANTHROPIC_API_KEY=<key>  # Claude Haiku
OPENAI_API_KEY=<key>     # GPT-4o-mini, Whisper

# RevenueCat
REVENUECAT_WEBHOOK_SECRET=<secret>

# Other
NODE_ENV=production
PORT=8080
```

### 13.4 Horizontal Scaling

**When to Scale:**

| Metric | Threshold | Action |
|--------|-----------|--------|
| API response time p95 | > 500ms for 5 minutes | Add 1 API server replica |
| CPU usage | > 80% sustained | Add 1 replica |
| Job queue depth | > 500 pending jobs | Add 1 worker replica |
| Redis memory | > 80% of limit | Upgrade Redis tier or add replicas |

**Railway Auto-Scaling:**

Railway supports horizontal auto-scaling via the web UI:
- Set min/max replicas
- Define scaling trigger (CPU, memory, custom metric via webhook)

For MVP, start with:
- API servers: 1 replica, scale to 3 if needed
- Workers: 1 replica, scale to 2 if job queue depth exceeds 100

---

## 14. Monitoring & Observability

### 14.1 Logging

**Tool: Pino (Fast, structured JSON logging)**

```typescript
import pino from 'pino';

const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  transport: process.env.NODE_ENV === 'development' ? {
    target: 'pino-pretty',
    options: { colorize: true }
  } : undefined
});

// Usage
logger.info({ user_id: 'user_123', save_id: 'save_xyz' }, 'Save created');
logger.error({ error: err.message, stack: err.stack }, 'Extract job failed');
```

**Log Aggregation: Railway built-in logs or Datadog (future)**

Railway provides log streaming in the web UI. For production, ship logs to Datadog or Logtail:

```typescript
import { LogtailTransport } from '@logtail/pino';

if (process.env.LOGTAIL_TOKEN) {
  logger.add(new LogtailTransport(process.env.LOGTAIL_TOKEN));
}
```

### 14.2 Metrics

**Tool: Prometheus + Grafana (via Railway add-on) or Datadog**

**Custom Metrics to Track:**

```typescript
import { Counter, Histogram, Gauge } from 'prom-client';

// Job queue metrics
const jobsEnqueued = new Counter({
  name: 'jobs_enqueued_total',
  help: 'Total extract jobs enqueued',
  labelNames: ['priority', 'mode']
});

const jobDuration = new Histogram({
  name: 'job_duration_seconds',
  help: 'Job processing duration',
  labelNames: ['status', 'mode'],
  buckets: [1, 5, 10, 20, 30, 45, 60]
});

const queueDepth = new Gauge({
  name: 'queue_depth',
  help: 'Current number of pending jobs'
});

// Scraper metrics
const scraperRequests = new Counter({
  name: 'scraper_requests_total',
  help: 'Scraper API requests',
  labelNames: ['provider', 'status']
});

const scraperCost = new Counter({
  name: 'scraper_cost_total',
  help: 'Total scraper cost in USD',
  labelNames: ['provider']
});

// Example usage
jobsEnqueued.inc({ priority: 'high', mode: 'full_on_device' });
jobDuration.observe({ status: 'complete', mode: 'full_on_device' }, 18.5);
scraperCost.inc({ provider: 'apify' }, 0.0026);
```

**Expose Metrics Endpoint:**

```typescript
import { register } from 'prom-client';

router.get('/metrics', async (req, res) => {
  res.set('Content-Type', register.contentType);
  res.send(await register.metrics());
});
```

### 14.3 Alerting

**Critical Alerts (PagerDuty or Opsgenie):**

| Alert | Condition | Severity | Notification Channel |
|-------|-----------|----------|---------------------|
| API Down | 5xx error rate > 10% for 5 min | Critical | PagerDuty + Slack |
| Database Down | Connection errors > 5 in 1 min | Critical | PagerDuty + Slack |
| Job Queue Stalled | No jobs processed in 10 min | High | Slack |
| DLQ Depth High | DLQ > 50 jobs | High | Slack |
| All Scrapers Down | All 3 circuit breakers OPEN | Critical | PagerDuty + Slack |
| Quota Reset Failed | Cron job didn't run on 1st of month | High | Slack |
| Scraper Cost Spike | Daily cost > $50 | Medium | Email + Slack |

**Warning Alerts (Slack only):**

| Alert | Condition | Action |
|-------|-----------|--------|
| High Latency | API p95 latency > 500ms | Investigate, consider scaling |
| Cache Miss Rate Low | Cache hit rate < 15% | Check Redis, investigate URL normalization |
| Low Quota Remaining | >50% of free users exhausted quota by mid-month | Consider UX tweaks to reduce over-extraction |

### 14.4 Dashboards

**Grafana Dashboards (3 key views):**

**1. System Health Dashboard:**
- API request rate (req/min)
- API latency (p50, p95, p99)
- Error rate (5xx, 4xx)
- Database connection pool usage
- Redis memory usage
- Job queue depth (pending, active, completed, failed)

**2. Extract Pipeline Dashboard:**
- Jobs enqueued per hour (by priority, mode)
- Job duration distribution
- Job success rate
- Scraper provider usage (% per provider)
- Scraper cost per day
- Cache hit rate

**3. Business Metrics Dashboard:**
- New users per day
- Active extracts per day (free vs. paid)
- Quota exhaustion rate (% of free users hitting limit)
- Subscription events (new, renewals, cancellations)
- Revenue per day (from RevenueCat webhook events)

---

## 15. Security

### 15.1 SSRF Prevention (URL Validation)

**Implementation:**

```typescript
import { URL } from 'url';
import dns from 'dns/promises';
import ipaddr from 'ipaddr.js';

const ALLOWED_DOMAINS = [
  'instagram.com',
  'www.instagram.com',
  'tiktok.com',
  'www.tiktok.com',
  'vm.tiktok.com',
  'youtube.com',
  'www.youtube.com',
  'youtu.be'
];

async function validateUrl(urlString: string): Promise<void> {
  // Step 1: Parse URL
  let url: URL;
  try {
    url = new URL(urlString);
  } catch {
    throw new Error('Invalid URL format');
  }
  
  // Step 2: Check protocol
  if (!['http:', 'https:'].includes(url.protocol)) {
    throw new Error('Invalid protocol. Only http and https allowed.');
  }
  
  // Step 3: Check port (only 80 and 443)
  const port = url.port || (url.protocol === 'https:' ? '443' : '80');
  if (!['80', '443'].includes(port)) {
    throw new Error('Invalid port. Only 80 and 443 allowed.');
  }
  
  // Step 4: Check domain allowlist
  if (!ALLOWED_DOMAINS.includes(url.hostname)) {
    throw new Error(`Domain ${url.hostname} not allowed`);
  }
  
  // Step 5: Resolve hostname to IP and check for private ranges
  let addresses: string[];
  try {
    addresses = await dns.resolve4(url.hostname);
  } catch {
    throw new Error('Hostname resolution failed');
  }
  
  for (const addr of addresses) {
    const ip = ipaddr.parse(addr);
    if (ip.range() !== 'unicast') {
      throw new Error(`IP ${addr} is not publicly routable (SSRF prevention)`);
    }
  }
}
```

### 15.2 Rate Limiting (API Layer)

Already covered in Section 4.3. Key points:

- Per-user, per-endpoint token bucket
- Dynamic limits (paid users get higher limits)
- Redis-backed to work across multiple server replicas
- 429 response with `Retry-After` header

### 15.3 Input Validation (Request Schema)

**Tool: Zod for runtime type checking**

```typescript
import { z } from 'zod';

const createSaveSchema = z.object({
  url: z.string().url(),
  platform: z.enum(['instagram', 'tiktok', 'youtube', 'twitter', 'pinterest', 'threads', 'other']),
  caption_text: z.string().max(5000).optional(),
  thumbnail_url: z.string().url().optional(),
  user_note: z.string().max(500).optional(),
  processing_tier: z.enum(['tier1_ios', 'tier1_android', 'tier2', 'tier2_5']).optional()
});

router.post('/saves', authenticate, async (req, res) => {
  try {
    const data = createSaveSchema.parse(req.body);
    // Proceed with validated data
  } catch (error) {
    return res.status(400).json({ error: 'Invalid request body', details: error.errors });
  }
});
```

### 15.4 SQL Injection Prevention

**Strategy: Parameterized queries (always)**

PostgreSQL client libraries (pg, Supabase client) automatically parameterize queries. Never use string concatenation.

**Bad:**

```typescript
// DON'T DO THIS
const saves = await db.query(`SELECT * FROM saved_content WHERE user_id = '${userId}'`);
```

**Good:**

```typescript
const saves = await db.query('SELECT * FROM saved_content WHERE user_id = $1', [userId]);

// Or via Supabase client
const { data } = await supabase
  .from('saved_content')
  .select('*')
  .eq('user_id', userId);
```

### 15.5 Secrets Management

**Never commit secrets to git.**

**Local Development:** `.env` file (gitignored)

```bash
# .env (not committed)
SUPABASE_URL=https://...
SUPABASE_ANON_KEY=...
```

**Production:** Railway environment variables (encrypted at rest)

**Rotation Policy:**

| Secret | Rotation Frequency | Process |
|--------|-------------------|---------|
| Database password | Every 90 days | Update Railway secret, restart services |
| API keys (Apify, OpenAI) | On suspected compromise | Regenerate from provider dashboard, update secret |
| JWT signing key | Every 180 days | Supabase handles this automatically |
| RevenueCat webhook secret | On suspected compromise | Regenerate from RevenueCat dashboard |

### 15.6 Temp File Security

Covered in F5 spec, key points:

- Temp files stored in platform temp directory (auto-cleaned by OS)
- Named with UUIDs (no path traversal)
- Deleted within 60 seconds of processing
- Background cron sweeps stale files every 5 minutes

---

## 16. Cost Analysis

### 16.1 Per-Extract Cost Breakdown

| Component | Cost Range | Notes |
|-----------|------------|-------|
| Scraper API (Apify/SC/BD) | $0.002-0.003 | Primary cost driver |
| Whisper Transcription (server) | $0.002-0.003 | Only for Tier 2 devices, or Groq ($0.0001/min, 50x cheaper) |
| LLM Extraction (server) | $0.002-0.003 | Claude Haiku or GPT-4o-mini |
| Server Compute (CPU/GPU) | $0.001-0.002 | Pro-rated from monthly server cost |
| **Total (Mode A)** | **$0.002-0.003** | On-device transcription + extraction |
| **Total (Mode B)** | **$0.005-0.008** | Server transcription, on-device extraction |
| **Total (Mode C)** | **$0.008-0.012** | Fully server-side |

**Optimization: Use Groq for Whisper Instead of OpenAI**

Groq offers Whisper API at $0.0001/minute (vs. OpenAI $0.006/minute) — **60x cheaper**. This drops Mode B cost from $0.005-0.008 to **$0.003-0.004**.

### 16.2 Monthly Cost Projections

**Assumptions:**
- 60% users on Mode A (on-device), 25% Mode B, 15% Mode C
- Average 8 extracts/user/month (free: 5, paid: 15)
- Cache hit rate: 20% at 1K users, 30% at 100K users
- Paid conversion: 5% at 1K users, 8% at 100K users

| Scale | Total Extracts | Billable (post-cache) | Weighted Avg Cost | Monthly Extract Cost | Server Cost (Railway) | Revenue (Subs) | Net Margin |
|-------|----------------|----------------------|-------------------|---------------------|----------------------|----------------|------------|
| **1K users** | 8,000 | 6,400 | $0.004 | $26 | $20 | $250 | **$204 (81%)** |
| **10K users** | 80,000 | 60,000 | $0.004 | $240 | $50 | $4,000 | **$3,710 (93%)** |
| **100K users** | 800,000 | 560,000 | $0.0038 | $2,128 | $200 | $40,000 | **$37,672 (94%)** |

**Key Insight:** Even at 100K users, backend infrastructure costs are ~6% of revenue. The on-device-first strategy keeps margins extremely healthy.

### 16.3 Cost Optimization Levers

| Lever | Impact | Trade-off |
|-------|--------|-----------|
| **Increase on-device adoption** | 60% → 80% would save $400/month at 100K users | Requires newer devices, exclude users on old phones |
| **Switch Whisper to Groq** | Save $150/month at 100K users (Mode B transcription) | None — Groq is faster and cheaper |
| **Increase cache hit rate** | 30% → 40% would save $400/month at 100K users | Requires viral content network effects |
| **Negotiate scraper volume discount** | Apify offers discounts at $500+/month spend | Requires 150K+ extracts/month to hit tier |
| **Self-host Whisper (GPU server)** | Save $1,200/month at 100K users | Adds ops complexity, GPU server cost ($200/month) |

---

## 17. Sprint-by-Sprint Breakdown

### Sprint 0: Foundation (Weeks 1-2)

**Backend:**
- [ ] Set up Railway project, provision Postgres + Redis
- [ ] Initialize Node.js + TypeScript project structure
- [ ] Set up Express.js with basic routes (`/health`, `/auth/me`)
- [ ] Integrate Supabase Auth (Apple/Google Sign-In verification)
- [ ] Create database schema (users, saved_content, quotas, subscriptions)
- [ ] Deploy first version to Railway, test with Postman

**Deliverables:** Backend running on Railway, healthcheck passing, auth working

---

### Sprint 1: Core Save Flow (Weeks 3-4)

**Backend:**
- [ ] Implement `POST /saves` endpoint (create save, store in database)
- [ ] Implement `GET /saves` endpoint (list user's saves, paginated)
- [ ] Implement `GET /saves/:id` endpoint (single save detail)
- [ ] Add URL normalization utility (strip tracking params, resolve short URLs)
- [ ] Add RLS policies for saved_content table
- [ ] Test with mobile clients (iOS + Android)

**Deliverables:** Mobile apps can save URLs via share extension, view saves in library

---

### Sprint 2: Scraper Integration (Weeks 5-6)

**Backend:**
- [ ] Implement ScraperProvider interface + ApifyScraper
- [ ] Implement ScrapeCreatorsScraper + BrightDataScraper (fallbacks)
- [ ] Implement CircuitBreaker pattern
- [ ] Implement ScraperManager with provider chain
- [ ] Add scraper health check endpoint (`/internal/scrapers/health`)
- [ ] Test scraping Instagram, TikTok, YouTube URLs

**Deliverables:** Server can scrape video URLs from all 3 platforms, circuit breaker works

---

### Sprint 3: Job Queue + SSE (Weeks 7-8)

**Backend:**
- [ ] Set up BullMQ + Redis job queue
- [ ] Implement extract job enqueue (`POST /extract`)
- [ ] Implement extraction worker (Mode A: return video URL, wait for client)
- [ ] Implement SSE endpoint (`GET /extract/:job_id/stream`)
- [ ] Implement SSE event distribution (progress, video_ready, complete)
- [ ] Test end-to-end extract flow with mobile client

**Deliverables:** Deep extract triggered from mobile, progress shown via SSE, client downloads video

---

### Sprint 4: Server-Side Transcription + Extraction (Weeks 9-10)

**Backend:**
- [ ] Integrate Groq Whisper API (server transcription for Mode B/C)
- [ ] Integrate Claude Haiku API (server extraction for Mode C)
- [ ] Implement Mode B flow (server transcribes, client extracts)
- [ ] Implement Mode C flow (fully server-side)
- [ ] Add extraction prompt versioning (store in database)
- [ ] Test all 3 modes with different device capabilities

**Deliverables:** Server can handle full extraction for Tier 2 devices

---

### Sprint 5: Quota Management (Weeks 11-12)

**Backend:**
- [ ] Implement quota check before extraction (`canPerformDeepExtract`)
- [ ] Implement quota increment after successful extraction
- [ ] Implement `GET /users/me/quota` endpoint
- [ ] Implement monthly quota reset cron job (Supabase Edge Function)
- [ ] Test quota enforcement (free users hit limit at 10, paid users unlimited)
- [ ] Implement quota exhausted error response with reset date

**Deliverables:** Free users blocked after 10 extracts, quota resets on 1st of month

---

### Sprint 6: Subscription Integration (Weeks 13-14)

**Backend:**
- [ ] Implement RevenueCat webhook endpoint (`/webhooks/revenuecat`)
- [ ] Implement webhook signature verification
- [ ] Handle INITIAL_PURCHASE, RENEWAL, EXPIRATION, CANCELLATION events
- [ ] Update users.is_pro based on subscription status
- [ ] Create subscriptions table and sync from RevenueCat
- [ ] Test with RevenueCat sandbox environment

**Deliverables:** Upgrading to Pro in mobile app grants unlimited extracts

---

### Sprint 7: Caching + Cost Optimization (Weeks 15-16)

**Backend:**
- [ ] Implement URL deduplication cache (Redis)
- [ ] Add cache check before scraping (cache hit = instant return)
- [ ] Implement quota cache (fast quota checks)
- [ ] Add cost tracking metrics (per-extract cost, daily cost)
- [ ] Optimize prompt to reduce LLM token usage
- [ ] Monitor cache hit rate in Grafana

**Deliverables:** Viral content cached, cost per extract measured, cache hit rate >25%

---

### Sprint 8: Monitoring + Observability (Weeks 17-18)

**Backend:**
- [ ] Add Pino structured logging
- [ ] Expose Prometheus metrics endpoint (`/metrics`)
- [ ] Set up Grafana dashboards (System Health, Extract Pipeline, Business Metrics)
- [ ] Configure alerts (API down, job queue stalled, scraper failures)
- [ ] Add error tracking (Sentry or Railway built-in)
- [ ] Load test with k6 (1000 req/min, measure latency)

**Deliverables:** Full observability stack, alerts firing on failures, dashboards live

---

### Sprint 9: Security Hardening (Weeks 19-20)

**Backend:**
- [ ] Implement SSRF prevention (URL validation + DNS resolution check)
- [ ] Add rate limiting to all endpoints (express-rate-limit + Redis)
- [ ] Add input validation with Zod schemas
- [ ] Audit temp file cleanup (ensure all files deleted within 60s)
- [ ] Enable Supabase RLS on all tables
- [ ] Penetration testing (OWASP top 10)

**Deliverables:** Security audit passed, no critical vulnerabilities

---

### Sprint 10: Performance Optimization (Weeks 21-22)

**Backend:**
- [ ] Database query optimization (add missing indexes)
- [ ] Connection pooling tuning (PgBouncer settings)
- [ ] Redis pipelining for batch operations
- [ ] Job queue concurrency tuning (find optimal worker count)
- [ ] CDN setup for thumbnails (Supabase Storage + Cloudflare)
- [ ] Load test with 10K concurrent users, optimize bottlenecks

**Deliverables:** API p95 latency <200ms, job processing <25s avg, no OOM crashes

---

### Sprint 11-12: Polish + Launch Prep (Weeks 23-26)

**Backend:**
- [ ] Add missing endpoints (batch delete, batch recategorize)
- [ ] Implement admin endpoints (internal tools, manual re-extraction)
- [ ] Add A/B testing infrastructure for prompts
- [ ] Write API documentation (OpenAPI/Swagger spec)
- [ ] Final load testing + stress testing
- [ ] Production deployment checklist (backups, failover, runbooks)

**Deliverables:** Backend production-ready, documentation complete, launch green-lit

---

## Appendix A: API Reference (OpenAPI Spec)

```yaml
openapi: 3.0.0
info:
  title: Staqer API
  version: 1.0.0
  description: Backend API for Staqer mobile app

servers:
  - url: https://api.staq.app/v1
    description: Production
  - url: http://localhost:8080/v1
    description: Local Development

components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
  
  schemas:
    SavedContent:
      type: object
      properties:
        id:
          type: string
          format: uuid
        url:
          type: string
          format: uri
        platform:
          type: string
          enum: [instagram, tiktok, youtube, twitter, pinterest, threads, other]
        category:
          type: string
        extracted_data:
          type: object
        richness_score:
          type: number
          minimum: 0
          maximum: 1
        created_at:
          type: string
          format: date-time
    
    Error:
      type: object
      properties:
        code:
          type: string
        message:
          type: string
        details:
          type: object

paths:
  /saves:
    get:
      summary: List user's saves
      security:
        - bearerAuth: []
      parameters:
        - name: page
          in: query
          schema:
            type: integer
            default: 1
        - name: limit
          in: query
          schema:
            type: integer
            default: 50
            maximum: 100
        - name: category
          in: query
          schema:
            type: string
      responses:
        200:
          description: Success
          content:
            application/json:
              schema:
                type: object
                properties:
                  success:
                    type: boolean
                  data:
                    type: array
                    items:
                      $ref: '#/components/schemas/SavedContent'
                  meta:
                    type: object
                    properties:
                      page:
                        type: integer
                      limit:
                        type: integer
                      total:
                        type: integer
    
    post:
      summary: Create a new save
      security:
        - bearerAuth: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required:
                - url
                - platform
              properties:
                url:
                  type: string
                  format: uri
                platform:
                  type: string
                  enum: [instagram, tiktok, youtube]
                caption_text:
                  type: string
                thumbnail_url:
                  type: string
                  format: uri
                user_note:
                  type: string
      responses:
        201:
          description: Save created
          content:
            application/json:
              schema:
                type: object
                properties:
                  success:
                    type: boolean
                  data:
                    $ref: '#/components/schemas/SavedContent'

  /extract:
    post:
      summary: Trigger deep extract
      security:
        - bearerAuth: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required:
                - save_id
                - device_can_transcribe
              properties:
                save_id:
                  type: string
                  format: uuid
                device_can_transcribe:
                  type: boolean
                device_os:
                  type: string
                  enum: [ios, android]
      responses:
        202:
          description: Job enqueued
          content:
            application/json:
              schema:
                type: object
                properties:
                  success:
                    type: boolean
                  data:
                    type: object
                    properties:
                      job_id:
                        type: string
                      save_id:
                        type: string
                      status:
                        type: string
                      sse_stream_url:
                        type: string
```

---

## Appendix B: Database ERD

```
┌─────────────────────────┐
│ users                   │
├─────────────────────────┤
│ id (PK)                 │
│ email (UNIQUE)          │
│ is_pro                  │
│ pro_expires_at          │
│ apple_user_id (UNIQUE)  │
│ google_user_id (UNIQUE) │
│ created_at              │
│ updated_at              │
└─────────────────────────┘
           │
           │ 1:N
           ▼
┌─────────────────────────┐
│ saved_content           │
├─────────────────────────┤
│ id (PK)                 │
│ user_id (FK → users)    │
│ url                     │
│ canonical_url           │
│ platform                │
│ caption_text            │
│ thumbnail_url           │
│ category                │
│ extracted_data (JSONB)  │
│ richness_score          │
│ status                  │
│ created_at              │
│ deleted_at              │
└─────────────────────────┘
           │
           │ 1:N
           ▼
┌─────────────────────────┐
│ extraction_jobs         │
├─────────────────────────┤
│ id (PK)                 │
│ save_id (FK)            │
│ user_id (FK)            │
│ status                  │
│ mode                    │
│ scraper_provider        │
│ video_url               │
│ transcript              │
│ extracted_data (JSONB)  │
│ created_at              │
│ completed_at            │
└─────────────────────────┘

┌─────────────────────────┐
│ quotas                  │
├─────────────────────────┤
│ id (PK)                 │
│ user_id (FK → users)    │
│ month_year              │
│ deep_extracts_used      │
│ deep_extracts_limit     │
│ reset_date              │
│ UNIQUE(user_id, month)  │
└─────────────────────────┘

┌─────────────────────────┐
│ subscriptions           │
├─────────────────────────┤
│ id (PK)                 │
│ user_id (FK → users)    │
│ revenuecat_subscriber_id│
│ revenuecat_product_id   │
│ status                  │
│ expires_at              │
│ cancelled_at            │
│ created_at              │
│ updated_at              │
└─────────────────────────┘
```

---

## Appendix C: Cost Tracking Spreadsheet Template

| Date | Total Extracts | Cache Hits | Billable Extracts | Mode A | Mode B | Mode C | Avg Cost | Scraper Cost | LLM Cost | Server Cost | Total Cost | Revenue | Margin |
|------|---------------|------------|-------------------|--------|--------|--------|----------|--------------|----------|-------------|------------|---------|--------|
| 2026-02-24 | 500 | 100 | 400 | 240 | 100 | 60 | $0.004 | $1.20 | $0.30 | $0.10 | $1.60 | $25 | 94% |

---

**End of Backend Architecture Document**

Total Pages: ~50  
Estimated Implementation Time: 22-26 weeks (5-6 months)  
Team Size: 1 senior backend engineer + 1 mobile engineer per platform