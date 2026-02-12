---
description: "Integrate Schematic feature flags, entitlements, usage tracking, identify events, or webhook verification in backend code (Node/TypeScript, Python, Go, Java, C#)"
---

# Schematic Backend SDK Integration

## What is Schematic?

Schematic is a feature and entitlement management platform. Key concepts:

- **Features**: Product capabilities gated by flags (boolean on/off)
- **Entitlements**: What a company/user has access to, based on their plan
- **Plans**: Bundles of feature entitlements and usage limits
- **Flags**: Runtime checks that evaluate entitlement rules
- **Identify events**: Upsert user/company context for targeting
- **Track events**: Record usage for metered entitlements (e.g., API calls, seats)

## Language Detection

When the user is working in a specific language, show examples only for that language. If unclear, ask which language they're using.

## Installation

**Node/TypeScript:**
```bash
npm install @schematichq/schematic-typescript-node
```

**Python:**
```bash
pip install schematichq
```

**Go:**
```bash
go get github.com/schematichq/schematic-go
```

**Java (Gradle):**
```gradle
implementation 'com.schematichq:schematic-java:+'
```

**C# (.NET):**
```bash
dotnet add package SchematicHQ.Client
```

## Client Initialization

Always use environment variables for API keys. Never hardcode them.

**Node/TypeScript:**
```typescript
import { SchematicClient } from "@schematichq/schematic-typescript-node";

const schematic = new SchematicClient({
  apiKey: process.env.SCHEMATIC_API_KEY,
});

// Always close when done
process.on("SIGTERM", async () => {
  await schematic.close();
});
```

**Python (sync):**
```python
import os
from schematic.client import Schematic

client = Schematic(os.environ["SCHEMATIC_API_KEY"])

# Always shut down when done
client.shutdown()
```

**Python (async):**
```python
from schematic.client import AsyncSchematic

async with AsyncSchematic(os.environ["SCHEMATIC_API_KEY"]) as client:
    # client is automatically initialized and cleaned up
    pass
```

**Go:**
```go
import (
    schematicgo "github.com/schematichq/schematic-go"
    "github.com/schematichq/schematic-go/client"
    "github.com/schematichq/schematic-go/option"
)

schematicClient := client.NewSchematicClient(
    option.WithAPIKey(os.Getenv("SCHEMATIC_API_KEY")),
)
defer schematicClient.Close()
```

**Java:**
```java
import com.schematic.api.Schematic;

try (Schematic schematic = Schematic.builder()
        .apiKey(System.getenv("SCHEMATIC_API_KEY"))
        .build()) {
    // Use schematic — auto-closed at end of try block
}
```

**C#:**
```csharp
using SchematicHQ;

var schematic = new Schematic(Environment.GetEnvironmentVariable("SCHEMATIC_API_KEY"));

// Always shut down when done
await schematic.Shutdown();
```

## Check Feature Flags

**Node/TypeScript:**
```typescript
const isEnabled = await schematic.checkFlag({
  company: { id: "company-id" },
  user: { id: "user-id" },
}, "feature-flag-key");
```

**Python:**
```python
is_enabled = client.check_flag(
    flag_key="feature-flag-key",
    company={"id": "company-id"},
    user={"id": "user-id"},
)
```

**Go:**
```go
isEnabled := schematicClient.CheckFlag(ctx, "feature-flag-key", map[string]string{
    "id": "company-id",
}, map[string]string{
    "id": "user-id",
})
```

**Java:**
```java
boolean isEnabled = schematic.checkFlag(
    "feature-flag-key",
    Map.of("id", "company-id"),
    Map.of("id", "user-id")
);
```

**C#:**
```csharp
bool isEnabled = await schematic.CheckFlag(
    "feature-flag-key",
    new Dictionary<string, string> { { "id", "company-id" } },
    new Dictionary<string, string> { { "id", "user-id" } }
);
```

## Identify Events

Identify upserts user and company context. **Note:** Identify events are most commonly used from the frontend SDK. On the backend, they can be useful in contexts where there isn't a frontend session (e.g., AI agents, background jobs, or server-side user context updates).

**Important:** Before using `identify`, you must configure how users and companies are identified in Schematic. See [Key Management](https://docs.schematichq.com/developer_resources/key_management) for details on setting up user and company keys.

**Ask the user:** What keys have you configured in Schematic for identifying users and companies? (e.g., `app_id`, `internal_id`, `stripe_customer_id`, etc.)

**Node/TypeScript:**
```typescript
await schematic.identify({
  // Use the keys you configured in Schematic (e.g., "id", "email", etc.)
  keys: { id: "user-db-id" },
  name: "Jane Doe",
  traits: { email: "jane@example.com", role: "admin" },
  company: {
    // Use the keys you configured in Schematic for companies
    keys: { id: "company-db-id" },
    name: "Acme Corp",
    traits: { plan: "enterprise", industry: "saas" },
  },
});
```

**Python:**
```python
client.identify(
    # Use the keys you configured in Schematic (e.g., "id", "email", etc.)
    keys={"id": "user-db-id"},
    name="Jane Doe",
    traits={"email": "jane@example.com", "role": "admin"},
    company={
        # Use the keys you configured in Schematic for companies
        "keys": {"id": "company-db-id"},
        "name": "Acme Corp",
        "traits": {"plan": "enterprise", "industry": "saas"},
    },
)
```

**Go:**
```go
schematicClient.Identify(ctx, &schematicgo.EventBodyIdentify{
    // Use the keys you configured in Schematic (e.g., "id", "email", etc.)
    Keys:  map[string]string{"id": "user-db-id"},
    Name:  schematicgo.String("Jane Doe"),
    Traits: map[string]any{"email": "jane@example.com", "role": "admin"},
    Company: &schematicgo.EventBodyIdentifyCompany{
        // Use the keys you configured in Schematic for companies
        Keys:   map[string]string{"id": "company-db-id"},
        Name:   schematicgo.String("Acme Corp"),
        Traits: map[string]any{"plan": "enterprise"},
    },
})
```

**Java:**
```java
schematic.identify(
    // Use the keys you configured in Schematic (e.g., "id", "email", etc.)
    Map.of("id", "user-db-id"),
    EventBodyIdentifyCompany.builder()
        // Use the keys you configured in Schematic for companies
        .keys(Map.of("id", "company-db-id"))
        .name("Acme Corp")
        .traits(Map.of("plan", "enterprise"))
        .build(),
    "Jane Doe",
    Map.of("email", "jane@example.com", "role", "admin")
);
```

**C#:**
```csharp
schematic.Identify(
    // Use the keys you configured in Schematic (e.g., "id", "email", etc.)
    keys: new Dictionary<string, string> { { "id", "user-db-id" } },
    company: new EventBodyIdentifyCompany {
        // Use the keys you configured in Schematic for companies
        Keys = new Dictionary<string, string> { { "id", "company-db-id" } },
        Name = "Acme Corp",
        Traits = new Dictionary<string, object> { { "plan", "enterprise" } },
    },
    name: "Jane Doe",
    traits: new Dictionary<string, object> { { "email", "jane@example.com" } }
);
```

## Track Events

Track records usage for metered entitlements. Include `quantity` for metered features.

**Important:** Use the same keys you configured in Schematic for identifying companies and users. See [Key Management](https://docs.schematichq.com/developer_resources/key_management) for details.

**Node/TypeScript:**
```typescript
await schematic.track({
  event: "api-call",
  // Use the keys you configured in Schematic for companies/users
  company: { id: "company-id" },
  user: { id: "user-id" },
  quantity: 1,
});
```

**Python:**
```python
client.track(
    event="api-call",
    # Use the keys you configured in Schematic for companies/users
    company={"id": "company-id"},
    user={"id": "user-id"},
    quantity=1,
)
```

**Go:**
```go
schematicClient.Track(ctx, &schematicgo.EventBodyTrack{
    Event: "api-call",
    // Use the keys you configured in Schematic for companies/users
    Company: map[string]string{"id": "company-id"},
    User:    map[string]string{"id": "user-id"},
    Quantity: schematicgo.Int(1),
})
```

**Java:**
```java
schematic.track("api-call",
    // Use the keys you configured in Schematic for companies/users
    Map.of("id", "company-id"),
    Map.of("id", "user-id"),
    null,  // traits
    1      // quantity
);
```

**C#:**
```csharp
schematic.Track(
    eventName: "api-call",
    // Use the keys you configured in Schematic for companies/users
    company: new Dictionary<string, string> { { "id", "company-id" } },
    user: new Dictionary<string, string> { { "id", "user-id" } },
    quantity: 1
);
```

## Webhook Verification

Schematic signs webhooks with HMAC-SHA256. Always verify signatures.

**Node/TypeScript (Express):**
```typescript
import { verifyWebhookSignature } from "@schematichq/schematic-typescript-node";

app.post("/webhooks/schematic", express.raw({ type: "application/json" }), (req, res) => {
  try {
    verifyWebhookSignature(req, process.env.SCHEMATIC_WEBHOOK_SECRET, req.body);
    const event = JSON.parse(req.body.toString());
    // Handle event
    res.sendStatus(200);
  } catch (err) {
    res.sendStatus(400);
  }
});
```

**Python (Flask):**
```python
from schematic.webhook import verify_webhook_signature

@app.route("/webhooks/schematic", methods=["POST"])
def handle_webhook():
    try:
        verify_webhook_signature(request, os.environ["SCHEMATIC_WEBHOOK_SECRET"])
        event = request.get_json()
        # Handle event
        return "", 200
    except Exception:
        return "", 400
```

**Go:**
```go
import "github.com/schematichq/schematic-go/webhooks"

func handleWebhook(w http.ResponseWriter, r *http.Request) {
    err := webhooks.VerifyWebhookSignature(r, os.Getenv("SCHEMATIC_WEBHOOK_SECRET"))
    if err != nil {
        http.Error(w, "Invalid signature", http.StatusBadRequest)
        return
    }
    // Handle event
    w.WriteHeader(http.StatusOK)
}
```

**Java (Spring):**
```java
import com.schematic.webhook.WebhookVerifier;
import com.schematic.webhook.WebhookSignatureException;

@PostMapping("/webhooks/schematic")
public ResponseEntity<Void> handleWebhook(@RequestBody String body, @RequestHeader Map<String, String> headers) {
    try {
        WebhookVerifier.verifyWebhookSignature(body, headers, System.getenv("SCHEMATIC_WEBHOOK_SECRET"));
        // Handle event
        return ResponseEntity.ok().build();
    } catch (WebhookSignatureException e) {
        return ResponseEntity.badRequest().build();
    }
}
```

**C#:**
```csharp
using SchematicHQ.Client.Webhooks;

[HttpPost("webhooks/schematic")]
public IActionResult HandleWebhook([FromBody] string body, [FromHeader] Dictionary<string, string> headers) {
    try {
        WebhookVerifier.VerifyWebhookSignature(body, headers, Environment.GetEnvironmentVariable("SCHEMATIC_WEBHOOK_SECRET"));
        // Handle event
        return Ok();
    } catch (WebhookSignatureException) {
        return BadRequest();
    }
}
```

## Advanced Configuration

### Flag Defaults (fallback when API is unreachable)

**Node:** `new SchematicClient({ apiKey, flagDefaults: { "my-flag": true } })`
**Python:** `Schematic(api_key, SchematicConfig(flag_defaults={"my-flag": True}))`
**Go:** `option.WithDefaultFlagValues(map[string]bool{"my-flag": true})`
**Java:** `Schematic.builder().apiKey(key).flagDefaults(Map.of("my-flag", true)).build()`
**C#:** `new Schematic(key, new ClientOptions { FlagDefaults = new() { { "my-flag", true } } })`

### Offline Mode (no network calls, uses defaults)

**Node:** `new SchematicClient({ apiKey, offline: true })`
**Python:** `Schematic(api_key, SchematicConfig(offline=True))`
**Go:** `option.WithOfflineMode()`
**Java:** `Schematic.builder().apiKey(key).offline(true).build()`
**C#:** `new Schematic(key, new ClientOptions { Offline = true })`

### Cache Configuration

All SDKs use a local in-memory cache (1000 items, 5s TTL) by default. To disable:

**Node:** `new SchematicClient({ apiKey, cacheProviders: { flagChecks: [] } })`
**Python:** `Schematic(api_key, SchematicConfig(cache_providers=[]))`
**Go:** `option.WithDisabledFlagCheckCache()`
**Java:** `Schematic.builder().apiKey(key).cacheProviders(List.of()).build()`
**C#:** `new Schematic(key, new ClientOptions { CacheProviders = new() })`

## Key Gotchas

1. **Always close/shutdown the client** — it runs background event buffers that need cleanup
2. **Use environment variables** for API keys — never hardcode secrets
3. **Company and user keys** are how Schematic identifies entities — use stable IDs from your database
4. **Track events are non-blocking** — they're buffered and sent in batches (default every 5s)
5. **Webhook secrets** are different from API keys — find them in Schematic dashboard under Webhooks
