# Stable Baseline for Claude Code

Connect [Stable Baseline](https://stablebaseline.io) to [Claude Code](https://claude.ai/code): the MCP server is configured for you, and eleven skills teach Claude how to drive it well.

## What you get

- **MCP auto-configuration.** The plugin registers the `sb` MCP server, so no manual config is needed.
- **Eleven skills.** Claude picks them up on its own when a request matches, and you can invoke one by name (see below).
- **The full Stable Baseline tool surface.** 196 tools across 18 categories: living documents in CDMD, 29 diagram engines, whiteboards, AI-designed decks and illustrations, brand kits, plans and tasks, improvements and risks, and the shared Knowledge Graph.

## Installation

### 1. Add the marketplace

```
/plugin marketplace add stablebaseline/claude-code
```

### 2. Install the plugin

```
/plugin install stable-baseline@stablebaseline
```

### 3. Set your API key

Add to your shell profile (`~/.bashrc`, `~/.zshrc`, etc.):

```bash
export STABLE_BASELINE_API_KEY="sta_your_api_key_here"
```

Mint a key at [Settings > MCP Setup](https://app.stablebaseline.io/settings/mcp-setup). Keys are prefixed `sta_` and can be scoped to a workspace, project or folder, with read, write and delete chosen per content type.

If you would rather not hold a key, the server also supports OAuth 2.1 with Dynamic Client Registration; any MCP client that implements it can sign in without one.

### 4. Run setup

```
/stable-baseline:sb-setup
```

This analyses your repository, creates a documentation structure that fits it, and writes auto-sync rules into `AGENTS.md`.

## Skills

These are **skills**, not slash commands. Claude invokes them automatically when your request matches one, so most of the time you simply ask for what you want. To run one deliberately, prefix it with the plugin name.

| Skill | What it does |
|---|---|
| `/stable-baseline:sb-setup` | Full onboarding: analyse the repo, create docs, configure sync |
| `/stable-baseline:sb-sync` | Regenerate the AGENTS.md sync rules, preserving existing content |
| `/stable-baseline:sb-create-doc` | Create a document, with CDMD formatting guidance |
| `/stable-baseline:sb-edit-doc` | Edit a document with merge-safe patching and optimistic locking |
| `/stable-baseline:sb-create-diagram` | Create a diagram, with type selection and DSL guidance |
| `/stable-baseline:sb-create-whiteboard` | Build a whiteboard by hand, or hand a goal to the multi-agent designer |
| `/stable-baseline:sb-edit-whiteboard` | Update a whiteboard: add, move or remove elements |
| `/stable-baseline:sb-search` | Search the Knowledge Graph, your shared company brain |
| `/stable-baseline:sb-manage-images` | Upload and manage images inside documents |
| `/stable-baseline:sb-manage-data` | Upload CSV, JSON or TSV data for Vega and Vega-Lite charts |
| `/stable-baseline:sb-update` | Capture changes and decisions from the current conversation into the docs |

Plans, tasks, improvements, decks and brand kits have no dedicated skill yet, but every tool for them is available through the MCP server, so you can ask for them directly.

## Costs worth knowing

Most tools are free. Two are not, and both tell you the price before doing anything:

- **`autoDesignWhiteboard`** costs 50 credits. It uses a two-call handshake: the first call returns a quote and your balance and spends nothing, and only a second call with `confirm: true` runs it. It then works in the background, so the board fills in over one to three minutes. Credits are refunded automatically if it fails server-side.
- **Deck design and generated illustrations** are similarly quoted before they run.

## Other MCP clients

This plugin is for Claude Code. For Cursor, VS Code, Windsurf or any other MCP client, point it at the server directly:

```
https://api.stablebaseline.io/functions/v1/cloud-serve/mcp
```

See the [setup guide](https://stablebaseline.io/docs/mcp/setup). The authoritative OAuth configuration is always the server's own metadata at [`/.well-known/oauth-authorization-server`](https://api.stablebaseline.io/functions/v1/cloud-serve/.well-known/oauth-authorization-server); do not hand-copy endpoint URLs.

## License

Apache License 2.0. See [LICENSE](./LICENSE). Use of the Stable Baseline service is governed by the [Stable Baseline Terms](https://stablebaseline.io/terms).

## Links

- [Documentation](https://stablebaseline.io/docs)
- [Claude Code recipe](https://stablebaseline.io/docs/recipes/claude)
- [MCP setup guide](https://stablebaseline.io/docs/mcp/setup)
- [Tool catalogue](https://stablebaseline.io/docs/mcp/tools)
