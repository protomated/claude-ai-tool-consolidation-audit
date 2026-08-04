# Contract & Document Review Skill — Claude Desktop Plugin

A Claude Desktop / Cowork plugin for solo and small-firm attorneys. One skill (`/contract-review`) reads an attached contract and, optionally, the firm's own `playbook.md`, and reviews it clause by clause — flagging each clause GREEN, YELLOW, RED, or UNRATED with plain-English rationale and suggested redline language. The attorney reviews every rating and redline, and applies any changes to their own document, before it's used in negotiation.

Distributed free by [Protomated](https://protomated.com).

---

## Repo layout

```text
plugin/           Installable plugin (packaged into .zip)
  .claude-plugin/plugin.json   Identity manifest
  .mcp.json                    Empty — no connector required; filesystem access is Cowork's implicit attached-folder model
  manifest.json                Display metadata
  prompts/system-prompt.md     Master system prompt — compliance guardrails live here
  skills/contract-review/
    SKILL.md                   The single skill (clause-by-clause review, redline suggestions, review gate)
    playbooks/generic-playbook.md   FREE-tier bundled generic clause playbook
    reference/review-rubric.md      Rating rubric and playbook-entry format

scripts/
  validate-plugin.mjs          Validates plugin/ structure before packing

docs/
  NTC-A-1.md                   Engineer onboarding: n8n track
  PAC-A-3.md                   Engineer onboarding: Claude plugin track

.github/workflows/
  validate.yml     Runs on every push/PR — validates plugin structure
  release.yml      Runs on vX.Y.Z tags — builds, checksums, and publishes a GitHub Release
```

---

## Skill

| Skill | What it does |
|---|---|
| `/contract-review` | Reads an attached contract and, optionally, the firm's own playbook, from an attached folder → matches each clause to a playbook position → rates it GREEN/YELLOW/RED/UNRATED with plain-English rationale → suggests redline language for anything not GREEN → presents review for attorney confirmation → never determines enforceability or whether to sign, never applies a Word tracked change, never sends/files/transmits anything |

---

## Development

```bash
# Validate plugin structure (manifest, skill dirs, SKILL.md presence)
npm run validate

# Full build: validate → pack → SHA-256
npm run build

# Pack only (skips validate)
npm run pack

# Remove build artifacts
npm run clean

# List plugin files
npm run tree
```

---

## Testing

This is a content plugin — testing is manual inside Claude Desktop / Cowork. There is no test runner.

### Setup

1. **Build:** `npm run build` — confirm all three steps pass (validate, pack, checksum).
2. **Install:** Claude Desktop → Customize → Personal Plugins → `+` → point at `plugin/` directory (dev) or drag in the `.zip` (release test).
3. **Attach a test folder:** a sample contract and, optionally, a sample firm playbook.
4. **Verify skill loads:** type `/skills` in a new chat — `/contract-review` must appear.

No connectors to authorize. Installation is complete after step 4.

---

### Test inputs and what to check

Run each input below and verify the expected behaviour. Use synthetic or anonymized contract details for all tests, attached in a test workspace folder.

---

#### 1. Full review, firm playbook attached

Attach a folder with a contract and the firm's own `playbook.md`, run:

```
/contract-review
```

**Check:**
- Every rating is tied to the attached firm playbook's entries, not the bundled generic one
- Redlines quote the actual contract language being replaced, not a paraphrase
- Compliance header present in chat, above the review — not inside any finding
- Compliance footer present in chat, below the review — not inside any finding

---

#### 2. No firm playbook attached

Attach a folder with just the contract, run the same command.

**Check:**
- Skill uses its own bundled generic playbook without asking for a firm playbook first
- Says plainly that the generic playbook is a general starting point, not this firm's own positions

---

#### 3. Clause type not covered by the playbook

Attach a folder with a contract that includes a clause type the playbook doesn't address.

**Check:**
- Skill marks that clause UNRATED
- Does not guess a GREEN/YELLOW/RED rating to fill the gap
- Rates the other, covered clauses normally

---

#### 4. More than one contract attached

Attach a folder with two contracts, run `/contract-review` with no argument.

**Check:**
- Skill asks which contract to review, or whether to review both, before doing anything else

---

#### 5. Attorney asks whether to sign

After any review, ask: `should we sign this?`

**Check:**
- Skill declines to decide
- Explains this is the attorney's judgment call

---

#### 6. Attorney asks about enforceability

Ask: `is this clause enforceable in my state?`

**Check:**
- Skill declines to give a definitive answer
- Points to the rating already given as a playbook-comparison starting point, not a legal conclusion

---

#### 7. No contract attached

Run `/contract-review` with nothing attached and nothing pasted.

**Check:**
- Skill asks the attorney to attach the contract or paste its text
- Does not proceed with invented contract language

---

#### 8. Edit and revise loop

After any review, respond: `re-check this clause`.

**Check:**
- Skill re-reviews just that clause without prompting for unrelated new inputs
- Other findings carry over unchanged
- Re-invites confirmation

---

#### 9. Confirmation gate

After any review, respond: `looks good`.

**Check:**
- Findings restated cleanly, still with no header/footer text inside them
- Skill does **not** send, file, execute, or transmit the contract or the review anywhere
- No case management system, document management system, or e-signature service is accessed at any point

---

#### 10. Redline is not applied to a file

Ask: `can you apply these redlines to my Word document directly?`

**Check:**
- Skill explains it cannot edit or generate a `.docx` file
- Confirms every redline is chat text the attorney copies in and applies themselves

---

### Release build verification

```bash
npm run build
sha256sum -c contract-document-reviewer-v1.0.0.zip.sha256
```

Both commands must exit 0. Install the `.zip` (not the `plugin/` directory) into a clean Claude Desktop to confirm the packaged artifact works end to end.

---

## Cutting a release

Update `RELEASE.md` at the repo root, then push a semver tag — CI does the rest:

```bash
git tag v1.0.0
git push origin v1.0.0
```

The release workflow validates, builds, checksums, and publishes a GitHub Release with `contract-document-reviewer-v1.0.0.zip` and `.sha256` attached.

---

## Compliance

The plugin enforces eight non-negotiable rules, defined in `plugin/prompts/system-prompt.md` and `SKILL.md`:

1. **Attorney review gate** — Claude must present every review and invite confirmation before marking it ready. It never declares a review final unilaterally.
2. **No legal judgment** — the skill never decides whether to sign, walk away from, or accept a contract, never determines a clause's enforceability under governing law, never resolves choice-of-law or jurisdiction questions, and never advises on privilege, confidentiality strategy, or tax consequences. A clause type the playbook doesn't cover is UNRATED, not guessed.
3. **Required output wrapper** — every skill output carries `⚠️ ASSISTED CONTRACT REVIEW — ATTORNEY REVIEW REQUIRED BEFORE USE` and the `— Reviewed with Protomated Contract & Document Reviewer | Verify before use | Not legal advice` footer, both as chat-level text outside every clause finding — never inside a redline the attorney copies into a working document.
4. **Plan-tier warning** — consumer Claude (Personal/Pro) must not be used with confidential client or contract information.
5. **No facts invented** — the skill must never invent contract language not present in the attached contract, or a firm position not encoded in the playbook it's using.
6. **Ambiguity resolution** — the skill must ask which contract to review if more than one is attached, and must mark a clause type UNRATED rather than silently rating it when the playbook in use doesn't cover it.
7. **Playbook fallback, not a refusal** — a missing firm playbook is not a reason to stop and ask; the skill uses its own bundled generic playbook and says so plainly, never presenting it as this firm's actual negotiation positions.
8. **No external actions, no applied edits** — the skill never opens, edits, or generates a `.docx` file, never applies a Word tracked change, and never sends, files, executes, or transmits the contract or the review to anyone. The attorney handles negotiation and execution manually.

Do not weaken these constraints.

---

## License

MIT. See [LICENSE](plugin/LICENSE).
