# Stable Baseline — Claude Code Plugin

Connect [Stable Baseline](https://stablebaseline.io) to [Claude Code](https://claude.ai/code) with slash commands and automatic MCP configuration.

## What you get

- **MCP auto-configuration** — The plugin configures the `sb` MCP server automatically
- **8 slash commands** — `/sb-setup`, `/sb-sync`, `/sb-create-doc`, `/sb-create-diagram`, `/sb-edit-doc`, `/sb-manage-images`, `/sb-manage-data`, `/sb-update`
- **All 34+ MCP tools** — Create documents, insert diagrams, manage images, upload data files, and more

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

Get an API key from your [Stable Baseline workspace settings](https://app.stablebaseline.io/).

### 4. Run setup

```
/sb-setup
```

This analyses your repository, creates tailored documentation, and configures auto-sync rules in `AGENTS.md`.

## Slash commands

| Command | What it does |
|---|---|
| `/sb-setup` | Full onboarding — analyse repo, create docs, configure sync |
| `/sb-sync` | Regenerate AGENTS.md sync rules (preserves existing content) |
| `/sb-create-doc` | Create a single document with CDMD formatting guidance |
| `/sb-create-diagram` | Create a diagram with type selection and DSL guidance |
| `/sb-edit-doc` | Edit an existing document with merge-safe patching |
| `/sb-manage-images` | Upload and manage images in documents |
| `/sb-manage-data` | Upload CSV/JSON/TSV data files for Vega/Vega-Lite charts |
| `/sb-update` | Update docs with changes from the current conversation |

## Other MCP clients

This plugin is for Claude Code. If you use Cursor, VS Code, Windsurf, or another MCP-compatible IDE, use the MCP prompts directly — see [setup docs](https://stablebaseline.io/docs/mcp/setup).

## Links

- [Documentation](https://stablebaseline.io/docs)
- [Claude Code recipe](https://stablebaseline.io/docs/recipes/claude)
- [MCP setup guide](https://stablebaseline.io/docs/mcp/setup)
