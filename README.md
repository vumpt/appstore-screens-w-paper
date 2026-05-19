# App Store Screenshots (Paper)

An agent-driven skill for generating production App Store screenshots using [Paper](https://paper.design). Your agent reads your Paper design files, creates marketing artboards, and exports them at the exact resolutions Apple requires — all through a guided, stage-gated workflow.

## Quick Start

### 1. Install the skill

Choose **Global** (available everywhere) or **Project-bound** (only in one project).

#### Global install (recommended)

```bash
# OpenCode
mkdir -p ~/.agents/skills && \
  git clone https://github.com/vumpt/appstore-screens-w-paper.git \
  ~/.agents/skills/app-store-screenshots

# Claude Code
mkdir -p ~/.claude/skills && \
  git clone https://github.com/vumpt/appstore-screens-w-paper.git \
  ~/.claude/skills/app-store-screenshots

# Cursor
mkdir -p ~/.cursor/agents/skills && \
  git clone https://github.com/vumpt/appstore-screens-w-paper.git \
  ~/.cursor/agents/skills/app-store-screenshots

# Generic / other harness
mkdir -p ~/.agents/skills && \
  git clone https://github.com/vumpt/appstore-screens-w-paper.git \
  ~/.agents/skills/app-store-screenshots
```

#### Project-bound install

First, navigate to your project folder, then run the snippet for your harness:

```bash
cd /path/to/your/project
```

```bash
# OpenCode
mkdir -p .agents/skills && \
  git clone https://github.com/vumpt/appstore-screens-w-paper.git \
  .agents/skills/app-store-screenshots

# Claude Code
mkdir -p .claude/skills && \
  git clone https://github.com/vumpt/appstore-screens-w-paper.git \
  .claude/skills/app-store-screenshots

# Cursor
mkdir -p .cursor/agents/skills && \
  git clone https://github.com/vumpt/appstore-screens-w-paper.git \
  .cursor/agents/skills/app-store-screenshots

# Generic / other harness
mkdir -p .agents/skills && \
  git clone https://github.com/vumpt/appstore-screens-w-paper.git \
  .agents/skills/app-store-screenshots
```

### 2. Hook it up to your agent harness

| Harness | How to connect Paper MCP |
|---------|--------------------------|
| **OpenCode** | Add to `opencode.json`: `"mcp": { "paper": { "type": "remote", "url": "http://127.0.0.1:29979/mcp", "enabled": true } }` |
| **Claude Code CLI** | `/plugin marketplace add paper-design/agent-plugins` then `/plugin install paper-desktop@paper` |
| **Claude Code Desktop** | "Customize" sidebar → Add plugin → marketplace `paper-design/agent-plugins` → install Paper Desktop |
| **Cursor** | `/add-plugin paper-desktop` or install via Cursor Marketplace |
| **Codex** | Settings > MCP Servers → Streamable HTTP → name: `paper`, URL: `http://127.0.0.1:29979/mcp` |
| **VS Code Copilot** | `.vscode/mcp.json`: `"servers": { "paper": { "type": "http", "url": "http://127.0.0.1:29979/mcp" } }` |

### 3. Tell your agent to use it

> "Load the app-store-screenshots skill and generate App Store screenshots for my app."

Your agent will:
1. **Discover** — read your Paper project and codebase to understand your app
2. **Strategize** — propose a screenshot sequence (you approve or edit)
3. **Design** — create iPhone marketing artboards in Paper (you review)
4. **Refactor** — adapt approved designs for iPad dimensions
5. **Export** — render production PNGs at exact App Store sizes (1284×2778 and 2065×2751)
6. **(Optional) Variants** — generate A/B/C marketing copy variants for Product Page Optimization

## What you need

- **Paper Desktop** running with MCP enabled ([download](https://paper.design))
- **ffmpeg** installed (for iPad post-processing)
- A **Paper project** with at least one existing UI artboard (so the agent can read your design system)

## How it works

The skill is a set of structured instructions (`SKILL.md` + reference prompts in `references/`) that guides your agent through a 5–7 stage pipeline:

- **State is implicit** — progress is tracked by artboard names in Paper (`screen_01`, `ipad_01`, etc.), so you can stop and resume anytime.
- **Human gates** — the agent pauses at key decision points (strategy approval, design review) so you stay in control.
- **Marketing, not UI** — the agent is explicitly instructed to design billboard-style slides with huge type and bold whitespace, not app interfaces.

For full technical details — MCP tools, config schemas, troubleshooting, and per-stage prompting guidance — see `SKILL.md`.

## Repository structure

```
app-store-screenshots/
├── SKILL.md                        # Full skill spec for agents
├── references/
│   ├── discovery.txt               # Stage 1: audit app & Paper project
│   ├── copywriter.txt              # Stage 2: craft marketing angles
│   ├── designer_iphone.txt         # Stage 3: design iPhone artboards
│   ├── designer_ipad.txt           # Stage 5: adapt for iPad
│   ├── variant.txt                 # Stage 7: A/B/C copy variants
│   ├── infer_config.txt            # Auto-generate config.yaml
│   └── REFERENCE.md                # Quick index of reference files
├── assets/
│   └── config.yaml.template        # Optional config template
└── README.md                       # This file
```

## License

MIT
