---
description: "Add Schematic feature flags, entitlements, identify, or track events in frontend code (JavaScript, React, Vue)"
---

# Schematic Frontend SDK Integration

## Overview

Schematic's frontend SDKs let you check feature flags, track usage, and identify users client-side. They use a **publishable key** (not a secret API key) and are safe to expose in browser code.

Three packages:
- **`@schematichq/schematic-js`** — Core JS client (framework-agnostic)
- **`@schematichq/schematic-react`** — React hooks and provider
- **`@schematichq/schematic-vue`** — Vue composables and plugin

## Framework Detection

Show examples for the framework the user is working with. If unclear, ask whether they're using React, Vue, or plain JS.

## Installation

**React:**
```bash
npm install @schematichq/schematic-react
```

**Vue:**
```bash
npm install @schematichq/schematic-vue
```

**Plain JS:**
```bash
npm install @schematichq/schematic-js
```

## Setup

### React — SchematicProvider

Wrap your app (or a subtree) with `SchematicProvider`:

```tsx
import { SchematicProvider } from "@schematichq/schematic-react";

function App() {
  return (
    <SchematicProvider publishableKey={process.env.NEXT_PUBLIC_SCHEMATIC_PUBLISHABLE_KEY}>
      <YourApp />
    </SchematicProvider>
  );
}
```

### Vue — SchematicPlugin

Register the plugin in your app entry:

```typescript
import { createApp } from "vue";
import { SchematicPlugin } from "@schematichq/schematic-vue";
import App from "./App.vue";

const app = createApp(App);
app.use(SchematicPlugin, {
  publishableKey: import.meta.env.VITE_SCHEMATIC_PUBLISHABLE_KEY,
});
app.mount("#app");
```

### Plain JS — Direct Client

```typescript
import { Schematic } from "@schematichq/schematic-js";

const schematic = new Schematic("your-publishable-key");

// Always clean up
window.addEventListener("beforeunload", () => schematic.cleanup());
```

## Identify Users

Call identify after login or when user context changes. This sets the context for all subsequent flag checks.

### React

```tsx
import { useSchematicEvents } from "@schematichq/schematic-react";

function LoginHandler() {
  const { identify } = useSchematicEvents();

  const onLogin = (user) => {
    identify({
      keys: { id: user.id },
      traits: { email: user.email, name: user.name },
      company: {
        keys: { id: user.companyId },
        name: user.companyName,
        traits: { plan: user.plan },
      },
    });
  };

  return <button onClick={() => onLogin(currentUser)}>Log in</button>;
}
```

### Vue

```typescript
import { useSchematicEvents } from "@schematichq/schematic-vue";

const { identify } = useSchematicEvents();

function onLogin(user) {
  identify({
    keys: { id: user.id },
    traits: { email: user.email, name: user.name },
    company: {
      keys: { id: user.companyId },
      name: user.companyName,
      traits: { plan: user.plan },
    },
  });
}
```

### Plain JS

```typescript
schematic.identify({
  keys: { id: "user-id" },
  traits: { email: "jane@example.com" },
  company: {
    keys: { id: "company-id" },
    name: "Acme Corp",
  },
});
```

## Check Feature Flags

### React — useSchematicFlag

```tsx
import { useSchematicFlag } from "@schematichq/schematic-react";

function FeatureGate() {
  const isEnabled = useSchematicFlag("feature-flag-key");

  if (!isEnabled) return null;
  return <NewFeature />;
}
```

With fallback value:

```tsx
const isEnabled = useSchematicFlag("feature-flag-key", { fallback: false });
```

### React — useSchematicEntitlement

For metered features with usage limits:

```tsx
import { useSchematicEntitlement } from "@schematichq/schematic-react";

function UsageDisplay() {
  const { isEntitled, usage, limit } = useSchematicEntitlement("api-calls");

  return (
    <div>
      {isEntitled ? "Active" : "Limit reached"}
      <span>{usage} / {limit} used</span>
    </div>
  );
}
```

### Vue — useSchematicFlag

```vue
<script setup>
import { useSchematicFlag } from "@schematichq/schematic-vue";

const isEnabled = useSchematicFlag("feature-flag-key");
</script>

<template>
  <NewFeature v-if="isEnabled" />
</template>
```

### Vue — useSchematicEntitlement

```vue
<script setup>
import { useSchematicEntitlement } from "@schematichq/schematic-vue";

const { isEntitled, usage, limit } = useSchematicEntitlement("api-calls");
</script>

<template>
  <div>
    <span>{{ usage }} / {{ limit }} used</span>
  </div>
</template>
```

### Plain JS

```typescript
const isEnabled = await schematic.checkFlag({ key: "feature-flag-key" });
```

## Track Events

### React

```tsx
import { useSchematicEvents } from "@schematichq/schematic-react";

function ActionButton() {
  const { track } = useSchematicEvents();

  return (
    <button onClick={() => track({ event: "api-call", quantity: 1 })}>
      Make API Call
    </button>
  );
}
```

### Vue

```typescript
import { useSchematicEvents } from "@schematichq/schematic-vue";

const { track } = useSchematicEvents();

function onAction() {
  track({ event: "api-call", quantity: 1 });
}
```

### Plain JS

```typescript
schematic.track({ event: "api-call", quantity: 1 });
```

## Loading States

### React — useSchematicIsPending

```tsx
import { useSchematicIsPending } from "@schematichq/schematic-react";

function App() {
  const isPending = useSchematicIsPending();

  if (isPending) return <Spinner />;
  return <MainContent />;
}
```

### Vue — useSchematicIsPending

```vue
<script setup>
import { useSchematicIsPending } from "@schematichq/schematic-vue";

const isPending = useSchematicIsPending();
</script>

<template>
  <Spinner v-if="isPending" />
  <MainContent v-else />
</template>
```

## WebSocket Mode

For real-time flag updates without polling, enable WebSocket mode:

### React

```tsx
<SchematicProvider publishableKey={key} useWebSocket={true}>
  <App />
</SchematicProvider>
```

### Vue

```typescript
app.use(SchematicPlugin, {
  publishableKey: key,
  useWebSocket: true,
});
```

### Plain JS

```typescript
const schematic = new Schematic("key", { useWebSocket: true });
```

WebSocket mode maintains a persistent connection and pushes flag changes instantly. It auto-reconnects with exponential backoff.

## Advanced Options

### Debug Mode

Logs all flag evaluations and events to the console:

```typescript
// React
<SchematicProvider publishableKey={key} debug={true}>

// Vue
app.use(SchematicPlugin, { publishableKey: key, debug: true });

// JS
new Schematic("key", { debug: true });
```

### Offline Mode

Disables network requests, uses flag defaults:

```typescript
// React
<SchematicProvider publishableKey={key} offline={true}>

// Vue
app.use(SchematicPlugin, { publishableKey: key, offline: true });

// JS
new Schematic("key", { offline: true });
```

### Flag Value Defaults

Fallback values when the API is unreachable:

```typescript
// React
<SchematicProvider publishableKey={key} flagValueDefaults={{ "my-flag": true }}>

// Vue
app.use(SchematicPlugin, { publishableKey: key, flagValueDefaults: { "my-flag": true } });

// JS
new Schematic("key", { flagValueDefaults: { "my-flag": true } });
```

## Available Hooks / Composables

| Hook / Composable | Returns | Description |
|---|---|---|
| `useSchematicFlag(key)` | `boolean` (React) / `Ref<boolean>` (Vue) | Check a single flag |
| `useSchematicEntitlement(key)` | `{ isEntitled, usage, limit }` | Check entitlement with usage |
| `useSchematicEvents()` | `{ identify, track }` | Send identify/track events |
| `useSchematicIsPending()` | `boolean` / `Ref<boolean>` | True while initial data loads |
| `useSchematicContext()` | `{ context, setContext }` | Read/update evaluation context |
| `useSchematic()` | `{ client }` | Access the raw Schematic client |

## Key Gotchas

1. **Use publishable key** — not the secret API key. Publishable keys are safe for client-side code
2. **Call identify before checking flags** — flags evaluate against the identified user/company context
3. **Always clean up** — call `cleanup()` (JS) or let the provider handle it (React/Vue)
4. **Vue refs auto-unwrap in templates** — `useSchematicFlag` returns a `Ref<boolean>`, but Vue templates unwrap it automatically
5. **Events are buffered** — track events are queued and sent in batches, not immediately
6. **SSR considerations** — flag checks return defaults during server rendering; hydration picks up real values client-side
