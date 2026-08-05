# Tuteliq Integration Examples

## Single Message Check

### Node.js

```typescript
import { Tuteliq } from '@tuteliq/sdk'

const tuteliq = new Tuteliq(process.env.TUTELIQ_API_KEY)

const result = await tuteliq.detectUnsafe({
  content: "Let's meet at the park after school, don't tell your parents",
  context: { ageGroup: '10-12' },
})

if (result.unsafe) {
  console.log(`Detected: severity=${result.severity}`)
  console.log(`Categories: ${result.categories.join(', ')}`)
}
```

### Python

```python
import os
from tuteliq import Tuteliq

client = Tuteliq(api_key=os.environ["TUTELIQ_API_KEY"])

result = client.detect_unsafe(
    text="Let's meet at the park after school, don't tell your parents",
    age_group="10-12",
)

if not result.safe:
    print(f"Detected: severity={result.severity}")
    print(f"Categories: {[c.tag for c in result.categories]}")
```

## Conversation Analysis (Grooming Detection)

### Node.js

```typescript
const result = await tuteliq.detectGrooming({
  messages: [
    { role: 'stranger', content: 'Hey, how old are you?' },
    { role: 'child', content: "I'm 11" },
    { role: 'stranger', content: "You seem really mature for your age" },
    { role: 'stranger', content: "Let's talk on a different app, just us" },
  ],
  childAge: 11,
})

console.log(result.grooming_risk)  // "high"
console.log(result.risk_score)     // 0.92
console.log(result.flags)          // ["isolation", "secrecy"]

// Per-message risk breakdown
if (result.message_analysis) {
  for (const msg of result.message_analysis) {
    console.log(`Message ${msg.message_index}: risk=${msg.risk_score}`)
  }
}
```

### Python

```python
result = client.detect_grooming(
    messages=[
        {"role": "stranger", "text": "Hey, how old are you?"},
        {"role": "child", "text": "I'm 11"},
        {"role": "stranger", "text": "You seem really mature for your age"},
        {"role": "stranger", "text": "Let's talk on a different app, just us"},
    ],
    age_group="10-12",
)

print(result.grooming_detected)  # True
print(result.risk_score)          # 0.92
print(result.stage)               # "isolation"
```

## Emotion Analysis

### Node.js

```typescript
const result = await tuteliq.analyzeEmotions({
  content: "Nobody at school talks to me anymore. I just sit alone every day.",
  context: { ageGroup: '13-15' },
})

console.log(result.dominant_emotions)    // ["sadness", "loneliness"]
console.log(result.emotion_scores)       // { sadness: 0.87, loneliness: 0.75, ... }
console.log(result.trend)               // "worsening"
console.log(result.recommended_followup) // "Check in about school relationships..."
```

### Python

```python
result = client.analyze_emotions(
    text="Nobody at school talks to me anymore. I just sit alone every day.",
    age_group="13-15",
)

print(result.emotions)    # [{"label": "sadness", "score": 0.87}, ...]
print(result.distress)    # True
print(result.risk_level)  # "elevated"
```

## Multi-Endpoint Analysis

Run multiple detection endpoints on a single piece of content in one API call.

### Node.js

```typescript
const result = await tuteliq.analyseMulti({
  content: "You're so special. Nobody else understands you like I do. Keep this a secret.",
  detections: ['social-engineering', 'romance-scam', 'grooming'],
  context: { ageGroup: '13-15' },
})

console.log(result.summary.highest_risk)       // "critical"
console.log(result.summary.total_credits_used)  // 3
console.log(result.results.length)              // 3
```

### Python

```python
result = client.analyse_multi(
    inputs=[
        {"text": "You're so special. Nobody else understands you like I do.", "age_group": "13-15"},
        {"text": "Can you keep a secret from your mum?", "age_group": "10-12"},
    ],
    detections=["social-engineering", "romance-scam", "grooming"],
)

for item in result.results:
    for detection_type, detection in item.detections.items():
        if detection.detected:
            print(f"{detection_type}: severity={detection.severity}")
```

## Batch Processing

For high-volume operations. Credits = per-endpoint cost x item count.

### Node.js

```typescript
const result = await tuteliq.batch({
  items: messages.map(msg => ({
    type: 'unsafe',
    content: msg.content,
    context: { ageGroup: '13-15' },
  })),
})

for (const item of result.results) {
  if (item.success) {
    console.log(`Item ${item.index}: processed`)
  }
}
```

## Webhook Setup

```typescript
import { WebhookEventType } from '@tuteliq/sdk'

const result = await tuteliq.createWebhook({
  name: 'Safety Alerts',
  url: 'https://your-app.com/api/tuteliq-webhook',
  events: [WebhookEventType.INCIDENT_CRITICAL, WebhookEventType.GROOMING_DETECTED],
})

console.log('Secret:', result.secret) // Store this securely!
```

### Webhook Handler (Express)

```typescript
import crypto from 'node:crypto'

app.post('/api/tuteliq-webhook', (req, res) => {
  const signature = req.headers['x-tuteliq-signature']
  const expected = crypto
    .createHmac('sha256', process.env.WEBHOOK_SECRET)
    .update(JSON.stringify(req.body))
    .digest('hex')

  if (signature !== expected) {
    return res.status(401).send('Invalid signature')
  }

  const { event, data } = req.body
  console.log(`Webhook: ${event}, severity=${data.severity}`)
  res.sendStatus(200)
})
```

## Voice Analysis

### Node.js

```typescript
import { readFileSync } from 'fs'

const result = await tuteliq.analyzeVoice({
  file: readFileSync('recording.wav'),
  filename: 'recording.wav',
  analysisType: 'all',
  ageGroup: '13-15',
})

console.log(result.transcription.text)   // Full transcript
console.log(result.overall_severity)     // "low" | "medium" | "high" | "critical"
console.log(result.overall_risk_score)   // 0.0 - 1.0
```

## Real-Time Voice Streaming (WebSocket)

```typescript
const ws = new WebSocket(
  `wss://api.tuteliq.ai/api/v1/safety/voice/stream?api_key=${process.env.TUTELIQ_API_KEY}`
)

ws.on('message', (data) => {
  const msg = JSON.parse(data.toString())

  switch (msg.type) {
    case 'ready':
      console.log('Connected, sending audio...')
      break
    case 'transcription':
      console.log(`Transcript: ${msg.text}`)
      break
    case 'safety_alert':
      console.log(`ALERT: ${msg.category} severity=${msg.severity}`)
      break
    case 'session_summary':
      console.log(`Session ended: ${msg.credits_used} credits used`)
      break
  }
})

// Send audio chunks (PCM 16-bit LE, 16kHz mono, 4096-32768 bytes)
ws.send(audioBuffer)

// Send video frame (prefix with 0x01)
const frameBuffer = Buffer.concat([Buffer.from([0x01]), jpegBuffer])
ws.send(frameBuffer)
```

## Age Verification

### Node.js

```typescript
import { Tuteliq, VerificationMode } from '@tuteliq/sdk'

const session = await tuteliq.createVerificationSession({
  mode: VerificationMode.AGE,
})

console.log(session.url)         // Open in browser or webview
console.log(session.expires_at)  // ISO timestamp

// Poll for result
const result = await tuteliq.getVerificationSession(session.session_id)

if (result.status === 'completed' && result.result) {
  console.log(result.result.is_minor)     // true
  console.log(result.result.age_bracket)  // "13-15"
  console.log(result.result.credits_used) // 10
}
```

## Identity Verification

### Node.js

```typescript
import { Tuteliq, VerificationMode, DocumentType } from '@tuteliq/sdk'

const session = await tuteliq.createVerificationSession({
  mode: VerificationMode.IDENTITY,
  document_type: DocumentType.PASSPORT,
  redirect_url: 'https://example.com/done',
})

// Poll for result
const result = await tuteliq.getVerificationSession(session.session_id)

if (result.status === 'completed' && result.result) {
  console.log(result.result.full_name)      // "John Doe"
  console.log(result.result.date_of_birth)  // "1990-01-15"
  console.log(result.result.face_match)     // { matched: true, distance: 0.2, confidence: 0.98 }
  console.log(result.result.liveness)       // { valid: true }
  console.log(result.result.credits_used)   // 15
}
```

## Age Gate + COPPA Parental Consent Pattern

### Node.js

```typescript
import { VerificationMode } from '@tuteliq/sdk'

// 1. Create age verification session for child
const childSession = await tuteliq.createVerificationSession({
  mode: VerificationMode.AGE,
})

// Open childSession.url in browser, poll for result...
const childResult = await tuteliq.getVerificationSession(childSession.session_id)

if (childResult.status === 'completed' && childResult.result?.is_minor) {
  // 2. Under-13: verify parent's identity (COPPA)
  const parentSession = await tuteliq.createVerificationSession({
    mode: VerificationMode.IDENTITY,
  })

  // Open parentSession.url, poll for result...
  const parentResult = await tuteliq.getVerificationSession(parentSession.session_id)

  if (parentResult.status === 'completed' && parentResult.result) {
    await grantParentalConsent(childUserId, parentResult.result)
  }
}

// 3. Use verified age group in all subsequent safety calls
const safety = await tuteliq.detectUnsafe({
  content: messageContent,
  context: { ageGroup: childResult.result?.age_bracket },
})
```

## Error Handling Pattern

### Node.js

```typescript
import { Tuteliq, TuteliqError } from '@tuteliq/sdk'

async function analyzeWithRetry<T>(fn: () => Promise<T>, maxRetries = 3): Promise<T> {
  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    try {
      return await fn()
    } catch (error) {
      if (error instanceof TuteliqError) {
        // Don't retry client errors (except rate limits)
        if (error.status < 500 && error.code !== 'RATE_2001') throw error

        // Log the error
        console.error(`Attempt ${attempt + 1} failed: ${error.code} - ${error.message}`)
      }
      if (attempt === maxRetries) throw error

      // Exponential backoff
      const delay = Math.min(1000 * 2 ** attempt, 10_000)
      await new Promise(r => setTimeout(r, delay))
    }
  }
  throw new Error('Unreachable')
}

// Usage
const result = await analyzeWithRetry(() =>
  tuteliq.detectUnsafe({ content: 'message', context: { ageGroup: '13-15' } })
)
```

### Python

```python
from tuteliq import Tuteliq, TuteliqError

try:
    result = client.detect_unsafe(
        text="some content",
        age_group="10-12",
    )
except TuteliqError as e:
    print(f"Error {e.code}: {e.message} (HTTP {e.status})")

    if e.code == "RATE_2001":
        # Rate limited — retry after delay
        pass
    elif e.code.startswith("AUTH_"):
        # Authentication issue — check API key
        pass
    elif e.code.startswith("SVC_"):
        # Server error — safe to retry
        pass
```

## Real-Time Monitoring Pattern

Sliding window for real-time chat + batch queue for periodic analysis.

### Node.js

```typescript
class SafetyMonitor {
  private window: { text: string; timestamp: number }[] = []
  private batchQueue: string[] = []
  private readonly WINDOW_SIZE = 20
  private readonly BATCH_THRESHOLD = 100

  async onMessage(text: string, ageGroup: string) {
    // Real-time check for every message
    const result = await tuteliq.detectUnsafe({
      content: text,
      context: { ageGroup },
    })

    if (result.unsafe && result.risk_score >= 0.8) {
      this.handleCriticalAlert(result)
      return
    }

    // Maintain sliding window for conversation context
    this.window.push({ text, timestamp: Date.now() })
    if (this.window.length > this.WINDOW_SIZE) this.window.shift()

    // Queue for batch analysis
    this.batchQueue.push(text)
    if (this.batchQueue.length >= this.BATCH_THRESHOLD) {
      await this.flushBatch(ageGroup)
    }
  }

  private async flushBatch(ageGroup: string) {
    const items = this.batchQueue.splice(0)
    await tuteliq.batch({
      items: items.map(content => ({
        type: 'unsafe' as const,
        content,
        context: { ageGroup },
      })),
    })
  }

  private handleCriticalAlert(result: any) {
    console.error(`CRITICAL: ${result.categories.join(', ')}`)
  }
}
```
