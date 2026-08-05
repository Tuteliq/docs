# Tuteliq SDK Reference

## Node.js SDK

**Package:** `@tuteliq/sdk`
**Requirements:** Node.js 18+
**Features:** TypeScript definitions included, promise-based async client

```bash
npm install @tuteliq/sdk
```

```typescript
import { Tuteliq } from '@tuteliq/sdk'

const tuteliq = new Tuteliq(process.env.TUTELIQ_API_KEY, {
  timeout: 30_000,   // ms (default: 30000)
  retries: 3,        // automatic retries (default: 3)
  retryDelay: 1000,  // initial retry delay in ms (default: 1000)
})

// Quick check
const result = await tuteliq.detectUnsafe({
  content: "message to analyze",
  context: { ageGroup: '13-15' },
})
console.log(result.unsafe, result.severity, result.categories)
```

## Python SDK

**Package:** `tuteliq`
**Requirements:** Python 3.9+
**Features:** Synchronous and asynchronous clients (async uses httpx)

```bash
pip install tuteliq
```

### Synchronous Client

```python
import os
from tuteliq import Tuteliq

client = Tuteliq(
    api_key=os.environ["TUTELIQ_API_KEY"],
    base_url="https://api.tuteliq.ai",  # default
    timeout=30.0,                         # seconds
    retries=2,                            # automatic retries
)

result = client.detect_unsafe(
    text="message to analyze",
    age_group="13-15",
)
print(result.safe, result.severity, result.categories)
```

### Async Client

```python
import asyncio, os
from tuteliq import AsyncTuteliq

client = AsyncTuteliq(api_key=os.environ["TUTELIQ_API_KEY"])

async def main():
    result = await client.detect_unsafe(
        text="message to analyze",
        age_group="13-15",
    )
    print(result.safe)

asyncio.run(main())
```

## Swift SDK

**Requirements:** iOS 15+ / macOS 12+, Swift 5.7+

```swift
// Package.swift dependency
.package(url: "https://github.com/tuteliq/tuteliq-swift.git", from: "1.0.0")
```

```swift
import Tuteliq

let client = Tuteliq(apiKey: ProcessInfo.processInfo.environment["TUTELIQ_API_KEY"]!)

let result = try await client.detectUnsafe(
    text: "message to analyze",
    ageGroup: "13-15"
)
print(result.safe, result.severity)
```

## Kotlin SDK

**Requirements:** Kotlin 1.8+, Android API 26+ or JVM 11+

```kotlin
// build.gradle.kts
dependencies {
    implementation("ai.tuteliq:tuteliq-kotlin:1.0.0")
}
```

```kotlin
import ai.tuteliq.Tuteliq

val client = Tuteliq(apiKey = System.getenv("TUTELIQ_API_KEY"))

val result = client.detectUnsafe(
    text = "message to analyze",
    ageGroup = "13-15"
)
println("${result.safe} ${result.severity}")
```

## Flutter SDK

**Requirements:** Flutter 3.10+, Dart 3.0+

```yaml
# pubspec.yaml
dependencies:
  tuteliq: ^1.0.0
```

```dart
import 'package:tuteliq/tuteliq.dart';

final client = Tuteliq(apiKey: const String.fromEnvironment('TUTELIQ_API_KEY'));

final result = await client.detectUnsafe(
  text: 'message to analyze',
  ageGroup: '13-15',
);
print('${result.safe} ${result.severity}');
```

## React Native SDK

**Requirements:** React Native 0.72+

```bash
npm install @tuteliq/react-native
```

```typescript
import { Tuteliq } from '@tuteliq/react-native'
import Config from 'react-native-config'

const tuteliq = new Tuteliq(Config.TUTELIQ_API_KEY)

const result = await tuteliq.detectUnsafe({
  content: 'message to analyze',
  context: { ageGroup: '13-15' },
})
console.log(result.unsafe, result.severity)
```

## .NET SDK

**Requirements:** .NET 6.0+

```bash
dotnet add package Tuteliq
```

```csharp
using Tuteliq;

var client = new TuteliqClient(
    apiKey: Environment.GetEnvironmentVariable("TUTELIQ_API_KEY")
);

var result = await client.DetectUnsafeAsync(
    text: "message to analyze",
    ageGroup: "13-15"
);
Console.WriteLine($"{result.Safe} {result.Severity}");
```

## Unity SDK

**Requirements:** Unity 2021.3+, .NET Standard 2.1

```
// Install via Unity Package Manager
// Add git URL: https://github.com/tuteliq/tuteliq-unity.git
```

```csharp
using Tuteliq;

var client = new TuteliqClient(apiKey: Environment.GetEnvironmentVariable("TUTELIQ_API_KEY"));

// Use coroutine or async
var result = await client.DetectUnsafeAsync(
    text: "message to analyze",
    ageGroup: "13-15"
);
Debug.Log($"Safe: {result.Safe}, Severity: {result.Severity}");
```

## CLI

```bash
npm install -g @tuteliq/cli
```

```bash
# Set API key
export TUTELIQ_API_KEY=tq_your_key_here

# Detect unsafe content
tuteliq detect unsafe --text "message to analyze" --age-group 13-15

# Detect grooming in a conversation file
tuteliq detect grooming --file conversation.json --age-group 10-12

# Check usage
tuteliq usage
```

## MCP Server

**Package:** `@tuteliq/mcp`
**Requirements:** Claude Code, Cursor, or any MCP-compatible client

```bash
npm install -g @tuteliq/mcp
```

The MCP server exposes all Tuteliq API endpoints as tools, enabling AI assistants to call the API directly during development sessions. Configure it in your MCP client settings with your API key.

## Authentication Methods

All SDKs support two authentication methods:

1. **Bearer Token:** `Authorization: Bearer tq_your_key`
2. **API Key Header:** `x-api-key: tq_your_key`

Key format: starts with `tq_` (production). Keys are hashed with SHA-256 and shown only at creation time.

**Environments:**
- `production` — Premium tier, 1,000 req/min
- `staging` — Basic tier, 300 req/min
- `development` — Free tier, 60 req/min
