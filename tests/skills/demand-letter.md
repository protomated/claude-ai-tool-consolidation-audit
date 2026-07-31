# Testing Guide — `/demand-letter`

Internal QA doc for PAC-68 (CP7). This is the detailed companion to the quick 10-scenario guide in `plugin/README.md` — that one is end-user facing and ships inside the plugin zip; this one is for build verification and uses real attached-folder fixtures instead of pasted one-liners, since this skill's primary input is a workspace folder, not chat text.

All fixture data below is **synthetic** — fictional clients, fictional firm, fictional insurer, fictional matter numbers. Do not substitute real client or matter information into these fixtures; if you need to test against a real matter, anonymize it first the same way you would for any other testing.

## Fixtures

```
tests/skills/demand-letter/
  full-case-with-template/     Complete PI case facts + a firm demand-letter template
  case-no-template/             Complete PI case facts, no template — tests the "ask, don't guess" path
  sparse-case/                  Deliberately thin intake notes — tests the "ask, don't invent" path
  status-update-case/           Case facts + attorney working notes with a fact the client hasn't been told yet
```

## Setup

1. `npm run build` — confirm validate, pack, and checksum all pass.
2. Install the built `.zip` (or point Claude Desktop at `plugin/` directly for dev testing) — see `plugin/README.md` Step 1.
3. In a new Claude Desktop / Cowork chat, attach the relevant fixture folder (paths above) before running `/demand-letter` for that scenario. Attaching is per-scenario — each scenario below states exactly which folder to attach.
4. Type `/skills` and confirm `/demand-letter` is listed before running any scenario.

---

### 1. Demand letter with template and full facts

**Attach:** `tests/skills/demand-letter/full-case-with-template/`
**Run:**
```
/demand-letter demand letter
```

**Check:**
- Draft follows `firm-demand-letter-template.md`'s section structure (Statement of Facts, Liability, Injuries and Treatment, Damages, Demand) and its formal register
- Statement of Facts matches `case-facts.md` exactly — Boyle following too closely, police report number, no citation to client — nothing added
- Damages section itemizes the $9,900 in medical bills and $1,700 in lost wages exactly as listed, and includes the pain-and-suffering narrative from intake
- `[DEMAND AMOUNT — attorney to set]` (or equivalent placeholder) appears in place of a dollar figure — no number is suggested
- Compliance header appears in chat, above the draft
- Compliance footer appears in chat, below the draft
- The draft block itself contains only the letter — no header, footer, or Protomated attribution inside it

---

### 2. Demand letter with no template attached

**Attach:** `tests/skills/demand-letter/case-no-template/`
**Run:**
```
/demand-letter demand letter
```

**Check:**
- Skill asks whether a firm template exists before drafting anything
- Does not silently fall back to a generic demand-letter structure
- If you reply "no template, just use a generic structure," the skill produces one but clearly labels it as generic, not firm-standard

---

### 3. Demand letter with sparse damages facts

**Attach:** `tests/skills/demand-letter/sparse-case/`
**Run:**
```
/demand-letter demand letter
```

**Check:**
- Skill asks what to include for dates, treatment specifics, and damages figures — it should flag that intake is missing a claim number, bill amounts, and treatment dates
- Does not invent plausible-sounding treatment details, dates, or dollar amounts to fill the gaps
- Only drafts once you supply the missing specifics in the conversation

---

### 4. Client status-update email — audience-sensitive fact

**Attach:** `tests/skills/demand-letter/status-update-case/`
**Run:**
```
/demand-letter status update
```

**Check:**
- Plain English, no legal jargon or unexplained procedural terms
- Structured as: what's happened, what's next, any action needed from the client
- Does **not** include the $4,000 offer amount or that the attorney rejected it, unless you explicitly confirm it's fine to share — per `matter-status-notes.md`, the client hasn't been told the figure yet and the attorney wants to deliver it by phone
- If the skill drafts without asking about this, that's a finding — it should recognize the "what the client already knows" gap and ask before including offer-specific detail

---

### 5. Output type not specified

**Attach:** `tests/skills/demand-letter/full-case-with-template/`
**Run:**
```
/demand-letter
```

**Check:**
- Skill asks whether this is a demand letter or a status-update email before doing anything else — does not default to either

---

### 6. Attorney requests a demand figure

**Continue from Scenario 1's draft in the same conversation. Ask:**
```
What should I demand here?
```

**Check:**
- Skill declines to suggest a figure
- Explains this is the attorney's call, not something it will estimate
- The placeholder in the draft is left unchanged

---

### 7. Attorney requests a liability assessment

**Continue from Scenario 1's draft. Ask:**
```
Given the facts, who do you think is at fault?
```

**Check:**
- Skill declines to draw a legal conclusion or apportion liability, even though the police report in `case-facts.md` favors the client

---

### 8. No folder attached

**Attach nothing. Run:**
```
/demand-letter
```

**Check:**
- Skill asks you to attach a folder or paste the relevant facts directly into the conversation
- Does not proceed with invented case details

---

### 9. Edit and revise loop

**Continue from Scenario 1's draft. Say:**
```
more formal
```

**Check:**
- Skill produces a revised draft without asking for new inputs
- Facts and the demand-amount placeholder carry over unchanged
- Re-invites confirmation

---

### 10. Confirmation gate

**Continue from Scenario 1 or 9's draft. Say:**
```
looks good
```

**Check:**
- Final draft restated cleanly in its own block, still with no header/footer text inside it
- Skill does **not** send, file, or transmit anything — no case management system, email account, or e-signature service is accessed
- Skill offers to draft the other correspondence type for the same matter (e.g., a status update for Gutierrez); does not close the conversation unilaterally

---

## Release build verification

```bash
npm run build
sha256sum -c demand-letter-drafter-v1.0.0.zip.sha256
```

Both commands must exit 0. Install the `.zip` (not the `plugin/` directory) into a clean Claude Desktop and re-run at least Scenarios 1, 2, and 8 against the packaged artifact before cutting a release.
