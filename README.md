# Schematic AI Tools

AI coding skills for integrating with [Schematic](https://schematichq.com) SDKs. Teaches AI assistants how to write code for feature flags, entitlements, usage tracking, and billing UI components.

## What's Included

| Skill | Covers |
|---|---|
| **Backend SDK** | Node, Python, Go, Java, C# — flag checks, identify, track, webhooks |
| **Frontend SDK** | React, Vue, JS — hooks, composables, providers, real-time flags |
| **Components** | Embedded checkout, pricing tables, customer portal (React) |

## Install for Claude Code

```bash
# Add from marketplace
/plugin marketplace add schematichq/schematic-ai-tools

# Install the plugin
/plugin install schematic-ai-tools
```

Once installed, the skills auto-activate when you ask about feature flags, entitlements, tracking, checkout, or other Schematic integration tasks.

## Install for Cursor

Copy the `.cursorrules` file to your project root:

```bash
curl -o .cursorrules https://raw.githubusercontent.com/schematichq/schematic-ai-tools/main/.cursorrules
```

Or manually copy the `.cursorrules` file from this repo into your project root.

## Runtime Operations

These skills teach AI assistants how to **write integration code**. For runtime operations like querying your Schematic account data (listing flags, checking company entitlements, viewing plans), use the [Schematic MCP server](https://github.com/schematichq/schematic-mcp).

## Links

- [Schematic Docs](https://docs.schematichq.com)
- [Schematic MCP](https://github.com/schematichq/schematic-mcp) — runtime operations via MCP
- [Node SDK](https://github.com/schematichq/schematic-node)
- [Python SDK](https://github.com/schematichq/schematic-python)
- [Go SDK](https://github.com/schematichq/schematic-go)
- [Java SDK](https://github.com/schematichq/schematic-java)
- [C# SDK](https://github.com/schematichq/schematic-csharp)
- [JS/React/Vue SDK](https://github.com/schematichq/schematic-js)
