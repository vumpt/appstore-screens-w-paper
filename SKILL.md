---
name: app-store-screenshots
description: Agent-empowered App Store screenshot pipeline. Works with any agent harness (Claude Code, Codex, Cursor, OpenCode) via the Paper MCP. Design, review, and programmatically export screenshots at production resolution.
license: MIT
compatibility: Requires Paper Desktop with MCP enabled, ffmpeg, and Paper project with existing UI artboards.
metadata:
  author: Lachlan Glasgow
  version: "0.5.0"
  hermes:
    tags: [creative, app-store, marketing, paper, design, screenshots]
    related_skills: []
---

# App Store Screenshots Pipeline

Generate App Store screenshots using Paper (paper.design) via an agent-driven, stage-gated workflow. This skill formalizes the process described in [Making Killer App Store Screenshots](https://open.substack.com/pub/glachlan/p/making-killer-app-store-screenshots).

## When to use

- You need App Store Connect screenshot assets (iPhone 6.5", iPad 13", etc.)
- You have a Paper project with existing UI artboards
- You want editable, agent-tweakable marketing images — not baked AI image outputs
- You want to run A/B/C Product Page Optimization variants programmatically

## Prerequisites

### 1. Paper Desktop app running with MCP
The agent must verify Paper MCP is accessible before proceeding.

**Important:** The Paper MCP server is bundled inside the Paper Desktop app and runs locally on `http://127.0.0.1:29979/mcp`. Connector plugins are at `github.com/paper-design/agent-plugins`.

**Agent instruction (step 0):** Call `get_basic_info` via the Paper MCP. If this fails, stop and guide the user through harness-specific setup:

| Harness | Setup command |
|---------|---------------|
| Claude Code CLI | `/plugin marketplace add paper-design/agent-plugins` then `/plugin install paper-desktop@paper` |
| Claude Code Desktop | "Customize" sidebar → Add plugin → marketplace `paper-design/agent-plugins` → install Paper Desktop |
| Cursor | `/add-plugin paper-desktop` or install via Cursor Marketplace |
| Codex | Settings > MCP Servers → Streamable HTTP → name: `paper`, URL: `http://127.0.0.1:29979/mcp` |
| OpenCode | Add to `opencode.json`: `{ "mcp": { "paper": { "type": "remote", "url": "http://127.0.0.1:29979/mcp", "enabled": true } } }` |
| VS Code Copilot | `.vscode/mcp.json`: `{ "servers": { "paper": { "type": "http", "url": "http://127.0.0.1:29979/mcp" } } }` |

If the Paper MCP is not accessible, halt. Do not proceed.

### 2. ffmpeg
`ffmpeg` must be installed for post-processing iPad exports to exact App Store dimensions.

## Setup

### 1. Paper MCP Configuration
Add Paper MCP to your harness config (see Prerequisites above).

### 2. Prepare Your App
You need a Paper project with at least one existing UI artboard for the app. The agent will read from this to understand:
- Design system (colors, typography, spacing)
- Core UI patterns
- Feature set

### 3. (Optional) Config File
Create a `config.yaml` in your project to specify (see [assets/config.yaml.template](assets/config.yaml.template)):

```yaml
app:
  name: "MyApp"
  tagline: "One-sentence description"

screenshots:
  sequence:
    - feature: "Feature Name"
      angle: "marketing angle" # e.g. "simplicity", "speed", "privacy"
    - feature: "Another Feature"
      angle: "another angle"
```

If you skip the config, the agent will infer features and angles from Paper and ask you to approve before proceeding.

## Paper MCP Capabilities Reference

The Paper MCP server exposes the following tools:

| Tool | Purpose |
|------|---------|
| `get_basic_info` | File name, page name, node count, artboard list with dimensions |
| `get_selection` | Details about currently selected nodes |
| `get_node_info` | Details for a node by ID (size, visibility, lock, parent, children, text content) |
| `get_children` | Direct children of a node |
| `get_tree_summary` | Compact text summary of a node's subtree hierarchy |
| `get_screenshot` | Screenshot of a node by ID (base64 image; 1x or 2x only). **Preview only — not for production export.** |
| `get_jsx` | JSX for a node and its descendants (Tailwind or inline-styles) |
| `get_computed_styles` | Computed CSS styles for one or more nodes (batch) |
| `get_fill_image` | Image data from a node that has an image fill (base64 JPEG) |
| `get_font_family_info` | Look up whether a font family is available; inspect weights and styles |
| `get_guide` | Retrieve guided workflows for topics (e.g. `figma-import`) |
| `find_placement` | Suggested x/y on the canvas to place a new artboard without overlap |
| `create_artboard` | Create a new artboard; optional name and styles |
| `write_html` | Parse HTML and add or replace nodes |
| `set_text_content` | Set text content of one or more Text nodes (batch) |
| `rename_nodes` | Rename one or more layers (batch) |
| `duplicate_nodes` | Deep-clone nodes; returns new IDs and a descendant ID map |
| `update_styles` | Update CSS styles on one or more nodes |
| `delete_nodes` | Delete one or more nodes and all their descendants |
| `start_working_on_nodes` | Mark artboards as being worked on (show indicator) |
| `finish_working_on_nodes` | Clear the working indicator from artboards |
| `export` | Export nodes to image or video with full format and scale control. **Use this for production App Store assets.** |

### `paper_export` interface

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `nodes` | union | Yes | `"nodes-with-exports-only"` OR `{ nodeId: [{ format, scale, durationSeconds }] }` |
| `type` | enum | No | `"image"` (default) or `"video"` |

**Per-node override properties:**

| Property | Type | Description |
|----------|------|-------------|
| `format` | enum | `"avif"`, `"jpg"`, `"mp4"`, `"png"`, `"svg"`, `"webp"` |
| `scale` | string | `"0.5x"`, `"1x"`, `"2x"`, `"720p"`, `"1440p"`, `"512w"`, `"512h"` |
| `durationSeconds` | number | Video duration in seconds (1-300, default 10). Only for `type: "video"` |

**Example usage:**
- Export all pre-configured exports: `{ nodes: "nodes-with-exports-only", type: "image" }`
- Export specific nodes with overrides: `{ nodes: { "node-123": [{ format: "png", scale: "2x" }] }, type: "image" }`
- Export video: `{ nodes: { "node-456": [{ format: "mp4", scale: "1x", durationSeconds: 10 }] }, type: "video" }`

**Export vs screenshot:** `get_screenshot` is low-res (1x/2x) and for previews only. `paper_export` produces production-quality images at arbitrary scales and formats. Always use `export` for final App Store assets.

## Workflow

Agent-driven, stage-gated pipeline. State is implicit (tracked by artboard names in Paper). Human decision gates at Review stages.

```
┌──────────────┐   ┌──────────────┐   ┌──────────────┐   ┌──────────────┐
│  DISCOVERY   │ → │  STRATEGY    │ → │  DESIGN      │ → │  REVIEW      │
│ (read Paper) │   │ (plan shots) │   │ (iPhone)     │   │ (human gate) │
└──────────────┘   └──────────────┘   └──────────────┘   └──────────────┘
                                                                │
┌──────────────┐   ┌──────────────┐   ┌──────────────┐        │
│   EXPORT     │ ← │  REFACTOR    │ ← │   DONE?      │ ←───────┘
│ (MCP)        │   │ (iPad)       │   │ (human gate) │
└────┬─────────┘   └──────────────┘   └──────────────┘
     │
     ▼ (optional)
┌──────────────┐
│   VARIANTS   │ → Done
│  (copy swap) │
└──────────────┘
```
┌─────────────┐   ┌─────────────┐   ┌─────────────┐   ┌─────────────┐
│  DISCOVERY  │ → │  STRATEGY   │ → │   DESIGN    │ → │   REVIEW    │
│ (MCP read)  │   │ (plan)      │   │ (iphone)    │   │ (human/ai)  │
└─────────────┘   └─────────────┘   └─────────────┘   └─────────────┘
                                                              │
┌─────────────┐   ┌─────────────┐   ┌─────────────┐         │
│   EXPORT    │ ← │  REFACTOR   │ ← │   REVIEW    │ ←─────────┘
│ (MCP)       │   │  (ipad)     │   │  (approve)  │
└──────┬──────┘   └─────────────┘   └─────────────┘
       │
       ▼
┌─────────────┐   ┌─────────────┐
│   VARIANT   │ → │   DELIVER   │
│   (a/b/c)   │   │  (assets)   │
└─────────────┘   └─────────────┘
```

## Stages

### Stage 1: Discovery
The agent calls `paper_get_basic_info` to read:
- Existing artboards and their names/dimensions
- Page structure
- File name and current zoom

If a `config.yaml` exists, it also reads that. Otherwise, the agent infers features from artboard names and context clues.

**Agent responsibility:** Summarize the app, list detected features, propose 3-5 screenshot angles.

**Prompt guidance:** Use [references/discovery.txt](references/discovery.txt) to audit the Paper project, scan the codebase, and generate `dist/discovery.json` with app metadata and brand tokens.

### Stage 2: Strategy (Human Gate)
Agent proposes a screenshot sequence:
- **Headline** for each shot (e.g. "Privacy. Always.")
- **Subheading** (supporting message)
- **Feature** being showcased
- **Visual angle** (technical, emotional, benefit-focused, etc.)

**Human decision:** Approve, edit, or skip to next stage.

**Prompt guidance:** Use [references/copywriter.txt](references/copywriter.txt) to craft headlines and subheads per feature, and write `dist/strategy.yaml` with the approved sequence definitions.

### Stage 3: Design (iPhone)
Agent creates **428×926px** artboards in Paper (one per screenshot) named `screen_01`, `screen_02`, etc. using:
- `paper_create_artboard` — new marketing canvas
- `paper_write_html` — layout structure (typography, spacing, background)
- `paper_set_text_content` + `paper_update_styles` — headlines, body, CTAs
- `paper_get_screenshot` — 1x or 2x preview (for your eyes only, not for export)

**Critical design rule:** These are **marketing images, not UI**. Think billboards, not buttons:
- Font sizes 2-3× typical UI (headlines 48-72px, body 24-36px)
- HUGE whitespace. Dramatic padding.
- Bold color, high contrast
- Minimal text (max 2-3 lines per shot)
- Single feature per slide

Agent will ask for your feedback after drafts.

**Prompt guidance:** Use [references/designer_iphone.txt](references/designer_iphone.txt) to build artboards with proper typography, spacing, and brand consistency. Reference `dist/discovery.json` for brand tokens and `dist/strategy.yaml` for copy.

### Stage 4: Review (Human Gate)
You review the iPhone artboards directly in Paper. You can:
- **Approve** — proceed to iPad refactor
- **Refine** — request tweaks (text, color, spacing, imagery)
- **Reject** — start Stage 3 over with new copy angles

Agent will refine iteratively if you request changes.

**Agent responsibility (feedback loops):** When user provides feedback:
1. **Acknowledge current position:** "We're in Stage 4: iPhone Review. I'll incorporate your feedback..."
2. **Apply changes**
3. **State next action explicitly:** "Once you approve these changes, we'll move to Stage 5: iPad refactor (688×917 dimensions)."

### Stage 5: Refactor (iPad)
Once iPhone is approved, agent duplicates artboards to **688×917px** (iPad 13" dimensions) using:
- `paper_duplicate_nodes` — clone all `screen_XX` artboards into new `ipad_XX` artboards
- `paper_update_styles` — resize text, adjust padding to fit new dimensions
- `paper_move_nodes` — organize on canvas

**Critical:** iPad is wider than iPhone, but not always more spacious for text. A side-by-side layout (visual + text) often produces narrower text columns than the original iPhone layout. Agent must:
1. Calculate final text container width *before* choosing headline size
2. Use explicit line breaks for display type (56px+) to avoid widows and orphans
3. Reduce headline size (52–64px) in side-by-side layouts unless the text column exceeds 330px
4. Validate each headline wrapping before export (no lines under 4 characters, no weak orphans)

Agent will flag any text overflow or major spacing issues for you to manually tweak.

**Prompt guidance:** Use [references/designer_ipad.txt](references/designer_ipad.txt) to adapt iPhone layouts with container-aware typography. Calculate text column width *after* choosing layout, not before. Use the layout decision matrix and text-fit calculation to avoid orphaned words and awkward wrapping.

### Stage 6: Export
Agent calls `paper_export` to generate production PNGs via MCP:

```json
{
  "nodes": {
    "screen_01": [{ "format": "png", "scale": "3x" }],
    "screen_02": [{ "format": "png", "scale": "3x" }],
    "ipad_01": [{ "format": "png", "scale": "3x" }],
    "ipad_02": [{ "format": "png", "scale": "3x" }]
  },
  "type": "image"
}
```

Files export to Paper's default location. Agent will also post-process iPad PNGs with ffmpeg to exact App Store dimensions:
```bash
ffmpeg -i ipad_export.png -vf "scale=2065:2751" -pix_fmt rgb24 ipad_export_final.png
```

**Final assets:**
- iPhone: 1284×2778px @ 3x (6.5" variant)
- iPad: 2065×2751px (13" variant)

### Stage 7: Variants (Optional)
If you want A/B testing, agent duplicates approved artboards (`variant_a_01`, `variant_b_01`, etc.) and swaps headlines/angles using `paper_set_text_content`. Then re-exports all variants.

**Prompt guidance:** Use [references/variant.txt](references/variant.txt) to create Variant B and C copies with different marketing angles (e.g., professional, casual, security-focused), keeping layouts identical and only changing copy/mood colors.

## Usage

Simply invoke the skill and tell the agent what app you want screenshots for. Example:

> Load the app-store-screenshots skill and generate 4 App Store screenshots for my iOS app "Copyboard" (custom keyboard, copy pre-saved snippets).

The agent will:
1. Call `paper_get_basic_info` to understand your Paper project
2. Propose a strategy (you approve/edit)
3. Design iPhone artboards (you review)
4. (If approved) Refactor for iPad
5. Export PNGs at 3x scale
6. (Optional) Generate variants

**Resumable:** If interrupted, just ask the agent to continue from where it left off. State is implicit in Paper artboard names.

## Configuration (Optional)

Create a `config.yaml` to skip some agent inference. If you don't have one, use [references/infer_config.txt](references/infer_config.txt) to auto-generate it from your app codebase, App Store listing, or design system.

```yaml
app:
  name: "MyApp"
  tagline: "Two-sentence description of what it does"
  
screenshots:
  sequence:
    - feature: "Core feature 1"
      angle: "emotional angle"
    - feature: "Core feature 2"
      angle: "benefit angle"
    - feature: "Unique differentiator"
      angle: "fear/desire angle"
    
variants:
  enabled: false  # set to true to auto-generate A/B variants
  angles: ["professional", "casual"]
```

If no config exists, the agent infers everything and asks for approval.

## Key Principles

1. **Verify MCP first** — Call `paper_get_basic_info` immediately. If it fails, stop and guide user through MCP setup.
2. **State is implicit** — No config files needed. Artboard names (`screen_01`, `ipad_01`, etc.) track progress.
3. **iPhone first, iPad second** — Never try to design both at once. Complete iPhone review before iPad refactor.
4. **Marketing ≠ UI** — Force large typography (48-72px headlines), massive whitespace, bold colors. Not buttons and menus.
5. **3x export feels right** — Artboards at 428×926 with 3x export = 1284×2778 matches real iPhone 6.5" closely. Agent gets better spatial intuition.
6. **Human gates matter** — Stage 2 (Strategy), Stage 4 (Review), and Stage 6 (Variants) are all "pause and ask" moments. Don't auto-proceed.
7. **Delete before regenerating** — If user wants new screenshots, delete the old `screen_XX` and `ipad_XX` artboards first.
8. **`paper_export` is production** — Use it for final assets. `paper_get_screenshot` is preview-only; never export with that.

## Feedback Loop Guidance for Agents

When users provide feedback at any Review stage, **always preserve workflow context** by:

1. **State the current stage:** "Stage 4: iPhone Review" or "Stage 5: iPad Refactor"
2. **Summarize the feedback:** "I'll update the headline on `screen_02` from 'X' to 'Y', increase body text to 32px, and adjust padding..."
3. **Make the changes** in Paper
4. **State the immediate next action:** 
   - If at Stage 4 (iPhone Review) and user approves after feedback: "Next: We'll move to Stage 5 (iPad refactor). I'll duplicate these to 688×917px."
   - If at Stage 5 (iPad Refactor) and user approves: "Next: We'll export all iPhone and iPad artboards at 3x scale to production PNGs."
   - If at Stage 7 (Variants) and user approves: "Next: All variants are complete. Your final assets are ready."

**This prevents users from losing sight of where they are in the workflow.** Each feedback loop should end with a clear forward-looking statement about what comes next.

## Troubleshooting

| Problem | Fix |
|---------|-----|
| Paper MCP not accessible | Verify Paper Desktop is running. Check your harness config in Prerequisites. Try `paper_get_basic_info` directly. |
| `paper_get_basic_info` fails with "Session not found" | This is normal (MCP is running). Agent can proceed. |
| Agent designs tiny typography | Add to your prompt: "These are billboard-sized marketing shots. Make headlines 48-72px, body 24-36px." |
| iPad refactor has text overflow | Expected. Manually adjust in Paper after the agent creates it, or ask agent to re-run with smaller font. |
| `paper_export` succeeds but files are empty | Verify the node IDs match actual artboard names in Paper. Check format/scale are valid enums. |
| ffmpeg post-process fails | Install ffmpeg: `apt-get install ffmpeg` (Linux) or `brew install ffmpeg` (Mac). |
| App Store Connect rejects PNGs | Ensure `-pix_fmt rgb24` in ffmpeg command. iPad PNGs must be exactly 2065×2751. iPhone exactly 1284×2778. |
| Agent gets confused by old artboards | Delete the old `screen_XX` and `ipad_XX` artboards before starting new generation. Agent sees all artboards and may mix them up. |

## Agent Prompting Reference

When invoking the agent at each stage, use these framing keywords. **Always load the relevant reference file to provide detailed guidance:**

- **Discovery:** Load [references/discovery.txt](references/discovery.txt). "Read the Paper project using `paper_get_basic_info`. Audit the codebase and generate `dist/discovery.json` with app metadata and brand tokens."
- **Strategy:** Load [references/copywriter.txt](references/copywriter.txt). "Craft headlines and subheads for each feature angle. Write `dist/strategy.yaml` with the approved sequence definitions."
- **Design:** Load [references/designer_iphone.txt](references/designer_iphone.txt). "Create iPhone artboards at 428×926 for each approved angle. These are MARKETING images, not UI. Go 2-3× larger than typical UI font sizes."
- **Refactor:** Load [references/designer_ipad.txt](references/designer_ipad.txt). "Duplicate the approved `screen_XX` artboards into `ipad_XX` at 688×917. Reflow text/spacing for the wider canvas, maintaining hierarchy."
- **Export:** "Export all `screen_XX` and `ipad_XX` artboards to PNG at 3x scale using `paper_export`. Then post-process iPad PNGs with ffmpeg to 2065×2751."
- **Variants:** Load [references/variant.txt](references/variant.txt). "Create Variant B and C with different marketing angles (professional, casual, security-focused), keeping layouts identical."