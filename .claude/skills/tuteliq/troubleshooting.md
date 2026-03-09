# Tuteliq Troubleshooting Guide

## Error Code Reference

### Authentication Errors (AUTH_1xxx) — Do NOT retry

| Code | HTTP | Description |
|------|------|-------------|
| `AUTH_1001` | 401 | API key is required |
| `AUTH_1002` | 401 | Invalid API key |
| `AUTH_1003` | 401 | API key has been revoked |
| `AUTH_1004` | 401 | API key is inactive |
| `AUTH_1005` | 401 | API key has expired |
| `AUTH_1006` | 401 | Unauthorized access |
| `AUTH_1007` | 403 | Access forbidden |
| `AUTH_1008` | 503 | Authentication service unavailable (retry OK) |

### Rate Limiting Errors (RATE_2xxx) — Retry after delay

| Code | HTTP | Description |
|------|------|-------------|
| `RATE_2001` | 429 | Rate limit exceeded — check `Retry-After` header |
| `RATE_2002` | 429 | Daily request limit exceeded |
| `RATE_2003` | 429 | Request quota exceeded |

### Validation Errors (VAL_3xxx) — Do NOT retry, fix the request

| Code | HTTP | Description |
|------|------|-------------|
| `VAL_3001` | 400 | Validation failed |
| `VAL_3002` | 400 | Invalid input provided |
| `VAL_3003` | 400 | Missing required field |
| `VAL_3004` | 400 | Invalid format |
| `VAL_3005` | 400 | Batch size exceeds maximum |

### Service Errors (SVC_4xxx) — Safe to retry with backoff

| Code | HTTP | Description |
|------|------|-------------|
| `SVC_4001` | 500 | Unexpected error occurred |
| `SVC_4005` | 500 | AI analysis service error |
| `SVC_4007` | 503 | LLM service temporarily unavailable |

### Not Found Errors (NF_5xxx) — Do NOT retry

| Code | HTTP | Description |
|------|------|-------------|
| `NF_5001` | 404 | Resource not found |
| `NF_5002` | 404 | Endpoint not found |
| `NF_5003` | 404 | User not found |

### Analysis Errors (ANALYSIS_6xxx)

| Code | HTTP | Description | Retry? |
|------|------|-------------|--------|
| `ANALYSIS_6001` | 500 | Analysis failed | Yes |
| `ANALYSIS_6002` | 400 | Unsupported analysis type | No |
| `ANALYSIS_6003` | 400 | File exceeds maximum size | No |
| `ANALYSIS_6004` | 400 | File type not supported | No |
| `ANALYSIS_6005` | 400 | File is required but not provided | No |
| `ANALYSIS_6006` | 500 | Audio transcription failed | Yes |
| `ANALYSIS_6007` | 500 | Image analysis failed | Yes |
| `ANALYSIS_6009` | 400 | Detected language not supported | No |

### Subscription Errors (SUB_7xxx)

| Code | HTTP | Description |
|------|------|-------------|
| `SUB_7001` | 403 | No active subscription |
| `SUB_7002` | 403 | Subscription is inactive |
| `SUB_7003` | 403 | Subscription has expired |
| `SUB_7004` | 429 | Message limit reached |
| `SUB_7005` | 402 | Credits depleted |
| `SUB_7006` | 403 | Endpoint not available on plan |

### Verification Errors (VRF_9xxx)

| Code | HTTP | Description | Retry? |
|------|------|-------------|--------|
| `VRF_9001` | 400 | Document image is required | No |
| `VRF_9002` | 400 | Document image unreadable or too low quality | No — ask user to retake |
| `VRF_9003` | 400 | Unsupported document type | No — use passport, licence, or national ID |
| `VRF_9004` | 400 | Selfie image is required for liveness check | No |
| `VRF_9005` | 400 | Face not detected in selfie | No — ask user to retake |
| `VRF_9006` | 400 | Liveness check failed — possible spoofing detected | No |
| `VRF_9007` | 500 | Verification service error | Yes — retry once |
| `VRF_9008` | 403 | Age verification not available on your plan (requires Pro) | No — upgrade plan |
| `VRF_9009` | 403 | Identity verification not available on your plan (requires Business) | No — upgrade plan |

### WebSocket Errors (WS_10xxx)

| Code | Description |
|------|-------------|
| `WS_10001` | WebSocket authentication failed |
| `WS_10002` | WebSocket connection limit reached |
| `WS_10003` | Audio buffer exceeded maximum size |
| `WS_10004` | Session exceeded maximum duration |
| `WS_10005` | Invalid WebSocket message format |
| `WS_10006` | Real-time transcription failed |

## Retry Strategy

| Error Category | Retry? | Strategy |
|---------------|--------|----------|
| AUTH (except 1008) | No | Fix API key or permissions |
| AUTH_1008 | Yes | Auth service down — backoff |
| RATE | Yes | Respect `Retry-After` header |
| VAL | No | Fix the request payload |
| SVC | Yes | Exponential backoff, max 3 retries |
| ANALYSIS (5xx) | Yes | Backoff, may be transient |
| ANALYSIS (4xx) | No | Fix input (file size, format, language) |
| SUB | No | Check subscription/credits |
| VRF (4xx) | No | Fix input (image quality, document type, selfie) or upgrade plan |
| VRF_9007 | Yes | Verification service down — retry once |

**Recommended backoff:** `min(1000 * 2^attempt, 10000)` ms

## Rate Limit Tiers

| Plan | Requests/min | Max WebSocket Connections |
|------|-------------|--------------------------|
| Starter (Free) | 60 | 1 |
| Indie | 300 | 3 |
| Pro | 1,000 | 10 |
| Business | 3,000 | 50 |
| Enterprise | 10,000 | Unlimited |

**Headers on 429 responses:**
- `Retry-After` — seconds to wait before retrying

**Monitoring:** Use `/v1/usage` to check credit balance and `X-Credits-Remaining` header in every response.

## Common Pitfalls

### 1. Hardcoded API Keys
**Problem:** API key committed to source code or hardcoded in client-side code.
**Fix:** Always use environment variables (`TUTELIQ_API_KEY`). Never expose keys in frontend code — proxy through your backend.

### 2. Missing Age Group
**Problem:** Omitting `age_group` / `ageGroup` from requests, getting default calibration.
**Fix:** Always include age group. Severity scores are calibrated per developmental stage — a missing age group means less accurate risk assessment.

### 3. Wrong Endpoint for Conversations
**Problem:** Using `/v1/safety/unsafe` for multi-message conversations instead of `/v1/safety/grooming`.
**Fix:** Use `/v1/safety/grooming` for conversations — it accepts a `messages` array and detects escalation patterns across messages. `/v1/safety/unsafe` only analyzes single text inputs.

### 4. Not Handling Credits
**Problem:** Running out of credits mid-operation without graceful handling.
**Fix:** Check `X-Credits-Remaining` header in responses. Monitor with `/v1/usage`. Handle `SUB_7005` (credits depleted) gracefully.

### 5. Ignoring Language Detection
**Problem:** Assuming English when users write in other languages.
**Fix:** Tuteliq auto-detects language. Check `language_status` in responses — `"beta"` means detection accuracy may vary. 14 languages supported.

### 6. Oversized Batch Requests
**Problem:** Sending too many items in a single batch, hitting `VAL_3005`.
**Fix:** Check your plan's batch size limits. Split large batches into smaller chunks.

### 7. WebSocket Audio Format Mismatch
**Problem:** Sending audio in wrong format to the streaming endpoint.
**Fix:** Audio must be PCM 16-bit LE, 16 kHz sample rate, mono. Chunk size: 4096–32768 bytes. Video frames must be prefixed with `0x01` byte.
