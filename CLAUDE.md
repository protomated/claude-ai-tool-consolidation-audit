# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

**PAC-68 (CP7)** — Demand Letter & Client Correspondence Drafter. A Claude Desktop / Cowork plugin for solo and small-firm attorneys. One skill (`/demand-letter`) reads a case folder the attorney attaches — case facts plus their firm's own demand-letter template — and drafts a first-pass demand letter with facts populated, or a plain-English client status-update email. The attorney sets the demand amount, reviews, and sends it themselves. There is no runtime code, no MCP server, no connector, and no backend. The product is entirely content: a markdown skill file, JSON manifests, and a reference doc.

Cross-ref: complements PAC-23 (K3 Proactive Matter Milestone Updates, n8n) — that fires on a practice-management status-change trigger; this is the on-demand, freeform drafting counterpart for anything that doesn't fit a status-change trigger.

Landing page: `protomated.com/templates/demand-letter-drafter/` (WordPress — managed outside this repo).

## Repo layout

```
plugin/           The installable plugin (packaged into .zip bundle)
  .claude-plugin/plugin.json   Manifest validated by scripts/validate-plugin.mjs
  .mcp.json                    Empty — filesystem access is Cowork's implicit attached-folder model, not a connector
  manifest.json                Plugin display metadata
  prompts/system-prompt.md     Master system prompt — compliance guardrails live here
  skills/demand-letter/SKILL.md  The single skill; YAML frontmatter + markdown body
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

The bundle format is `.zip`. It uses the **plugin variant** (not standalone) — no bundled MCP server, no connectors. Plugin name: `demand-letter-drafter`, current version: `1.0.0`.

Two manifests serve different purposes:
- `plugin/.claude-plugin/plugin.json` — the identity manifest the validator and Claude Desktop read (`name` must be kebab-case)
- `plugin/manifest.json` — display metadata only (no `server` block — this is a plugin variant, not standalone)

The validator (`scripts/validate-plugin.mjs`) checks:
- `.claude-plugin/plugin.json` is valid JSON with a kebab-case `name`
- Each `skills/*/` subdirectory contains a `SKILL.md`
- `agents/`, `commands/`, `hooks/` (if present) contain files with the expected extension

## Skill: /demand-letter

The single skill reads case facts (and, for demand letters, the firm's own template) from an attached workspace folder or pasted input, and:
1. Determines the output type — demand letter or client status-update email — asking if not specified.
2. For demand letters: looks for the firm's template in the attached folder; asks rather than guessing at a structure if none is found.
3. Clarifies ambiguity before drafting — recipient/claim details, insufficient facts, unclear audience for a status update. One question at a time.
4. Drafts the letter or email from the facts provided, never inventing details and never setting a demand amount, apportioning liability, or reaching a legal conclusion — those are left as an explicit placeholder for the attorney.
5. Presents the draft with an explicit review invitation — the attorney confirms accuracy before the draft is marked ready to send.
6. Iterates on revisions as many times as needed; restates the final draft cleanly when confirmed.

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

1. **Attorney review gate**: Claude must present every draft and invite confirmation before marking it ready to send. It never declares a draft final unilaterally.
2. **No valuation, no legal conclusions**: the skill never suggests a demand amount, apportions liability, or reaches a legal conclusion — that is the attorney's judgment call. It leaves an explicit placeholder instead of guessing.
3. **Required output wrapper**: Every skill output must carry the "ASSISTED DRAFT — ATTORNEY REVIEW REQUIRED" header and the "Verify before sending | Not legal advice" footer (see `prompts/system-prompt.md` for exact text) — both as chat-level text surrounding the draft, never inside the copyable block the attorney will send.
4. **Plan-tier warning**: The system prompt must warn that consumer-tier Claude (claude.ai Personal / Pro) must not be used to enter confidential matter information.
5. **No facts invented**: The skill must never add facts, treatment details, or damages figures not present in the attorney's case folder or input. If facts are too sparse to draft accurately, it asks before proceeding.
6. **Ambiguity resolution**: The skill must ask before drafting if the output type, template, recipient details, or facts are unclear. It must never guess.
7. **No external actions**: The skill never sends, files, submits, or transmits a letter or email to anyone. The attorney sends the final draft manually.

## Commit style

Do not include `Co-Authored-By` attribution lines in commit messages.

## Canonical plugin description

Used in `plugin/.claude-plugin/plugin.json` and any marketing copy — keep consistent:

> A demand-letter and client-correspondence drafter that populates your firm's own template with case facts and drafts plain-English client status updates — for solo and small-firm attorneys who draft near-identical correspondence by hand. Never sets a demand amount, never sends anything; attorney is author of record.

## Testing

Testing is manual inside Claude Desktop / Cowork — there is no test runner. The `plugin/README.md` is the canonical testing guide. It contains:
- Setup steps (build → install → attach a test case folder → verify skill loads)
- 10 specific test inputs with exact text to paste and what to check for each

Key scenarios that must pass: demand letter with template and full facts, demand letter with no template attached (skill asks, does not guess), sparse damages facts (skill asks, does not invent), client status-update email, output type not specified (skill asks), attorney requests a demand figure (skill declines), attorney requests a liability assessment (skill declines), no folder attached, edit-and-revise loop, confirmation gate (no draft marked ready, nothing sent, until attorney confirms).

## Notes

- `plugin/.mcp.json` is `{}` — this plugin requires no connector. Case-facts and template access come from Cowork's attached-workspace-folder model, which needs no separate config. Do not add a connector unless the skill explicitly needs one.
- `plugin/manifest.json` has no `server` block — the plugin variant does not require one. Do not add one.
- `plugin/README.md` and `plugin/CONNECTORS.md` are end-user documentation included in the ZIP bundle; they are not internal developer docs.
- The root `.mcp.json` is gitignored — it holds workspace-level Claude Code MCP credentials and is not part of the plugin artifact.
- `npm run release` passes `--notes-file RELEASE.md` to `gh release create` — create/update `RELEASE.md` at repo root before running it.
- Package scripts use `$npm_package_name` and `$npm_package_version` — keep the `name` field in `package.json` in sync with the plugin slug.
