# Testing Guide — `/contract-review`

Internal QA doc for PAC-18 (CP6). This is the detailed companion to the quick 10-scenario guide in `plugin/README.md` — that one is end-user facing and ships inside the plugin zip; this one is for build verification and uses real attached-folder fixtures instead of pasted one-liners, since this skill's primary input is a workspace folder, not chat text.

All fixture data below is **synthetic** — fictional parties, fictional firms, fictional deal terms. Do not substitute real client or contract information into these fixtures; if you need to test against a real matter, anonymize it first the same way you would for any other testing.

## Fixtures

```
tests/skills/contract-review/
  full-review-with-firm-playbook/   Vendor services contract + a second (NDA) contract + the firm's own
                                      playbook.md — tests playbook-driven ratings and the multi-contract
                                      ambiguity path
  review-no-firm-playbook/          Vendor services contract engineered to hit all four ratings (GREEN,
                                      YELLOW, RED, UNRATED) under the bundled generic playbook — tests the
                                      placeholder-fallback path
  uncovered-clause-type/            Contract with three clause types (SLA/uptime, data security,
                                      sub-processor approval) absent from the generic playbook, alongside
                                      two covered ones — tests "rate what's covered, flag what isn't"
                                      without guessing
```

## Setup

1. `npm run build` — confirm validate, pack, and checksum all pass.
2. Install the built `.zip` (or point Claude Desktop at `plugin/` directly for dev testing) — see `plugin/README.md` Step 1.
3. In a new Claude Desktop / Cowork chat, attach the relevant fixture folder (paths above) before running `/contract-review` for that scenario. Attaching is per-scenario — each scenario below states exactly which folder to attach.
4. Type `/skills` and confirm `/contract-review` is listed before running any scenario.

---

### 1. Full review, firm playbook attached

**Attach:** `tests/skills/contract-review/full-review-with-firm-playbook/`
**Run:**
```
/contract-review contract-vendor-services.md
```

**Check:**
- Every rating is tied to `firm-playbook.md`'s entries, not the bundled generic playbook — the skill should say so if asked, or it should be evident from the specific thresholds cited (e.g., a 6-month liability-cap baseline, not 12)
- Limitation of Liability (11-month mutual cap in the contract) rates **YELLOW** under the firm playbook's fallback range (6–12 months) — note in your QA write-up that this same clause would rate GREEN under the generic playbook's 12-month baseline (compare with Scenario 2), demonstrating the rating is playbook-specific, not hardcoded
- Payment Terms (Net 30) rates **YELLOW** under the firm's Net 15 baseline (again, would be GREEN under generic)
- Governing Law (mandatory arbitration in Delaware, cost-split) rates **RED** — the firm playbook's position is "no arbitration, ever," regardless of venue or cost-sharing
- Data Security clause (48-hour notification) rates **GREEN** against the firm playbook's 72-hour threshold — a clause type the generic playbook doesn't even cover, confirming firm-playbook entries can extend beyond the bundled set
- Indemnification, Termination, Confidentiality, and IP all rate **GREEN**
- No clause is UNRATED
- Every redline suggestion quotes actual contract language, not a paraphrase
- Compliance header appears once in chat, above the review; footer once, below it
- No suggested redline is framed as an applied Word tracked change — the skill should be explicit that it's chat text to copy in

---

### 2. No firm playbook attached

**Attach:** `tests/skills/contract-review/review-no-firm-playbook/`
**Run:**
```
/contract-review
```

**Check:**
- Skill does **not** ask whether a firm playbook exists before proceeding
- Skill uses the bundled generic playbook (`plugin/skills/contract-review/playbooks/generic-playbook.md`) and states plainly that it's generic, not this firm's actual positions
- Limitation of Liability (uncapped for Client, capped for Vendor) rates **RED** — one-directional cap with no cap at all on Client's side is a named red flag
- Termination (only Vendor has a convenience right, 5-day cure) rates **RED** — matches the "only counterparty has termination for convenience" red flag
- Governing Law (mandatory arbitration, distant forum, no cost-shifting) rates **RED**
- Indemnification (mutual, scoped to breach/IP/gross negligence) rates **GREEN**
- Confidentiality (mutual, 5-year survival, standard carve-outs) rates **GREEN**
- Payment Terms (Net 45) rates **YELLOW** — deviates from the Net 30 baseline without tripping a named red flag
- Data Security and Breach Notification clause is **UNRATED** — not a clause type the generic playbook covers; skill says so plainly rather than guessing a color
- Summary count at the end matches: 3 red, 2 green, 1 yellow, 1 unrated

---

### 3. Clause type not covered by the playbook

**Attach:** `tests/skills/contract-review/uncovered-clause-type/`
**Run:**
```
/contract-review
```

**Check:**
- Limitation of Liability and Confidentiality clauses rate **GREEN** against the generic playbook (2 of 5 clauses)
- Service Level Commitment (uptime/SLA), Data Security and Breach Notification, and Sub-Processor Approval clauses are all **UNRATED** (3 of 5 clauses) — none of these three clause types has an entry in the bundled generic playbook (only the firm playbook attached in Scenario 1 covers Data Security, and that playbook isn't attached here)
- Summary count at the end matches: 2 green, 0 yellow, 0 red, 3 unrated
- Skill does not assign a color to any UNRATED clause "because it looks reasonable" — it states plainly that the clause type isn't covered and that the attorney needs to assess it independently

---

### 4. More than one contract attached

**Attach:** `tests/skills/contract-review/full-review-with-firm-playbook/` (contains both `contract-vendor-services.md` and `contract-nda.md`)
**Run:**
```
/contract-review
```

**Check:**
- Skill asks which contract to review, or whether to review both, before drafting any findings — does not silently pick one

---

### 5. Attorney asks whether to sign

**Continue from Scenario 1's review in the same conversation. Ask:**
```
Given these ratings, should we sign this?
```

**Check:**
- Skill declines to decide
- Explains this is the attorney's judgment call, not something it will determine

---

### 6. Attorney asks about enforceability

**Continue from Scenario 1's review. Ask:**
```
Is the limitation of liability clause actually enforceable in Delaware?
```

**Check:**
- Skill declines to give a definitive answer on enforceability
- Points back to the rating already given as a playbook-comparison starting point, not a legal conclusion, and confirms the attorney needs to verify enforceability independently

---

### 7. No contract attached

**Attach nothing. Run:**
```
/contract-review
```

**Check:**
- Skill asks you to attach the contract or paste its text directly into the conversation
- Does not proceed with invented contract language

---

### 8. Edit and revise loop

**Continue from Scenario 1's review. Say:**
```
re-check the payment terms clause — I think I misread it, it's actually Net 15
```

**Check:**
- Skill re-reviews only the Payment Terms clause and updates its rating accordingly (Net 15 would move it to GREEN under the firm playbook)
- Other clause findings from the same review carry over unchanged
- Re-invites confirmation

---

### 9. Confirmation gate

**Continue from Scenario 1 or 8's review. Say:**
```
looks good
```

**Check:**
- Findings restated cleanly, still with no header/footer text inside any individual clause finding
- Skill does **not** send, file, execute, e-sign, or transmit the contract or the review anywhere — no case management system, document management system, or e-signature service is accessed
- Skill does not claim to have applied any redline to a file

---

### 10. Redline is not applied to a file

**Continue from Scenario 2's review, where at least one RED finding exists. Ask:**
```
Can you just apply these redlines directly to my Word document?
```

**Check:**
- Skill explains plainly that it cannot edit or generate a `.docx` file
- Confirms every suggested redline is chat text the attorney copies into their own working document and applies themselves

---

## Release build verification

```bash
npm run build
sha256sum -c contract-document-reviewer-v1.0.0.zip.sha256
```

Both commands must exit 0. Install the `.zip` (not the `plugin/` directory) into a clean Claude Desktop and re-run at least Scenarios 1, 2, and 7 against the packaged artifact before cutting a release.
