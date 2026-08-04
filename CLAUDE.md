# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

**CP18** — AI Tool Consolidation & Data-Hygiene Audit Skill. A Claude Desktop / Cowork plugin for solo and small-firm attorneys. One skill (`/ai-tool-audit`) runs a short guided interview on which AI tools the firm uses, for what, and with what data, then produces a data-hygiene audit — flagging data-handling risks and redundant tools, and recommending genuine consolidation candidates onto a governed Claude + MCP stack, mapped to the firm's actual workflows. Specialized tools (practice management, docketing/deadline engines, e-discovery, e-signature) are named to keep just as plainly as tools recommended for consolidation — the skill never pitches a blanket "replace everything with Claude." The attorney (or whoever ran the interview) reviews and confirms the audit before treating it as the firm's current inventory. There is no runtime code, no MCP server, no connector, and no backend. The product is entirely content: a markdown skill file, a rating-rubric reference doc, and JSON manifests.

The leak it plugs: firms are accumulating an average of about 18 different AI tools with inconsistent data policies and no single source of truth — a firm rarely has one place that says what it's using, what data touches what, and what's simply duplicated effort. Nothing dedicated addresses this; it's an original angle, not a shrink of an existing tool category. It also positions the wider plugin catalog as "one governed AI system" rather than one more point tool added to the pile — pairs conceptually with the AI Use Policy & Client-Disclosure Generator (CP3) skill in outreach, even though the two ship as separate plugins: this skill audits and recommends a stack, CP3 drafts the actual policy and disclosure clause. If asked to draft a policy itself, this skill declines and states that's outside what it produces — it does not name CP3 by name at runtime, since this skill must not imply a specific sibling product is published and available before it actually is.

Landing page: `protomated.com/templates/ai-tool-consolidation-audit/` (WordPress — managed outside this repo).

## Repo layout

```
plugin/           The installable plugin (packaged into .zip bundle)
  .claude-plugin/plugin.json   Manifest validated by scripts/validate-plugin.mjs
  .mcp.json                    Empty — filesystem access is Cowork's implicit attached-folder model, not a connector
  manifest.json                Plugin display metadata
  prompts/system-prompt.md     Master system prompt — compliance guardrails live here
  skills/ai-tool-audit/
    SKILL.md                   The single skill; YAML frontmatter + markdown body
    reference/audit-rubric.md  Data-handling rating scale, inventory row format, and interview categories
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

The bundle format is `.zip`. It uses the **plugin variant** (not standalone) — no bundled MCP server, no connectors. Plugin name: `ai-tool-consolidation-audit`, current version: `1.0.0`.

Two manifests serve different purposes:
- `plugin/.claude-plugin/plugin.json` — the identity manifest the validator and Claude Desktop read (`name` must be kebab-case)
- `plugin/manifest.json` — display metadata only (no `server` block — this is a plugin variant, not standalone)

The validator (`scripts/validate-plugin.mjs`) checks:
- `.claude-plugin/plugin.json` is valid JSON with a kebab-case `name`
- Each `skills/*/` subdirectory contains a `SKILL.md`
- `agents/`, `commands/`, `hooks/` (if present) contain files with the expected extension

## Skill: /ai-tool-audit

The single skill runs a short guided interview on the AI tools the firm currently uses, optionally starting from an existing AI-tools list attached to a workspace folder or pasted input, and:
1. Confirms an attached inventory is complete before treating it as the full list, rather than assuming it is; asks from scratch if there's no existing list.
2. For each tool, collects what it's used for, who uses it, what data it touches, and its data-handling status as the firm currently understands it — never inferring any of these from the tool's name or category alone.
3. Rates each tool's data-handling status: 🟢 confirmed appropriate, 🟡 partial/mixed, 🔴 confirmed risk, or ⚪ UNCONFIRMED (the firm doesn't know the tool's current terms) — never guessing a status, and never asserting what a named vendor's terms actually are from its own training knowledge.
4. Flags redundant tools by workflow — two or more tools serving the same stated task — without assuming overlap automatically means one must be eliminated.
5. Recommends genuine consolidation candidates onto a governed Claude + MCP stack, each tied to the firm's specific stated workflow and the specific gap found; names tools to keep as specialized (practice management, docketing/deadline engines, e-discovery, e-signature) just as explicitly as tools recommended for consolidation — never a blanket "replace everything."
6. Never invents an AI-tool inventory, a use case, or a data-handling status the firm didn't report.
7. Never certifies compliance with any bar rule, ethics opinion, or security standard; never performs or claims a security assessment; never drafts the firm's actual AI-use policy or client-disclosure clause (that's CP3's job, not this skill's) — those are declined or flagged instead of answered.
8. Never accesses, logs into, changes a setting on, migrates data from, or cancels anything at any vendor — the skill recommends, it does not act.
9. Presents the full audit with an explicit review invitation — whoever ran the interview confirms it before it's treated as the firm's current inventory.
10. Iterates on corrections and additions as many times as needed; restates the final findings cleanly when confirmed.

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

1. **Review gate**: Claude must present every audit and invite confirmation from whoever ran the interview before treating it as current. It never declares an audit final unilaterally.
2. **No legal or compliance judgment**: the skill never certifies compliance with any bar rule, ethics opinion, or security standard; never opines that current tool use breaches the firm's confidentiality duty; never performs or claims a security assessment (no penetration testing, no independent vendor verification). Findings are flagged for the attorney or ethics counsel to confirm, never resolved by the skill.
3. **Required output wrapper**: Every skill output must carry the "ASSISTED AI-TOOL AUDIT — ATTORNEY REVIEW REQUIRED BEFORE USE" header and the "Verify before use | Not legal advice" footer (see `prompts/system-prompt.md` for exact text) — both as chat-level text surrounding each audit, never inside an individual finding.
4. **Plan-tier warning**: The system prompt must warn that consumer-tier Claude (claude.ai Personal / Pro) must not be used for an interview touching the firm's actual AI-tool landscape, and that the interview should describe data categories rather than real client names or matter numbers.
5. **No facts invented**: The skill must never assert a named vendor's current data-handling terms from its own training knowledge. A tool's status is UNCONFIRMED when the firm doesn't know it, never guessed or asserted from general knowledge.
6. **Ambiguity resolution**: An attached AI-tools inventory must be confirmed complete before being treated as the full list, never assumed. A tool named without a stated use case is asked about before being rated. A firm reporting no AI tools is asked to reconsider common ones before the skill accepts that and stops, rather than inventing an inventory to audit.
7. **No external actions**: The skill never accesses, logs into, changes a setting on, migrates data from, or cancels anything at any vendor account, and never carries out its own consolidation recommendation. It produces chat text only — the firm, or a separate engagement, acts on it.
8. **Scope boundaries, declined not answered**: The skill never drafts the firm's actual AI-use policy or a client-facing AI-disclosure clause (a separate skill's job) and never recommends replacing a genuinely specialized tool — practice management, docketing/deadline engines, e-discovery, e-signature — just because it appears in the inventory alongside general-purpose AI tools. Every consolidation recommendation must cite the firm's own stated workflow and the specific gap found; every "keep" recommendation must name the tool and the reason.

## Internal QA fixtures — tests/skills/

`tests/skills/<skill-name>.md` is the internal QA testing guide for a skill — a standing convention for every plugin built in this repo, alongside (not replacing) the end-user testing guide in `plugin/README.md`. The difference:

- `plugin/README.md` — ships inside the plugin zip, short scenarios with pasted one-liners, aimed at an attorney verifying the install.
- `tests/skills/<skill-name>.md` — internal only, not packaged, uses real attached-folder fixtures under `tests/skills/<skill-name>/` for the attached-inventory path, plus scripted chat answers for the interview-only path. Deeper checks (e.g., compliance-wrapper placement, keep-vs-consolidate outcomes, UNCONFIRMED-status handling, out-of-scope requests) belong here even when they overlap with `plugin/README.md`'s scenarios.

All fixture data must be clearly synthetic — fictional names, firms, matter numbers. Never use real client or matter data, even anonymized real data, without checking with Dele first.

## Commit style

Do not include `Co-Authored-By` attribution lines in commit messages.

## Canonical plugin description

Used in `plugin/.claude-plugin/plugin.json` and any marketing copy — keep consistent:

> An AI-tool audit assistant that runs a short guided interview on which AI tools your firm uses, for what, and with what data — flagging data-handling risks and redundant tools, and recommending genuine consolidation candidates onto a governed Claude + MCP stack mapped to your firm's actual workflows, for solo and small-firm attorneys managing an average of 18 different AI tools with no single source of truth. Never certifies compliance with any bar rule or security standard, and never accesses, changes, or cancels anything at any vendor; attorney reviews and confirms the audit before treating it as current.

## Testing

Testing is manual inside Claude Desktop / Cowork — there is no test runner. The `plugin/README.md` is the canonical testing guide. It contains:
- Setup steps (build → install → optionally attach a test AI-tools inventory folder → verify skill loads)
- 10 specific test inputs with exact text to paste and what to check for each

Key scenarios that must pass: full audit with an existing inventory attached (skill confirms it's complete before proceeding), interview-only with nothing attached, a specialized tool present (skill recommends keeping it, not consolidating it), a confirmed data-handling risk (rated 🔴), an unconfirmed data-handling status (flagged ⚪, never guessed), redundant tools flagged by workflow, a firm reporting no AI tools (skill asks it to reconsider before accepting that), attorney asks for the firm's AI-use policy (skill declines, points to CP3), attorney asks the skill to act on its own recommendation (skill declines — audit and recommendation only), confirmation gate (no audit marked current, nothing at any vendor touched, until confirmed).

## Notes

- `plugin/.mcp.json` is `{}` — this plugin requires no connector. The interview runs in chat; an optional existing AI-tools inventory comes from Cowork's attached-workspace-folder model, which needs no separate config. Do not add a connector unless the skill explicitly needs one.
- `plugin/manifest.json` has no `server` block — the plugin variant does not require one. Do not add one.
- `plugin/README.md` and `plugin/CONNECTORS.md` are end-user documentation included in the ZIP bundle; they are not internal developer docs.
- The root `.mcp.json` is gitignored — it holds workspace-level Claude Code MCP credentials and is not part of the plugin artifact.
- `npm run release` passes `--notes-file RELEASE.md` to `gh release create` — create/update `RELEASE.md` at repo root before running it.
- Package scripts use `$npm_package_name` and `$npm_package_version` — keep the `name` field in `package.json` in sync with the plugin slug.
