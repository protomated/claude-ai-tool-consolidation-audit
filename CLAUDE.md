# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

**PAC-71 (CP10)** — Estate Planning Document Assembly Skill. A Claude Desktop / Cowork plugin for solo and small-firm estate planning attorneys. One skill (`/estate-documents`) reads a client's intake answers — family structure, assets, beneficiaries, healthcare wishes — plus, optionally, the firm's own state-specific templates, and populates a basic will, healthcare power of attorney, financial power of attorney, and HIPAA authorization consistently from one intake pass, flagging missing required fields per document type. The attorney verifies state execution formalities and finalizes each document before the client signs. There is no runtime code, no MCP server, no connector, and no backend. The product is entirely content: a markdown skill file, bundled placeholder templates, JSON manifests, and a reference doc.

Cross-ref: PAC-55 (Estate planning starter pack, n8n) — this Claude Skill becomes a component of that pack once built.

Landing page: `protomated.com/templates/estate-planning-document-assembler/` (WordPress — managed outside this repo).

## Repo layout

```
plugin/           The installable plugin (packaged into .zip bundle)
  .claude-plugin/plugin.json   Manifest validated by scripts/validate-plugin.mjs
  .mcp.json                    Empty — filesystem access is Cowork's implicit attached-folder model, not a connector
  manifest.json                Plugin display metadata
  prompts/system-prompt.md     Master system prompt — compliance guardrails live here
  skills/estate-documents/
    SKILL.md                   The single skill; YAML frontmatter + markdown body
    templates/                 FREE-tier generic placeholder templates (one per document type)
    reference/intake-checklist.md   Required/optional intake fields per document type
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

The bundle format is `.zip`. It uses the **plugin variant** (not standalone) — no bundled MCP server, no connectors. Plugin name: `estate-planning-document-assembler`, current version: `1.0.0`.

Two manifests serve different purposes:
- `plugin/.claude-plugin/plugin.json` — the identity manifest the validator and Claude Desktop read (`name` must be kebab-case)
- `plugin/manifest.json` — display metadata only (no `server` block — this is a plugin variant, not standalone)

The validator (`scripts/validate-plugin.mjs`) checks:
- `.claude-plugin/plugin.json` is valid JSON with a kebab-case `name`
- Each `skills/*/` subdirectory contains a `SKILL.md`
- `agents/`, `commands/`, `hooks/` (if present) contain files with the expected extension

## Skill: /estate-documents

The single skill reads intake answers (and, optionally, the firm's own state-specific templates) from an attached workspace folder or pasted input, and:
1. Determines which of the four document types to draft — will, healthcare POA, financial POA, HIPAA authorization, or all — asking if not specified.
2. For each document type: uses the firm's own template if attached, or this plugin's bundled generic placeholder template if not — and says so plainly when a placeholder is used.
3. Checks intake against the required-fields checklist per document type before drafting; drafts only the document types with complete fields, flags exactly what's missing for any that are blocked, and never lets one blocked document hold up the others.
4. Drafts each complete document from the intake provided, never inventing facts and never determining state execution requirements, resolving family/guardianship conflicts, advising on tax strategy, or deciding whether the client needs documents beyond these four — those are left as explicit placeholders or declines for the attorney.
5. Keeps names, agents, and dates consistent across every document drafted in the same session.
6. Presents the draft set with an explicit review invitation — the attorney confirms accuracy before the set is marked ready.
7. Iterates on revisions as many times as needed; restates the final set cleanly when confirmed.

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

1. **Attorney review gate**: Claude must present every draft and invite confirmation before marking a document set ready. It never declares a set final unilaterally.
2. **No legal judgment**: the skill never determines which documents a client needs, resolves a family/guardianship conflict, advises on tax strategy, assesses capacity/undue influence, or determines a state's execution requirements — those are the attorney's judgment calls. It leaves an explicit placeholder instead of guessing.
3. **Required output wrapper**: Every skill output must carry the "ASSISTED DRAFT — ATTORNEY REVIEW & STATE-SPECIFIC VERIFICATION REQUIRED" header and the "Verify before use | Not legal advice" footer (see `prompts/system-prompt.md` for exact text) — both as chat-level text surrounding each draft, never inside the copyable block the attorney or client will use.
4. **Plan-tier warning**: The system prompt must warn that consumer-tier Claude (claude.ai Personal / Pro) must not be used to enter confidential client or matter information.
5. **No facts invented**: The skill must never add family details, asset information, or figures not present in the attorney's intake answers. If a document type's required fields are missing, it flags exactly what's missing rather than guessing, and drafts the other complete document types anyway.
6. **Ambiguity resolution**: The skill must ask which document(s) to draft if unspecified, and must flag (never silently resolve) any inconsistent naming of the same person across documents. Missing firm template is not an ambiguity to ask about — see rule 7.
7. **Placeholder-template fallback, not a refusal**: If no firm template is attached for a document type, the skill uses its own bundled generic placeholder template and says so plainly — it does not stop and ask for a template first. The placeholder must never be presented as state-specific or legally sufficient on its own.
8. **No external actions**: The skill never notarizes, files, records, submits, or schedules a signing ceremony for any document. The attorney and client handle execution manually.

## Internal QA fixtures — tests/skills/

`tests/skills/<skill-name>.md` is the internal QA testing guide for a skill — a standing convention for every plugin built in this repo, alongside (not replacing) the end-user testing guide in `plugin/README.md`. The difference:

- `plugin/README.md` — ships inside the plugin zip, short scenarios with pasted one-liners, aimed at an attorney verifying the install.
- `tests/skills/<skill-name>.md` — internal only, not packaged, uses real attached-folder fixtures under `tests/skills/<skill-name>/` when the skill's input is a workspace folder rather than chat text. Deeper checks (e.g., compliance-wrapper placement, cross-document consistency, partial-drafting edge cases) belong here even when they overlap with `plugin/README.md`'s scenarios.

All fixture data must be clearly synthetic — fictional names, firms, matter numbers — and labeled as such at the top of each fixture file. Never use real client or matter data, even anonymized real data, without checking with Dele first.

## Commit style

Do not include `Co-Authored-By` attribution lines in commit messages.

## Canonical plugin description

Used in `plugin/.claude-plugin/plugin.json` and any marketing copy — keep consistent:

> An estate planning document assembler that populates a basic will, healthcare POA, financial POA, and HIPAA authorization from one intake pass — using your firm's templates, or generic placeholders if none are attached — for solo and small-firm estate planning attorneys who assemble near-identical document sets by hand. Never determines execution requirements or which documents a client needs; attorney reviews and finalizes every document before the client signs.

## Testing

Testing is manual inside Claude Desktop / Cowork — there is no test runner. The `plugin/README.md` is the canonical testing guide. It contains:
- Setup steps (build → install → attach a test intake folder → verify skill loads)
- 10 specific test inputs with exact text to paste and what to check for each

Key scenarios that must pass: all four documents with firm templates and complete intake, no firm templates attached (skill uses placeholders and says so, does not ask first), missing required fields for one document type (skill drafts the others, flags what's missing), document selection not specified (skill asks), attorney asks which documents the client needs (skill declines), attorney asks about state execution requirements (skill declines), no folder attached, edit-and-revise loop, confirmation gate (no document set marked ready, nothing notarized or filed, until attorney confirms), cross-document consistency (same person's name/role matches across documents).

## Notes

- `plugin/.mcp.json` is `{}` — this plugin requires no connector. Intake and template access come from Cowork's attached-workspace-folder model, which needs no separate config. Do not add a connector unless the skill explicitly needs one.
- `plugin/manifest.json` has no `server` block — the plugin variant does not require one. Do not add one.
- `plugin/README.md` and `plugin/CONNECTORS.md` are end-user documentation included in the ZIP bundle; they are not internal developer docs.
- The root `.mcp.json` is gitignored — it holds workspace-level Claude Code MCP credentials and is not part of the plugin artifact.
- `npm run release` passes `--notes-file RELEASE.md` to `gh release create` — create/update `RELEASE.md` at repo root before running it.
- Package scripts use `$npm_package_name` and `$npm_package_version` — keep the `name` field in `package.json` in sync with the plugin slug.
