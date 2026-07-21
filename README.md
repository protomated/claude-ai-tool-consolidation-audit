# Billing Narrative & Time-Entry Drafter — Claude Desktop Plugin

A Claude Desktop plugin for solo and small-firm attorneys. One skill (`/billing-narrative`) takes rough time-entry notes, an email thread, or calendar event details and drafts a clean, billing-code-appropriate narrative with a suggested time increment. The attorney reviews and pastes the final entry into their billing system.

Distributed free by [Protomated](https://protomated.com).

---

## Repo layout

```text
plugin/           Installable plugin (packaged into .zip)
  .claude-plugin/plugin.json   Identity manifest
  .mcp.json                    Empty — no connectors required
  manifest.json                Display metadata
  prompts/system-prompt.md     Master system prompt — compliance guardrails live here
  skills/billing-narrative/
    SKILL.md                   The single skill (narrative drafting + review gate)

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
| `/billing-narrative` | Takes rough notes → resolves ambiguities one question at a time → drafts a professional billing narrative with suggested time increment → presents draft for attorney review → marks entry ready to paste only after confirmation |

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

This is a content plugin — testing is manual inside Claude Desktop. There is no test runner.

### Setup

1. **Build:** `npm run build` — confirm all three steps pass (validate, pack, checksum).
2. **Install:** Claude Desktop → Customize → Personal Plugins → `+` → point at `plugin/` directory (dev) or drag in the `.zip` (release test).
3. **Verify skill loads:** type `/skills` in a new chat — `/billing-narrative` must appear.

No connectors to authorize. Installation is complete after step 3.

---

### Test inputs and what to check

Run each input below and verify the expected behaviour. Use synthetic or anonymized matter details for all tests.

---

#### 1. Basic conference call

```
/billing-narrative tc w client 30 min re PI settlement, reviewed demand letter, advised to counter at 85k
```

**Check:**
- Skill asks about billing increment style (0.1 hr or 0.25 hr) before or alongside drafting — first session only
- Narrative leads with "Conferred with client" and names the matter type and outcome
- Suggested time: 0.5 hr (or nearest increment)
- Compliance header present: `⚠️ ASSISTED DRAFT — ATTORNEY REVIEW REQUIRED`
- Compliance footer present: `— Drafted with Protomated Billing Narrative Drafter`
- Skill invites confirmation before marking entry ready to paste

---

#### 2. Email exchange

```
/billing-narrative responded to 3 emails from opp counsel re discovery schedule and doc production
```

**Check:**
- Narrative leads with "Corresponded with opposing counsel"
- Specifically names discovery schedule and document production — does not generalize to "re: case matters"
- Suggested time: ~0.3 hr
- Confirmation step present

---

#### 3. Document drafting session

```
/billing-narrative drafted motion for summary judgment, reviewed 3 supporting cases, added argument re proximate cause
```

**Check:**
- Narrative names the motion type and the research done — not just "drafted motion"
- Suggested time: 1.0–2.0 hr; flagged as estimate
- Does not add facts beyond what was in the notes

---

#### 4. Court appearance

```
/billing-narrative attended scheduling conference Judge Smith, Smith v Jones, approx 45 min
```

**Check:**
- Narrative leads with "Attended scheduling conference"
- Includes judge name and matter name from the notes
- Suggested time: 0.8 hr (0.1 style) or 0.75 hr (0.25 style) — nearest to 45 min
- If increment style not yet established, skill asks before drafting

---

#### 5. Research session

```
/billing-narrative researched TX statute of limitations for negligence claims, reviewed 2 cases, drafted memo section on key holdings
```

**Check:**
- Narrative names the research topic and output (memo section)
- Suggested time flagged as estimate with stated basis
- Does not invent a case count or outcome beyond what the notes say

---

#### 6. Ambiguous notes — skill asks, does not guess

```
/billing-narrative worked on Smith file
```

**Check:**
- Skill does **not** draft a narrative immediately
- Asks what was accomplished: "What was the main thing you did on the Smith file?" or equivalent
- Does not fill the gap with plausible-sounding activity
- Only drafts after attorney supplies the missing facts

---

#### 7. UTBMS codes requested

Run the same input as test 1, and when the skill asks about coding style, answer "UTBMS."

**Check:**
- Draft includes a suggested L-code (e.g., L160 Settlement/Non-Binding ADR) and A-code (e.g., A106 Communicate (with client))
- Code suggestion includes a one-line explanation of why that code was chosen
- If the activity spans two codes, skill flags it and suggests splitting

---

#### 8. Multi-activity bundle — skill offers to split

```
/billing-narrative reviewed client intake form, then drafted retainer agreement, then called client to confirm signing
```

**Check:**
- Skill identifies three distinct activities
- Asks: combined entry or separate entries?
- If combined: narrative covers all three in sequence
- If split: drafts three separate entries, each with its own time suggestion

---

#### 9. Edit and revise loop

After any draft, respond: `shorter` — then respond: `looks good`.

**Check:**
- Skill produces a shorter variant without prompting the user for new inputs
- Time suggestion carries over unchanged unless the attorney changes it
- After "looks good," skill restates the final narrative cleanly as ready to paste
- Offers to draft another entry for the matter

---

#### 10. Confirmation gate

After any draft, respond: `looks good`.

**Check:**
- Entry is restated cleanly as the final ready-to-paste narrative
- Skill does **not** submit, record, or transmit the entry anywhere
- No billing system is accessed at any point
- Skill offers to draft another entry; does not close the conversation unilaterally

---

### Release build verification

```bash
npm run build
sha256sum -c billing-narrative-drafter-v1.0.0.zip.sha256
```

Both commands must exit 0. Install the `.zip` (not the `plugin/` directory) into a clean Claude Desktop to confirm the packaged artifact works end to end.

---

## Cutting a release

Update `RELEASE.md` at the repo root, then push a semver tag — CI does the rest:

```bash
git tag v1.0.1
git push origin v1.0.1
```

The release workflow validates, builds, checksums, and publishes a GitHub Release with `billing-narrative-drafter-v1.0.1.zip` and `.sha256` attached.

---

## Compliance

The plugin enforces six non-negotiable rules, defined in `plugin/prompts/system-prompt.md` and `SKILL.md`:

1. **Attorney review gate** — Claude must present every draft and invite confirmation before marking the entry ready to paste. It never declares a draft final unilaterally.
2. **Required output wrapper** — every skill output begins with `⚠️ ASSISTED DRAFT — ATTORNEY REVIEW REQUIRED` and ends with the `— Drafted with Protomated Billing Narrative Drafter | Verify before billing | Not legal advice` footer.
3. **Plan-tier warning** — consumer Claude (Personal/Pro) must not be used with confidential matter information.
4. **No facts invented** — the skill must never add facts not present in the attorney's notes. If notes are too sparse, it asks before proceeding.
5. **Ambiguity resolution** — the skill must ask before drafting if the activity type, multi-entry split, or time basis is unclear. It must never guess.
6. **No external actions** — the skill never submits, records, or transmits entries to any billing system. The attorney pastes the final narrative manually.

Do not weaken these constraints.

---

## License

MIT. See [LICENSE](plugin/LICENSE).
