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
# Add the marketplace
/plugin marketplace add SchematicHQ/schematic-ai-tooling

# Install the plugin
/plugin install schematic-ai-tools@schematic-ai-tools
```

### Configure your API key

The plugin includes the [Schematic MCP server](https://github.com/schematichq/schematic-mcp) for querying and managing your Schematic account directly from Claude. To use it, you'll need a Schematic API key:

1. Sign up or log in at [schematichq.com](https://schematichq.com)
2. Go to **Settings > API Keys** and create a new key
3. Set it in your terminal before running Claude Code:

```bash
export SCHEMATIC_API_KEY="your-api-key"
```

To avoid setting this every time, add the line above to your shell profile (`~/.zshrc` or `~/.bashrc`), then restart your terminal.

Once configured, the MCP server lets you list flags, check entitlements, manage companies and plans, and more — all from within Claude. The skills also auto-activate when you ask about feature flags, entitlements, tracking, checkout, or other Schematic integration tasks.

## Install for Cursor

Copy the `.cursorrules` file to your project root. This file contains condensed integration rules covering backend SDKs (Node, Python, Go, Java, C#), frontend SDKs (React, Vue, JS), and embedded billing components. For detailed examples and comprehensive documentation, see the `skills/` directory.

```bash
curl -o .cursorrules https://raw.githubusercontent.com/schematichq/schematic-ai-tools/main/.cursorrules
```

Or manually copy the `.cursorrules` file from this repo into your project root.

## Using without the plugin

If you'd prefer to use the MCP server standalone (without the plugin), you can add it directly:

```bash
claude mcp add --transport stdio schematic-mcp \
  --env SCHEMATIC_API_KEY=your-api-key \
  -- npx -y @schematichq/schematic-mcp
```

See the [Schematic MCP documentation](https://github.com/schematichq/schematic-mcp) for more details.

## Links

- [Schematic Docs](https://docs.schematichq.com)
- [Schematic MCP](https://github.com/schematichq/schematic-mcp) ([npm](https://www.npmjs.com/package/@schematichq/schematic-mcp)) — runtime operations via MCP
- [Node SDK](https://github.com/schematichq/schematic-node)
- [Python SDK](https://github.com/schematichq/schematic-python)
- [Go SDK](https://github.com/schematichq/schematic-go)
- [Java SDK](https://github.com/schematichq/schematic-java)
- [C# SDK](https://github.com/schematichq/schematic-csharp)
- [JS/React/Vue SDK](https://github.com/schematichq/schematic-js)
