# Tuteliq API Endpoints Reference

**Base URL:** `https://api.tuteliq.ai`

## Core Safety Endpoints

| Endpoint | Method | Purpose | Credits |
|----------|--------|---------|---------|
| `/v1/safety/unsafe` | POST | Scan text for harmful content across KOSA categories | 1 |
| `/v1/safety/bullying` | POST | Detect bullying and harassment patterns | 1 |
| `/v1/safety/grooming` | POST | Analyze conversation for grooming indicators | 1 per 10 msgs |
| `/v1/safety/voice` | POST | Transcribe and analyze audio files | 5 |
| `/v1/safety/image` | POST | Analyze images for unsafe visual content | 3 |
| `/v1/safety/video` | POST | Frame-by-frame video analysis | 10 |

## Fraud Detection Endpoints

| Endpoint | Purpose | Credits |
|----------|---------|---------|
| `/v1/fraud/social-engineering` | Detect manipulation/impersonation tactics | 1 |
| `/v1/fraud/app-detection` | Fake apps and malicious links | 1 |
| `/v1/fraud/romance-scam` | Romance scam patterns | 1 |
| `/v1/fraud/mule-recruitment` | Money mule recruitment targeting minors | 1 |

## Safety Extended Endpoints

| Endpoint | Purpose | Credits |
|----------|---------|---------|
| `/v1/safety/gambling-harm` | Underage gambling and addiction patterns | 1 |
| `/v1/safety/coercive-control` | Isolation, financial control, surveillance | 1 |
| `/v1/safety/vulnerability-exploitation` | Cross-endpoint vulnerability profiling | 1 |
| `/v1/safety/radicalisation` | Extremist rhetoric and recruitment | 1 |

## Analysis & Guidance Endpoints

| Endpoint | Purpose | Credits |
|----------|---------|---------|
| `/v1/analysis/emotions` | Emotional well-being analysis | 1 per 10 msgs |
| `/v1/analysis/multi` | Fan-out to up to 10 endpoints in one call | Sum of endpoints |
| `/v1/guidance/action-plan` | Age-appropriate guidance generation | 2 |
| `/v1/reports/generate` | Structured safety reports | 3 |

## Verification Endpoints (Beta)

| Endpoint | Method | Purpose | Credits | Min Tier |
|----------|--------|---------|---------|----------|
| `/v1/verification/age` | POST | Verify user age via document analysis, biometric estimation, or combined | 5 | Pro ($99/mo) |
| `/v1/verification/identity` | POST | Document authentication + face matching + liveness detection | 10 | Business ($349/mo) |

**Age Verification methods:** `document` (government ID), `biometric` (selfie), `combined` (both)
**Identity checks:** MRZ validation, hologram detection, tamper analysis, face matching, liveness detection (photo-of-photo, screen replay, printed mask, deepfake)

## Batch & Streaming

| Endpoint | Purpose | Credits |
|----------|---------|---------|
| `/v1/batch/process` | Bulk processing for high-volume operations | Per-endpoint x count |
| `wss://api.tuteliq.ai/api/v1/safety/voice/stream` | Real-time audio/video streaming | 1 per audio flush, 3 per video frame |

## Utility Endpoints

| Endpoint | Purpose |
|----------|---------|
| `/v1/usage` | Check credit balance and usage metrics |
| `/v1/compliance/gdpr` | GDPR data subject rights |
| `/v1/health` | Service health checks |
| `/v1/webhooks/config` | Configure webhook endpoints |

## Request Patterns

### Text Analysis (single message)

```json
{
  "text": "content to analyze",
  "context": {
    "age_group": "13-15",
    "language": "en",
    "country": "GB"
  }
}
```

### Conversation Analysis (grooming, emotions)

```json
{
  "messages": [
    { "role": "stranger", "text": "Hey, how old are you?" },
    { "role": "child", "text": "I'm 11" },
    { "role": "stranger", "text": "Let's talk on a different app, just us" }
  ],
  "context": {
    "age_group": "10-12"
  }
}
```

### Multi-Endpoint Analysis

```json
{
  "inputs": [
    { "text": "message to analyze", "ageGroup": "13-15" }
  ],
  "detections": ["social-engineering", "romance-scam", "grooming"]
}
```

## Response Structure

### Success Response

```json
{
  "detected": true,
  "severity": 0.85,
  "confidence": 0.91,
  "risk_score": 0.87,
  "categories": [
    { "tag": "GROOMING", "label": "Grooming", "confidence": 0.92 }
  ],
  "credits_used": 1,
  "language": "en",
  "language_status": "stable",
  "detected_language": "en"
}
```

### Grooming Response (additional fields)

```json
{
  "groomingDetected": true,
  "riskScore": 0.92,
  "stage": "isolation",
  "messageAnalysis": [
    { "messageIndex": 0, "riskScore": 0.3 },
    { "messageIndex": 2, "riskScore": 0.95 }
  ]
}
```

### Multi-Endpoint Response

```json
{
  "results": [
    {
      "detections": {
        "social_engineering": { "detected": true, "severity": 0.7 },
        "romance_scam": { "detected": false, "severity": 0.1 }
      }
    }
  ]
}
```

### Voice Analysis Response

```json
{
  "transcript": "full transcription",
  "safe": true,
  "severity": "low",
  "emotions": [{ "label": "sadness", "score": 0.87 }],
  "credits_used": 5
}
```

## Detection Categories

### KOSA Core (9 categories)
1. **Eating Disorders** — Pro-anorexia, pro-bulimia, dangerous diet challenges
2. **Substance Use** — Drug/alcohol/vaping promotion
3. **Suicidal Behaviors** — Self-harm, suicidal ideation (always elevated to CRITICAL)
4. **Depression & Anxiety** — Emotional distress, hopelessness
5. **Compulsive Usage** — Dark patterns, engagement manipulation
6. **Harassment & Bullying** — Insults, exclusion, cyberbullying
7. **Sexual Exploitation** — Grooming, explicit content, sextortion
8. **Voice & Audio Threats** — Verbal harassment, threats in audio
9. **Visual Content Risks** — Inappropriate images, violent/explicit content

### Extended Categories
- Social Engineering, App Fraud, Romance Scams, Money Mule Recruitment
- Gambling Harm, Coercive Control, Vulnerability Exploitation, Radicalisation

## Age Group Calibration

| Age Group | Effect |
|-----------|--------|
| `under-10` / `5-9` | Most protective — lower thresholds, higher severity scores |
| `10-12` | High protection — content rated medium for teens may be elevated |
| `13-15` | Moderate calibration — standard teen-appropriate thresholds |
| `14-17` | Least adjustment — closer to default thresholds |

Severity scores are adjusted based on developmental stage. Content flagged as `medium` for a 16-year-old may be elevated to `high` or `critical` for a younger user.

## Supported Languages (27)

| Code | Language | Status |
|------|----------|--------|
| `en` | English | Stable |
| `es` | Spanish | Beta |
| `pt` | Portuguese | Beta |
| `uk` | Ukrainian | Beta |
| `sv` | Swedish | Beta |
| `no` | Norwegian | Beta |
| `da` | Danish | Beta |
| `fi` | Finnish | Beta |
| `de` | German | Beta |
| `fr` | French | Beta |
| `nl` | Dutch | Beta |
| `pl` | Polish | Beta |
| `it` | Italian | Beta |
| `tr` | Turkish | Beta |
| `ro` | Romanian | Beta |
| `el` | Greek | Beta |
| `cs` | Czech | Beta |
| `hu` | Hungarian | Beta |
| `bg` | Bulgarian | Beta |
| `hr` | Croatian | Beta |
| `sk` | Slovak | Beta |
| `lt` | Lithuanian | Beta |
| `lv` | Latvian | Beta |
| `et` | Estonian | Beta |
| `sl` | Slovenian | Beta |
| `mt` | Maltese | Beta |
| `ga` | Irish | Beta |

Language is auto-detected via trigram + LLM confirmation. Response includes `language`, `language_status`, and `detected_language` fields.

### Age Verification Response

```json
{
  "verified": true,
  "estimated_age": 15,
  "age_range": "13-15",
  "is_minor": true,
  "confidence": 0.97,
  "method": "combined",
  "document_type": "passport",
  "document_country": "GB",
  "biometric_age": 15,
  "document_age": 15,
  "credits_used": 5
}
```

### Identity Verification Response

```json
{
  "verified": true,
  "match_score": 0.98,
  "liveness_passed": true,
  "document_authenticated": true,
  "estimated_age": 34,
  "age_range": "adult",
  "is_minor": false,
  "confidence": 0.99,
  "document_type": "driving_licence",
  "document_country": "US",
  "flags": [],
  "checks": {
    "mrz_valid": true,
    "tamper_detected": false,
    "face_match": true,
    "liveness": true,
    "document_expired": false
  },
  "credits_used": 10
}
```

## Response Headers

- `X-Credits-Remaining` — Current credit balance after the request
- `Retry-After` — Seconds to wait before retrying (on 429 responses)
