# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

**PAC-18 (CP6)** — Contract & Document Review Skill (Firm Playbook). A Claude Desktop / Cowork plugin for solo and small-firm attorneys. One skill (`/contract-review`) reads a contract the attorney attaches, plus, optionally, the firm's own configurable `playbook.md`, and reviews it clause by clause — flagging each clause GREEN, YELLOW, RED, or UNRATED with plain-English rationale and suggested redline language. The attorney reviews every rating and redline, applies any changes to their own document, and confirms before the review is used in negotiation. There is no runtime code, no MCP server, no connector, and no backend. The product is entirely content: a markdown skill file, a bundled generic playbook, a rating-rubric reference doc, and JSON manifests.

Cross-ref: net-new. The n8n catalog's cut list explicitly excluded "AI clause review/contract risk flagging" as legal analysis reserved for humans. This catalog's governance update (see PAC-61) puts it back in scope for Claude plugins specifically, because Claude reasons and drafts with a human always reviewing before anything is sent, unlike an autonomous n8n workflow.

Landing page: `protomated.com/templates/contract-document-reviewer/` (WordPress — managed outside this repo).

## Repo layout

```
plugin/           The installable plugin (packaged into .zip bundle)
  .claude-plugin/plugin.json   Manifest validated by scripts/validate-plugin.mjs
  .mcp.json                    Empty — filesystem access is Cowork's implicit attached-folder model, not a connector
  manifest.json                Plugin display metadata
  prompts/system-prompt.md     Master system prompt — compliance guardrails live here
  skills/contract-review/
    SKILL.md                   The single skill; YAML frontmatter + markdown body
    playbooks/generic-playbook.md   FREE-tier bundled generic clause playbook
    reference/review-rubric.md      GREEN/YELLOW/RED/UNRATED rating rubric and playbook-entry format
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

The bundle format is `.zip`. It uses the **plugin variant** (not standalone) — no bundled MCP server, no connectors. Plugin name: `contract-document-reviewer`, current version: `1.0.0`.

Two manifests serve different purposes:
- `plugin/.claude-plugin/plugin.json` — the identity manifest the validator and Claude Desktop read (`name` must be kebab-case)
- `plugin/manifest.json` — display metadata only (no `server` block — this is a plugin variant, not standalone)

The validator (`scripts/validate-plugin.mjs`) checks:
- `.claude-plugin/plugin.json` is valid JSON with a kebab-case `name`
- Each `skills/*/` subdirectory contains a `SKILL.md`
- `agents/`, `commands/`, `hooks/` (if present) contain files with the expected extension

## Skill: /contract-review

The single skill reads a contract the attorney attaches and, optionally, the firm's own `playbook.md` encoding its negotiation positions, from an attached workspace folder or pasted input, and:
1. Determines which contract to review, asking if more than one is attached and unspecified.
2. Uses the firm's own playbook if attached, or this plugin's bundled generic playbook if not — and says so plainly when the generic playbook is used.
3. Matches each clause in the contract to the applicable playbook entry, rating it GREEN (meets the playbook position), YELLOW (within an acceptable fallback range), RED (conflicts with a must-have or trips a red-flag trigger), or UNRATED (the playbook in use doesn't cover this clause type) — never guessing a rating to fill a coverage gap.
4. Suggests redline language for every clause that isn't GREEN, quoting the actual contract language and proposing a replacement — as chat text only, since the plugin cannot edit or generate a `.docx` file and never applies a Word tracked change.
5. Never invents contract language not present in the attached contract, and never invents a firm position not encoded in the playbook it's using.
6. Never decides whether the client should sign, walk away from, or accept the contract; never determines a clause's enforceability under governing law; never resolves choice-of-law or jurisdiction questions; never advises on privilege, confidentiality strategy, or tax consequences — those are the attorney's judgment calls, left as an explicit UNRATED flag or decline instead of a guess.
7. Presents the full review with an explicit review invitation — the attorney confirms it before it's used in negotiation.
8. Iterates on revisions as many times as needed; restates the final findings cleanly when confirmed.

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

1. **Attorney review gate**: Claude must present every review and invite confirmation before marking it ready. It never declares a review final unilaterally.
2. **No legal judgment**: the skill never decides whether to sign, walk away from, or accept a contract, never determines a clause's enforceability under governing law, never resolves a choice-of-law or jurisdiction question, and never advises on privilege, confidentiality strategy, or tax consequences — those are the attorney's judgment calls. A clause type the playbook in use doesn't cover is marked UNRATED, never guessed.
3. **Required output wrapper**: Every skill output must carry the "ASSISTED CONTRACT REVIEW — ATTORNEY REVIEW REQUIRED BEFORE USE" header and the "Verify before use | Not legal advice" footer (see `prompts/system-prompt.md` for exact text) — both as chat-level text surrounding each review, never inside a redline suggestion the attorney copies into their own document.
4. **Plan-tier warning**: The system prompt must warn that consumer-tier Claude (claude.ai Personal / Pro) must not be used to enter confidential client or contract information.
5. **No facts invented**: The skill must never invent contract language not present in the attached contract, or a firm position not encoded in the playbook it's using. A clause type not covered by the playbook is flagged UNRATED, never rated by guesswork.
6. **Ambiguity resolution**: The skill must ask which contract to review if more than one is attached and unspecified, and must mark a clause UNRATED (never silently assign a color) when the playbook in use doesn't address that clause type. Missing firm playbook is not an ambiguity to ask about — see rule 7.
7. **Playbook fallback, not a refusal**: If no firm playbook is attached, the skill uses its own bundled generic playbook and says so plainly — it does not stop and ask for a playbook first. The generic playbook must never be presented as this firm's actual negotiation positions.
8. **No external actions, no applied edits**: The skill never opens, edits, or generates a `.docx` file, never applies a Word tracked change, and never sends, files, executes, e-signs, or transmits the contract or the review to anyone. The attorney handles negotiation and execution manually.

## Internal QA fixtures — tests/skills/

`tests/skills/<skill-name>.md` is the internal QA testing guide for a skill — a standing convention for every plugin built in this repo, alongside (not replacing) the end-user testing guide in `plugin/README.md`. The difference:

- `plugin/README.md` — ships inside the plugin zip, short scenarios with pasted one-liners, aimed at an attorney verifying the install.
- `tests/skills/<skill-name>.md` — internal only, not packaged, uses real attached-folder fixtures under `tests/skills/<skill-name>/` when the skill's input is a workspace folder rather than chat text. Deeper checks (e.g., compliance-wrapper placement, playbook-vs-generic rating differences, coverage-gap edge cases) belong here even when they overlap with `plugin/README.md`'s scenarios.

All fixture data must be clearly synthetic — fictional names, firms, matter numbers. Never use real client or matter data, even anonymized real data, without checking with Dele first.

## Commit style

Do not include `Co-Authored-By` attribution lines in commit messages.

## Canonical plugin description

Used in `plugin/.claude-plugin/plugin.json` and any marketing copy — keep consistent:

> A contract and document review assistant that reviews a contract clause-by-clause against a configurable playbook — flagging each clause GREEN, YELLOW, or RED with plain-English rationale and suggested redline language — using your firm's own playbook, or a generic clause playbook if none is attached, for solo and small-firm attorneys who review contracts occasionally and can't justify a $99-400/mo dedicated tool. Never decides whether to sign, negotiate, or reject a contract, and never sends, files, or executes anything; attorney reviews and finalizes every position before use.

## Testing

Testing is manual inside Claude Desktop / Cowork — there is no test runner. The `plugin/README.md` is the canonical testing guide. It contains:
- Setup steps (build → install → attach a test contract folder → verify skill loads)
- 10 specific test inputs with exact text to paste and what to check for each

Key scenarios that must pass: full review with a firm playbook attached (ratings tied to the firm's own positions), no firm playbook attached (skill uses the bundled generic playbook and says so, does not ask first), a clause type not covered by the playbook (skill marks it UNRATED, does not guess), more than one contract attached (skill asks which one), attorney asks whether to sign (skill declines), attorney asks about enforceability (skill declines), no contract attached, edit-and-revise loop, confirmation gate (no review marked ready, nothing sent or filed, until attorney confirms), redline-is-not-an-applied-edit (skill confirms it cannot touch the `.docx` file directly).

## Notes

- `plugin/.mcp.json` is `{}` — this plugin requires no connector. Contract and playbook access come from Cowork's attached-workspace-folder model, which needs no separate config. Do not add a connector unless the skill explicitly needs one.
- `plugin/manifest.json` has no `server` block — the plugin variant does not require one. Do not add one.
- `plugin/README.md` and `plugin/CONNECTORS.md` are end-user documentation included in the ZIP bundle; they are not internal developer docs.
- The root `.mcp.json` is gitignored — it holds workspace-level Claude Code MCP credentials and is not part of the plugin artifact.
- `npm run release` passes `--notes-file RELEASE.md` to `gh release create` — create/update `RELEASE.md` at repo root before running it.
- Package scripts use `$npm_package_name` and `$npm_package_version` — keep the `name` field in `package.json` in sync with the plugin slug.
