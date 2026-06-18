# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

This is a **Claude Code Skills Library** for Pika Labs — a collection of 16 AI-powered content creation skills installed under `.claude/skills/`. Each skill is a self-contained multi-stage workflow that orchestrates Pika MCP tools to generate video, images, and audio assets.

This is **not a traditional software project**: there is no build step, no test suite, and no linter. Skills are static Markdown definitions invoked at runtime by Claude Code.

## Invoking Skills

Skills are triggered by `/skillname` commands in Claude Code. Each `SKILL.md` defines its argument-hint in frontmatter:

```
/app-sizzle "App Name" screens=<url>
/persona-builder @instagram_handle
/content-director @tiktok_handle talking
/build-a-brand "brand concept" --quick
/founder-product-video https://product.com --founder "Jane Doe"
/ugc-ads product=<url>
```

## Skills Manifest

All 16 skills are tracked in `skills-lock.json` (source: `Pika-Labs/Pika-Plugins`, branch: `staging`). This file records content hashes for integrity verification. When updating or adding skills, keep this file in sync.

## Architecture

### Skill Structure

Each skill lives at `.claude/skills/<name>/SKILL.md` (mirrored in `.agents/skills/<name>/SKILL.md`). Complex skills include a `references/` or `formats/` subdirectory with templates, layout specs, and workflow guides.

### Runtime Pipeline Pattern

Every skill follows this pattern:
1. **Asset sourcing** — scrape social profiles, fetch URLs, accept user uploads
2. **Analysis** — `analyze_media`, `scrape_social` to extract context
3. **Cost gate** — display estimated cost, require explicit user approval before paid calls
4. **Generation** — call MCP tools (Seedance primary, Kling fallback for video; GPT-image-2 for images)
5. **Post-processing** — captions, dubbing, harmony, overlays
6. **QA** — `analyze_media` post-flight checks, moderation recovery

### Key MCP Tool Categories

| Category | Primary Tools |
|---|---|
| Video generation | `generate_reference_video`, `generate_viral_hook` |
| Image generation | `generate_image`, `render_html` |
| Audio | `generate_harmony`, `dub_video`, `edit_audio_autotune` |
| Avatar/lipsync | `generate_lipsync`, `edit_video` |
| Social scraping | `scrape_social` |
| Export | `html_to_pdf`, `add_captions`, `task_status` |

### Cross-Skill Dependencies

- `persona-builder` → outputs `persona.md` consumed by `ugc-ads`, `podcast`, `founder-product-video`, `app-sizzle`, `app-store-screens`
- `build-a-brand` → outputs `brand.md` + kit consumed by `app-store-screens`, `founder-product-video`, and other visual skills
- `content-director` → routes into 4 format playbooks in `.claude/skills/content-director/formats/` (talking, pov, dance, duet)

### Resilience Patterns

Skills embed these runtime guarantees:
- **Fallback providers**: Seedance → Kling when primary fails
- **Moderation recovery**: retry loop for audio false positives
- **Time guards**: 5-min pre-generation warning, 15-min hard ceiling per task
- **Long-running polling**: async task status checks via `task_status`

## Updating Skills

To update a skill, edit its `SKILL.md` (and any files in its `references/` or `formats/` subdirectory), then update the corresponding entry in `skills-lock.json` with the new content hash. Changes must be mirrored in both `.claude/skills/` and `.agents/skills/`.
