# Estate Planning Document Assembly Skill — Claude Desktop Plugin

A Claude Desktop / Cowork plugin for solo and small-firm estate planning attorneys. One skill (`/estate-documents`) reads a client's intake answers — family structure, assets, beneficiaries, healthcare wishes — plus, optionally, the firm's own state-specific templates, and populates a basic will, healthcare power of attorney, financial power of attorney, and HIPAA authorization consistently from one intake pass. The attorney verifies state execution formalities, reviews, and finalizes each document before the client signs.

Distributed free by [Protomated](https://protomated.com).

---

## Repo layout

```text
plugin/           Installable plugin (packaged into .zip)
  .claude-plugin/plugin.json   Identity manifest
  .mcp.json                    Empty — no connector required; filesystem access is Cowork's implicit attached-folder model
  manifest.json                Display metadata
  prompts/system-prompt.md     Master system prompt — compliance guardrails live here
  skills/estate-documents/
    SKILL.md                   The single skill (four-document assembly, review gate)
    templates/                 FREE-tier generic placeholder templates (one per document type)
    reference/intake-checklist.md   Required/optional intake fields per document type

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
| `/estate-documents` | Reads intake answers and, optionally, the firm's own state-specific templates, from an attached folder → checks required fields per document type → drafts a basic will, healthcare POA, financial POA, and/or HIPAA authorization, keeping names and agents consistent across the set → presents draft for attorney review → never determines execution requirements or notarizes/files/submits anything |

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
3. **Attach a test folder:** sample intake answers and, optionally, sample firm templates for one or more document types.
4. **Verify skill loads:** type `/skills` in a new chat — `/estate-documents` must appear.

No connectors to authorize. Installation is complete after step 4.

---

### Test inputs and what to check

Run each input below and verify the expected behaviour. Use synthetic or anonymized client details for all tests, attached in a test workspace folder.

---

#### 1. All four documents, firm templates attached, complete intake

Attach a folder with firm templates and complete intake, run:

```
/estate-documents all
```

**Check:**
- Each draft follows its attached template's structure and phrasing
- Cites only facts present in the attached intake
- Names, agents, and dates are consistent across all four documents
- Compliance header present in chat, above the draft set — not inside any draft
- Compliance footer present in chat, below the draft set — not inside any draft
- Each draft block contains only that document's body, no Protomated branding

---

#### 2. No firm templates attached

Attach a folder with complete intake but no templates, run the same command.

**Check:**
- Skill uses its own bundled placeholder templates without asking for a firm template first
- Says plainly, for each document, that the placeholder is generic and not state-specific

---

#### 3. Missing required fields for one document type

Attach a folder where one document type (e.g., financial POA) is missing a required field.

**Check:**
- Skill drafts the other, complete document types
- Lists exactly what's missing for the blocked document type
- Does not infer or guess a plausible-sounding value to fill the gap

---

#### 4. Document(s) not specified

```
/estate-documents
```

**Check:**
- Skill asks which document(s) to draft before doing anything else

---

#### 5. Attorney asks which documents the client needs

After any draft, ask: `does this client need a trust too?`

**Check:**
- Skill declines to decide
- Explains this is the attorney's judgment call

---

#### 6. Attorney asks about state execution requirements

Ask: `how many witnesses does my state require?`

**Check:**
- Skill declines to give a definitive answer
- Points to the execution-requirements placeholder in the draft for the attorney to verify independently

---

#### 7. No folder attached

Run `/estate-documents` with nothing attached and no intake pasted.

**Check:**
- Skill asks the attorney to attach a folder or paste the intake answers directly
- Does not proceed with invented client details

---

#### 8. Edit and revise loop

After any draft, respond: `add a section`.

**Check:**
- Skill produces a revised draft without prompting for new inputs
- Facts and placeholders carry over unchanged
- Cross-document consistency is preserved
- Re-invites confirmation

---

#### 9. Confirmation gate

After any draft set, respond: `looks good`.

**Check:**
- Final set restated cleanly in its own blocks, still with no header/footer text inside them
- Skill does **not** notarize, file, record, or submit any document
- No case management system, e-signature service, or state filing system is accessed at any point
- Skill offers to draft any remaining document type for the same client; does not close the conversation unilaterally

---

#### 10. Cross-document consistency

Across a completed draft set (e.g., from Scenario 1), check the same person named as healthcare agent in the Healthcare POA appears with identically spelled name and matching role wherever they recur (e.g., the HIPAA Authorization).

**Check:**
- Name spelling matches exactly across every document
- Primary/alternate ordering matches wherever the same people appear in more than one document

---

### Release build verification

```bash
npm run build
sha256sum -c estate-planning-document-assembler-v1.0.0.zip.sha256
```

Both commands must exit 0. Install the `.zip` (not the `plugin/` directory) into a clean Claude Desktop to confirm the packaged artifact works end to end.

---

## Cutting a release

Update `RELEASE.md` at the repo root, then push a semver tag — CI does the rest:

```bash
git tag v1.0.0
git push origin v1.0.0
```

The release workflow validates, builds, checksums, and publishes a GitHub Release with `estate-planning-document-assembler-v1.0.0.zip` and `.sha256` attached.

---

## Compliance

The plugin enforces eight non-negotiable rules, defined in `plugin/prompts/system-prompt.md` and `SKILL.md`:

1. **Attorney review gate** — Claude must present every draft and invite confirmation before marking a document set ready. It never declares a set final unilaterally.
2. **No legal judgment** — the skill never decides which documents a client needs, resolves a family/guardianship conflict, advises on tax strategy, assesses capacity/undue influence, or determines a state's execution requirements. It leaves a placeholder for the attorney to fill in.
3. **Required output wrapper** — every skill output carries `⚠️ ASSISTED DRAFT — ATTORNEY REVIEW & STATE-SPECIFIC VERIFICATION REQUIRED` and the `— Drafted with Protomated Estate Planning Document Assembler | Verify before use | Not legal advice` footer, both as chat-level text outside every copyable draft — never inside a document the client might sign.
4. **Plan-tier warning** — consumer Claude (Personal/Pro) must not be used with confidential client or matter information.
5. **No facts invented** — the skill must never add family details, asset information, or figures not present in the attorney's intake answers. If required fields for a document type are missing, it flags exactly what's missing and drafts the other document types anyway.
6. **Ambiguity resolution** — the skill must ask which document(s) to draft if unspecified, and must flag rather than silently resolve any inconsistent naming of the same person across documents.
7. **Placeholder-template fallback, not a refusal** — a missing firm template is not a reason to stop and ask; the skill uses its own bundled generic placeholder and says so plainly, never presenting it as state-specific or legally sufficient on its own.
8. **No external actions** — the skill never notarizes, files, records, submits, or schedules a signing ceremony for any document. The attorney and client handle execution manually.

Do not weaken these constraints.

---

## License

MIT. See [LICENSE](plugin/LICENSE).
