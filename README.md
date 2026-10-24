# Stable Baseline for Claude Code

Connect [Stable Baseline](https://stablebaseline.io) to [Claude Code](https://claude.ai/code): the MCP server is configured for you, and fifteen skills teach Claude how to drive it well.

## What you get

- **MCP auto-configuration.** The plugin registers the `sb` MCP server, so no manual config is needed.
- **Fifteen skills.** Claude picks them up on its own when a request matches, and you can invoke one by name (see below).
- **The full Stable Baseline tool surface.** 222 tools across 20 categories: living documents in CDMD, 40+ diagram types, whiteboards, AI-designed decks and illustrations, brand kits, plans and tasks, improvements and risks, databases with views, forms and live dashboards, your organisation's agents (talk to them, hand them work, set them up), and the shared Knowledge Graph.

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

Create a key under Organisation settings, Connected apps, [Developers](https://app.stablebaseline.io/settings/developers), on the API keys tab. Keys are prefixed `sta_` and can be scoped to a workspace, project or folder, with Read, Edit and Delete chosen for each kind of content: Documents, Whiteboards, Plans and tasks, Improvements, and Databases and dashboards. Databases and dashboards start unticked: tick them if you want Claude to work with your databases, and choose Only some databases to hold a key to chosen databases and tables, each with its own level. Under Your agents, Use your agents is on and Set up agents is off: tick Set up agents if you want Claude to make and change agents and their duties. A key's choices are fixed once it is made, so make a new key to change them.

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
| `/stable-baseline:sb-edit-whiteboard` | Update a whiteboard: add, move or remove elements, change or undo an AI design, or export it as PowerPoint |
| `/stable-baseline:sb-search` | Find anything by name, count things in one call, list what a person owns or created, see what changed, and ask the Knowledge Graph, your shared company brain |
| `/stable-baseline:sb-manage-images` | Upload and manage images inside documents |
| `/stable-baseline:sb-manage-data` | Upload CSV, JSON or TSV data for Vega and Vega-Lite charts |
| `/stable-baseline:sb-update` | Capture changes and decisions from the current conversation into the docs |
| `/stable-baseline:sb-database` | Build a database and its tables, fields, views and forms; import CSV or Excel; query and update rows; answer a form |
| `/stable-baseline:sb-dashboard` | Build a live dashboard on a database table, check each page as a picture, download it as a PDF |
| `/stable-baseline:sb-agent` | Talk to your organisation's agents, hand them work, follow and approve their jobs, undo one, read how an agent is set up, and set up agents and duties |
| `/stable-baseline:sb-connect` | Connect a Microsoft 365 mailbox and put it to work in an agent's duty, with the duty drafted for you |

Plans, tasks, improvements, decks and brand kits have no dedicated skill yet, but every tool for them is available through the MCP server, so you can ask for them directly.

## Costs worth knowing

Most tools are free, databases and dashboards included. The ones that cost credits tell you the price before doing anything:

- **`autoDesignWhiteboard`** costs credits (78 a board at the time of writing). It uses a two-call handshake: the first call returns the exact cost, your balance and a single-use quote token and spends nothing, and only a second call with `confirm: true` and that token runs it. It then works in the background: flowcharts, mind maps and the other common kinds fill in within seconds, and other boards over one to three minutes. Changing how a finished design looks is free. Credits are refunded automatically if it fails server-side.
- **Deck design and generated illustrations** are quoted the same way before they run.
- **Agents**: asking or delegating to an agent costs no credits, within a fair-use number of questions a day. A tool with its own fee costs the same when an agent uses it, and the agent's proposal shows the fee first.
- **Duties** (work an agent does on its own) use credits: 1 when a run checks and finds nothing to do, 5 when it acts. Duties are set up in the Stable Baseline web app, or with `setUpAgentDuty` when the key has Set up agents ticked. A duty that uses a connected app is always set up in the web app.

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
- [Databases](https://stablebaseline.io/docs/databases), [Agents](https://stablebaseline.io/docs/agents), [Duties](https://stablebaseline.io/docs/duties) and [Connectors](https://stablebaseline.io/docs/connectors)
