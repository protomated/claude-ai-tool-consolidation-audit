# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

**PAC-69 (CP8)** — Court Deadline Reasoning & Calendar Drafting. A Claude Desktop plugin for solo and small-firm attorneys. One skill (`/court-deadline`) takes a trigger date and the applicable rule in plain English, computes the deadline step by step with an auditable reasoning chain, and offers to draft a calendar event via Google Calendar. There is no runtime code, no MCP server, and no backend. The product is entirely content: a markdown skill file, JSON manifests, and a reference doc.

Landing page: `protomated.com/templates/court-deadline-reasoning-skill/` (WordPress — managed outside this repo).

## Repo layout

```
plugin/           The installable plugin (packaged into .zip bundle)
  .claude-plugin/plugin.json   Manifest validated by scripts/validate-plugin.mjs
  .mcp.json                    Declares Google Calendar connector requirement (only)
  manifest.json                Plugin display metadata
  prompts/system-prompt.md     Master system prompt — compliance guardrails live here
  skills/court-deadline/SKILL.md  The single skill; YAML frontmatter + markdown body
scripts/
  validate-plugin.mjs          Validates plugin/ structure before packing
docs/
  Court Deadline Reasoning - Technical.md   Technical specification
  NTC-A-1.md, PAC-A-3.md                   Engineer onboarding reference docs
```

## Commands

All commands run from the repo root.

```bash
# Validate plugin structure (manifest, skill dirs, SKILL.md presence)
npm run validate

# Full build: validate → pack → SHA-256 → artifact
npm run build

# Pack only (skips validate)
npm run pack

# Cut a GitHub release (runs build first; requires RELEASE.md at repo root)
npm run release

# Remove build artifacts
npm run clean

# List plugin files (excludes node_modules)
npm run tree
```

## Plugin format

The bundle format is `.zip`. It uses the **plugin variant** (not standalone) — no bundled MCP server. Plugin name: `court-deadline-reasoning`, current version: `1.0.0`.

Two manifests serve different purposes:
- `plugin/.claude-plugin/plugin.json` — the identity manifest the validator and Claude Desktop read (`name` must be kebab-case)
- `plugin/manifest.json` — display metadata only (no `server` block — this is a plugin variant, not standalone)

The validator (`scripts/validate-plugin.mjs`) checks:
- `.claude-plugin/plugin.json` is valid JSON with a kebab-case `name`
- Each `skills/*/` subdirectory contains a `SKILL.md`
- `agents/`, `commands/`, `hooks/` (if present) contain files with the expected extension

## Skill: /court-deadline

The single skill takes a trigger date and the applicable deadline rule in plain English, then:
1. Resolves any ambiguities in the rule before computing (day type, counting anchor, rollover behavior, holiday scope)
2. Computes the deadline step by step with a full reasoning chain
3. Presents the reasoning chain as the primary output (auditable, not a footnote)
4. Offers to draft a Google Calendar event — shown in full before any creation, confirmed by the attorney before creation

It also handles federal holiday exclusions (built-in U.S. federal holiday calendar) and month/week arithmetic.

Each `SKILL.md` has YAML frontmatter:
```yaml
---
name: skill-name
description: shown to attorney in /skills list
argument-hint: "[hint shown in Claude Desktop]"
---
```

## Compliance constraints — non-negotiable

These rules are enforced in `prompts/system-prompt.md` and `SKILL.md`. Do not weaken them:

1. **Confirmation gating**: Claude must show the attorney exactly what calendar event it will create and get explicit in-conversation confirmation before creating any event.
2. **Required output wrapper**: Every skill output must begin and end with the prescribed "NOT A SUBSTITUTE FOR DOCKETING SOFTWARE" header/footer (see `prompts/system-prompt.md` for exact text).
3. **Plan-tier warning**: The system prompt must warn that consumer-tier Claude (claude.ai Personal / Pro) must not be used to enter confidential matter information.
4. **Hard compliance note**: All outputs must carry this note verbatim: "computes from the rule you provide; does not know your jurisdiction's rules; not a substitute for docketing software or your own verification." This is not optional.
5. **Ambiguity resolution**: The skill must ask before computing if the rule is ambiguous on day type, counting anchor, rollover, or holiday scope. It must never guess.

## Commit style

Do not include `Co-Authored-By` attribution lines in commit messages.

## Canonical plugin description

Used in `plugin/.claude-plugin/plugin.json` and any marketing copy — keep consistent:

> A step-by-step deadline calculator that reasons through the rule you supply, shows its work auditably, and drafts a calendar event — for solo and small-firm attorneys handling one-off or complex court date logic.

## Testing

Testing is manual inside Claude Desktop — there is no test runner. The root `README.md` is the canonical testing guide. It contains:
- Setup steps (build → install → connect Google Calendar → verify skill loads)
- 10 specific test inputs with exact text to paste and what to check for each
- Release build verification (`npm run build` + `sha256sum -c`)

Key scenarios that must pass: basic calendar-day count, weekend rollover, federal holiday rollover, business-day count, ambiguous-rule prompts (no guessing), month arithmetic edge cases, confirmation gate decline (no event created), confirmation gate confirm (event appears in Google Calendar), multiple deadlines from one rule.

## Notes

- `plugin/manifest.json` has no `server` block — the plugin variant does not require one. Do not add one.
- `plugin/README.md` and `plugin/CONNECTORS.md` are end-user documentation included in the ZIP bundle; they are not internal developer docs.
- The root `.mcp.json` is gitignored — it holds workspace-level Claude Code MCP credentials and is not part of the plugin artifact.
- `npm run release` passes `--notes-file RELEASE.md` to `gh release create` — create/update `RELEASE.md` at repo root before running it.
- The Google Calendar connector identifier in `plugin/.mcp.json` is `google-calendar`. Verify this against Claude Desktop's live Connectors panel if connector behavior changes.
