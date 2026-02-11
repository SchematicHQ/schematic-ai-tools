---
description: "Add Schematic embedded billing UI — checkout, pricing tables, customer portal, or subscription management components"
---

# Schematic Components Integration

## Overview

Schematic Components are pre-built React UI components for billing and subscription management. They render checkout flows, pricing tables, and customer portals that connect to your Schematic + Stripe setup.

**Key pattern**: Your backend generates a temporary access token, your frontend uses it to render the component. This keeps your API key server-side while letting the component interact with Schematic's API.

## Installation

```bash
npm install @schematichq/schematic-components @stripe/stripe-js
```

`@stripe/stripe-js` is a required peer dependency for payment processing.

## Architecture: Token Exchange

```
┌─────────────┐         ┌─────────────┐         ┌──────────┐
│   Frontend   │──(1)──▶│   Backend    │──(2)──▶│ Schematic │
│  (React app) │         │  (your API)  │         │   API     │
│              │◀──(3)──│              │◀──(3)──│           │
│              │         └─────────────┘         └──────────┘
│  Renders     │
│  SchematicEmbed
│  with token  │
└─────────────┘
```

1. Frontend requests a temporary token from your backend
2. Backend calls Schematic API with your secret API key to issue a temporary token
3. Token is returned to frontend
4. Frontend passes token to `SchematicEmbed`

## Backend: Issue Temporary Access Token

The backend uses your Schematic secret API key to generate a short-lived token scoped to a specific company.

**Node/TypeScript:**
```typescript
import { SchematicClient } from "@schematichq/schematic-typescript-node";

const schematic = new SchematicClient({
  apiKey: process.env.SCHEMATIC_API_KEY,
});

// API endpoint to issue a temporary token
app.get("/schematic-token", async (req, res) => {
  const response = await schematic.accesstokens.issueTemporaryAccessToken({
    lookup: { companyId: req.user.companyId },
  });
  res.json({ token: response.data.token });
});
```

**Python:**
```python
from schematic.client import Schematic

client = Schematic(os.environ["SCHEMATIC_API_KEY"])

@app.route("/schematic-token")
def get_token():
    response = client.accesstokens.issue_temporary_access_token(
        lookup={"company_id": request.user.company_id},
    )
    return {"token": response.data.token}
```

**Go:**
```go
resp, err := schematicClient.API().Accesstokens.IssueTemporaryAccessToken(ctx,
    &schematicgo.IssueTemporaryAccessTokenRequestBody{
        Lookup: map[string]string{"company_id": companyID},
    },
)
token := resp.Data.Token
```

## Frontend: Render Components

### Basic Setup with SchematicEmbed

`SchematicEmbed` renders a component configured in the Schematic dashboard. The `id` is your component ID from Schematic (starts with `cmpn_`).

```tsx
import { SchematicEmbed } from "@schematichq/schematic-components";
import { useEffect, useState } from "react";

function BillingPage() {
  const [token, setToken] = useState<string | null>(null);

  useEffect(() => {
    fetch("/schematic-token")
      .then((res) => res.json())
      .then((data) => setToken(data.token));
  }, []);

  if (!token) return <div>Loading...</div>;

  return (
    <SchematicEmbed
      accessToken={token}
      id="cmpn_YOUR_COMPONENT_ID"
    />
  );
}
```

### Using EmbedProvider for Multiple Components

If you have multiple Schematic components on the same page, wrap them in an `EmbedProvider` to share the token:

```tsx
import { EmbedProvider, SchematicEmbed } from "@schematichq/schematic-components";

function BillingPage() {
  const [token, setToken] = useState<string | null>(null);

  useEffect(() => {
    fetch("/schematic-token")
      .then((res) => res.json())
      .then((data) => setToken(data.token));
  }, []);

  if (!token) return <div>Loading...</div>;

  return (
    <EmbedProvider accessToken={token}>
      <SchematicEmbed id="cmpn_PRICING_TABLE" />
      <SchematicEmbed id="cmpn_CUSTOMER_PORTAL" />
    </EmbedProvider>
  );
}
```

### EmbedProvider Props

```typescript
interface EmbedProviderProps {
  accessToken?: string;    // Temporary access token from your backend
  id?: string;             // Component ID (if using a single component)
  apiConfig?: {            // Optional API config overrides
    basePath?: string;
  };
  children?: React.ReactNode;
  mode?: "edit" | "view";  // "view" for production (default), "edit" for dashboard
}
```

## Component Types

Components are configured in the Schematic dashboard. The type is determined by the component configuration, not by props. Common types:

| Type | Description |
|---|---|
| **checkout** | Full checkout flow with plan selection and payment |
| **portal** | Customer portal for managing subscriptions |
| **payment** | Payment method management |
| **unsubscribe** | Cancellation flow |

## Custom API Configuration

If you're using a custom Schematic API endpoint (e.g., proxying through your backend):

```tsx
<SchematicEmbed
  accessToken={token}
  id="cmpn_YOUR_COMPONENT_ID"
  apiConfig={{ basePath: "https://your-api-proxy.com" }}
/>
```

## Full Example: Billing Page

```tsx
import { SchematicEmbed } from "@schematichq/schematic-components";
import { useEffect, useState } from "react";

export function BillingPage() {
  const [token, setToken] = useState<string | null>(null);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    async function fetchToken() {
      try {
        const res = await fetch("/api/schematic-token", {
          credentials: "include",
        });
        if (!res.ok) throw new Error("Failed to fetch token");
        const data = await res.json();
        setToken(data.token);
      } catch (err) {
        setError("Unable to load billing. Please try again.");
      }
    }
    fetchToken();
  }, []);

  if (error) return <div>{error}</div>;
  if (!token) return <div>Loading billing...</div>;

  return (
    <div>
      <h1>Billing</h1>
      <SchematicEmbed
        accessToken={token}
        id="cmpn_YOUR_COMPONENT_ID"
      />
    </div>
  );
}
```

## Key Gotchas

1. **Temporary tokens are short-lived** — fetch a new one each time the component mounts; don't cache them long-term
2. **Component IDs come from the Schematic dashboard** — they start with `cmpn_` and reference a pre-configured component
3. **Stripe peer dependency is required** — `@stripe/stripe-js` must be installed even if you don't use Stripe directly
4. **Token exchange keeps your API key secret** — never expose your Schematic secret API key in frontend code
5. **Components are React-only** — there is no Vue equivalent; for Vue apps, embed in an iframe or use a React micro-frontend
6. **The `lookup` object in token generation** identifies which company the token is scoped to — use the same keys you use in identify events
