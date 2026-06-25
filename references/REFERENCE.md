# App Store Screenshots Reference Guide

Specialized prompts for each stage of the screenshot pipeline. Load these when activating the corresponding workflow stage.

## Index

### [discovery.txt](discovery.txt)
**Stage 1: Discovery** — Analyze the app, codebase, and design system to extract metadata.

- Audits the Paper project via MCP
- Scans codebase for app metadata (name, features, tagline)
- Extracts brand tokens (colors, typography, spacing)
- Outputs: `dist/discovery.json`

### [copywriter.txt](copywriter.txt)
**Stage 2: Strategy (Human Gate)** — Craft marketing angles and copy for each screenshot.

- Guidance on storytelling across a screenshot sequence
- Emotional angle strategies (fear, aspiration, curiosity, simplicity, etc.)
- Rules for headlines (3–6 words), subheads (8–12 words)
- Outputs: `dist/strategy.yaml` with approved sequence

### [designer_iphone.txt](designer_iphone.txt)
**Stage 3: Design (iPhone)** — Create marketing artboards for iPhone (428×926px).

- Typography spec: headlines 56–72px, subheads 28–36px, body 20–24px
- Layout principles: one idea per artboard, generous whitespace, high contrast
- MCP workflow: create → write HTML → set text → preview
- Naming convention: `screen_01_hero`, `screen_02_feature`, etc.

### [designer_ipad.txt](designer_ipad.txt)
**Stage 5: Refactor (iPad)** — Adapt approved iPhone designs for iPad (688×917px).

- Reflow strategy: don't stretch, recompose for wider canvas
- Layout options: centered text, side-by-side with device frame
- MCP workflow: duplicate → rename → resize → reflow
- Post-processing: ffmpeg scaling to 2064×2752 exact compliance

### [infer_config.txt](infer_config.txt)
**Setup: Configuration Inference** — Auto-generate `config.yaml` from multiple sources.

- Scan sources: iOS/Android codebase, App Store listing, Notion docs, landing pages
- Extract: app name, bundle ID, feature list, brand colors, typography
- Infer optimal marketing angles based on feature descriptions
- Outputs: `config.yaml` ready for stages 1–7

### [variant.txt](variant.txt)
**Stage 7: Variants (Optional)** — Create A/B/C test variants with different marketing angles.

- Variant strategies: professional, casual, security-focused, aspirational, fear-based
- MCP workflow: duplicate → rename → swap copy → adjust mood colors
- Constraint: keep layouts identical, only change text and minor colors
- Integration: exports all variants alongside control set
