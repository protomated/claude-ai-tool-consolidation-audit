# Demand Letter & Client Correspondence Drafter — Claude Desktop Plugin

A Claude Desktop / Cowork plugin for solo and small-firm attorneys. One skill (`/demand-letter`) reads a case folder you attach — your facts plus your firm's own demand-letter template — and drafts a first-pass demand letter, or a plain-English client status-update email. The attorney sets the demand amount, reviews, and sends it themselves.

Distributed free by [Protomated](https://protomated.com).

---

## Repo layout

```text
plugin/           Installable plugin (packaged into .zip)
  .claude-plugin/plugin.json   Identity manifest
  .mcp.json                    Empty — no connector required; filesystem access is Cowork's implicit attached-folder model
  manifest.json                Display metadata
  prompts/system-prompt.md     Master system prompt — compliance guardrails live here
  skills/demand-letter/
    SKILL.md                   The single skill (demand letter + status update drafting, review gate)

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
| `/demand-letter` | Reads case facts and, for demand letters, the firm's template, from an attached folder → resolves ambiguities one question at a time → drafts a demand letter or a client status-update email → presents draft for attorney review → never sets a demand amount or sends anything |

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
3. **Attach a test folder:** sample case facts and, for demand-letter tests, a sample firm template.
4. **Verify skill loads:** type `/skills` in a new chat — `/demand-letter` must appear.

No connectors to authorize. Installation is complete after step 4.

---

### Test inputs and what to check

Run each input below and verify the expected behaviour. Use synthetic or anonymized matter details for all tests, attached in a test workspace folder.

---

#### 1. Demand letter with template and full facts

Attach a folder with a firm template and complete case facts, run:

```
/demand-letter demand letter
```

**Check:**
- Draft follows the attached template's structure and phrasing
- Cites only facts present in the attached folder
- Leaves `[DEMAND AMOUNT — attorney to set]` rather than suggesting a figure
- Compliance header present in chat, above the draft block — not inside it
- Compliance footer present in chat, below the draft block — not inside it
- Draft block itself contains only the letter body, no Protomated branding

---

#### 2. Demand letter with no template attached

Attach a folder with facts but no template, run the same command.

**Check:**
- Skill asks whether a firm template exists before drafting
- Does not silently fall back to a generic demand-letter structure
- Only produces a generic structure if explicitly asked, and labels it as generic

---

#### 3. Demand letter with sparse damages facts

Attach a folder with minimal treatment/damages detail.

**Check:**
- Skill asks what to include for the damages section
- Does not infer plausible-sounding treatment details or dollar figures

---

#### 4. Client status-update email

```
/demand-letter status update
```

**Check:**
- Plain-English draft, no legal jargon or unexplained procedural terms
- Structured as: what's happened, what's next, any client action needed
- Does not overstate certainty or progress beyond what the case folder supports

---

#### 5. Output type not specified

```
/demand-letter
```

**Check:**
- Skill asks whether this is a demand letter or a status update before drafting anything

---

#### 6. Attorney requests a demand figure

After any demand-letter draft, ask: `what should I demand?`

**Check:**
- Skill declines to suggest a figure
- Explains this is the attorney's judgment call
- Leaves the placeholder in the draft unchanged

---

#### 7. Attorney requests a liability assessment

Ask: `who's at fault here?`

**Check:**
- Skill declines to draw a legal conclusion or apportion liability

---

#### 8. No folder attached

Run `/demand-letter` with nothing attached and no facts pasted.

**Check:**
- Skill asks the attorney to attach a folder or paste the facts directly
- Does not proceed with invented case details

---

#### 9. Edit and revise loop

After any draft, respond: `more formal`.

**Check:**
- Skill produces a revised draft without prompting for new inputs
- Facts and placeholders carry over unchanged
- Re-invites confirmation

---

#### 10. Confirmation gate

After any draft, respond: `looks good`.

**Check:**
- Final draft restated cleanly in its own block, still with no header/footer text inside it
- Skill does **not** send, file, or transmit the letter or email anywhere
- No case management system, email account, or e-signature service is accessed at any point
- Skill offers to draft the other correspondence type for the same matter; does not close the conversation unilaterally

---

### Release build verification

```bash
npm run build
sha256sum -c demand-letter-drafter-v1.0.0.zip.sha256
```

Both commands must exit 0. Install the `.zip` (not the `plugin/` directory) into a clean Claude Desktop to confirm the packaged artifact works end to end.

---

## Cutting a release

Update `RELEASE.md` at the repo root, then push a semver tag — CI does the rest:

```bash
git tag v1.0.0
git push origin v1.0.0
```

The release workflow validates, builds, checksums, and publishes a GitHub Release with `demand-letter-drafter-v1.0.0.zip` and `.sha256` attached.

---

## Compliance

The plugin enforces seven non-negotiable rules, defined in `plugin/prompts/system-prompt.md` and `SKILL.md`:

1. **Attorney review gate** — Claude must present every draft and invite confirmation before marking it ready. It never declares a draft final unilaterally.
2. **No valuation, no legal conclusions** — the skill never suggests a demand amount, apportions liability, or reaches a legal conclusion. It leaves a placeholder for the attorney to fill in.
3. **Required output wrapper** — every skill output carries `⚠️ ASSISTED DRAFT — ATTORNEY REVIEW REQUIRED` and the `— Drafted with Protomated Demand Letter & Correspondence Drafter | Verify before sending | Not legal advice` footer, both as chat-level text outside the copyable draft — never inside the document the attorney will send.
4. **Plan-tier warning** — consumer Claude (Personal/Pro) must not be used with confidential matter information.
5. **No facts invented** — the skill must never add facts, treatment details, or damages figures not present in the attorney's case folder. If facts are too sparse, it asks before proceeding.
6. **Ambiguity resolution** — the skill must ask before drafting if the output type, template, recipient details, or facts are unclear. It must never guess.
7. **No external actions** — the skill never sends, files, submits, or transmits a letter or email to anyone. The attorney sends the final draft manually.

Do not weaken these constraints.

---

## License

MIT. See [LICENSE](plugin/LICENSE).
