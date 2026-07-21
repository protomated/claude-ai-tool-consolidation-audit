# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

**PAC-74 (CP13)** — Billing Narrative & Time-Entry Drafting. A Claude Desktop plugin for solo and small-firm attorneys. One skill (`/billing-narrative`) takes rough time-entry notes, an email thread, or calendar event details and drafts a clean, billing-code-appropriate narrative with a suggested time increment. The attorney reviews and pastes the final entry into Clio, MyCase, PracticePanther, or the Legal Billing Tracker. There is no runtime code, no MCP server, no connector, and no backend. The product is entirely content: a markdown skill file, JSON manifests, and a reference doc.

Landing page: `protomated.com/templates/billing-narrative-drafter/` (WordPress — managed outside this repo).

## Repo layout

```
plugin/           The installable plugin (packaged into .zip bundle)
  .claude-plugin/plugin.json   Manifest validated by scripts/validate-plugin.mjs
  .mcp.json                    Empty — this plugin requires no connectors
  manifest.json                Plugin display metadata
  prompts/system-prompt.md     Master system prompt — compliance guardrails live here
  skills/billing-narrative/SKILL.md  The single skill; YAML frontmatter + markdown body
scripts/
  validate-plugin.mjs          Validates plugin/ structure before packing
docs/
  NTC-A-1.md, PAC-A-3.md      Engineer onboarding reference docs
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

The bundle format is `.zip`. It uses the **plugin variant** (not standalone) — no bundled MCP server, no connectors. Plugin name: `billing-narrative-drafter`, current version: `1.0.0`.

Two manifests serve different purposes:
- `plugin/.claude-plugin/plugin.json` — the identity manifest the validator and Claude Desktop read (`name` must be kebab-case)
- `plugin/manifest.json` — display metadata only (no `server` block — this is a plugin variant, not standalone)

The validator (`scripts/validate-plugin.mjs`) checks:
- `.claude-plugin/plugin.json` is valid JSON with a kebab-case `name`
- Each `skills/*/` subdirectory contains a `SKILL.md`
- `agents/`, `commands/`, `hooks/` (if present) contain files with the expected extension

## Skill: /billing-narrative

The single skill takes rough time-entry notes (shorthand, email threads, calendar event descriptions) and:
1. Clarifies ambiguity before drafting — asks about activity type, whether to split bundled activities, billing increment style, and UTBMS code preference. One question at a time.
2. Drafts a professional billing narrative in active past tense, specific to the activity, without inventing facts not in the notes.
3. Suggests a time increment rounded to the attorney's billing style (0.1 hr or 0.25 hr), flagging estimates.
4. Presents the draft with an explicit review invitation — the attorney confirms accuracy before the entry is marked ready to paste.
5. Iterates on revisions as many times as needed; restates the final narrative cleanly when confirmed.

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

1. **Attorney review gate**: Claude must present every narrative draft and invite confirmation before marking the entry ready to paste. It never declares a draft final unilaterally.
2. **Required output wrapper**: Every skill output must begin with the "ASSISTED DRAFT — ATTORNEY REVIEW REQUIRED" header and end with the "Verify before billing | Not legal advice" footer (see `prompts/system-prompt.md` for exact text).
3. **Plan-tier warning**: The system prompt must warn that consumer-tier Claude (claude.ai Personal / Pro) must not be used to enter confidential matter information.
4. **No facts invented**: The skill must never add facts not present in the attorney's notes. If notes are too sparse to draft accurately, it asks before proceeding.
5. **Ambiguity resolution**: The skill must ask before drafting if the activity type, multi-entry split, or time basis is unclear. It must never guess.
6. **No external actions**: The skill never submits, records, or transmits entries to any billing system. The attorney pastes the final narrative manually.

## Commit style

Do not include `Co-Authored-By` attribution lines in commit messages.

## Canonical plugin description

Used in `plugin/.claude-plugin/plugin.json` and any marketing copy — keep consistent:

> A billing narrative drafter that converts rough notes into professional time-entry language, suggests a time increment, and prompts attorney review before billing — for solo and small-firm attorneys capturing time in Clio, MyCase, PracticePanther, or any billing system.

## Testing

Testing is manual inside Claude Desktop — there is no test runner. The `plugin/README.md` is the canonical testing guide. It contains:
- Setup steps (build → install → verify skill loads)
- 10 specific test inputs with exact text to paste and what to check for each

Key scenarios that must pass: basic conference call narrative, email exchange, document drafting, court appearance, research session, ambiguous notes (skill asks, does not guess), UTBMS codes on request, multi-activity split prompt, edit-and-revise loop, confirmation gate (no entry marked ready until attorney confirms).

## Notes

- `plugin/.mcp.json` is `{}` — this plugin requires no connectors. Do not add a connector unless the skill explicitly needs one.
- `plugin/manifest.json` has no `server` block — the plugin variant does not require one. Do not add one.
- `plugin/README.md` and `plugin/CONNECTORS.md` are end-user documentation included in the ZIP bundle; they are not internal developer docs.
- The root `.mcp.json` is gitignored — it holds workspace-level Claude Code MCP credentials and is not part of the plugin artifact.
- `npm run release` passes `--notes-file RELEASE.md` to `gh release create` — create/update `RELEASE.md` at repo root before running it.
- Package scripts use `$npm_package_name` and `$npm_package_version` — keep the `name` field in `package.json` in sync with the plugin slug.
