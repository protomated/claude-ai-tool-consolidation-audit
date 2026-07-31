# Testing Guide — `/estate-documents`

Internal QA doc for PAC-71 (CP10). This is the detailed companion to the quick 10-scenario guide in `plugin/README.md` — that one is end-user facing and ships inside the plugin zip; this one is for build verification and uses real attached-folder fixtures instead of pasted one-liners, since this skill's primary input is a workspace folder, not chat text.

All fixture data below is **synthetic** — fictional clients, fictional firm, fictional matter numbers. Do not substitute real client or matter information into these fixtures; if you need to test against a real matter, anonymize it first the same way you would for any other testing.

## Fixtures

```
tests/skills/estate-documents/
  full-intake-with-templates/   Complete intake for all four documents + the firm's own templates for each
  intake-no-templates/          Complete intake, no templates — tests the placeholder-fallback path
  partial-intake/                Complete Will and HIPAA fields; Healthcare POA missing treatment
                                  preferences, Financial POA missing an agent — tests the
                                  "draft what's complete, flag what's blocked" path
```

## Setup

1. `npm run build` — confirm validate, pack, and checksum all pass.
2. Install the built `.zip` (or point Claude Desktop at `plugin/` directly for dev testing) — see `plugin/README.md` Step 1.
3. In a new Claude Desktop / Cowork chat, attach the relevant fixture folder (paths above) before running `/estate-documents` for that scenario. Attaching is per-scenario — each scenario below states exactly which folder to attach.
4. Type `/skills` and confirm `/estate-documents` is listed before running any scenario.

---

### 1. All four documents, firm templates attached, complete intake

**Attach:** `tests/skills/estate-documents/full-intake-with-templates/`
**Run:**
```
/estate-documents all
```

**Check:**
- Each of the four drafts follows its matching `firm-*-template.md` section structure and register
- Testator/principal name "Diane R. Whitfield" is spelled identically in every document
- Thomas J. Whitfield appears as primary agent/executor context consistently; Renee Castellano appears as alternate/agent consistently, with matching relationship label ("sister") wherever she recurs
- Will: executor, guardian, residuary, and specific bequests (piano to Ava, coin collection to Lucas) match `intake-answers.md` exactly — nothing added
- Healthcare POA: life support, pain management, and organ donation preferences match intake exactly
- Financial POA: powers listed (banking/bill pay, real estate, tax matters) match intake, effective-immediately/durable language present
- HIPAA: authorized recipients (Thomas J. Whitfield, Renee Castellano) match intake
- State-specific execution language is left as a placeholder in every document — no witness count, notarization, or self-proving affidavit language is invented
- Compliance header appears once in chat, above the draft set
- Compliance footer appears once in chat, below the draft set
- Each draft block contains only that document's body — no header, footer, or Protomated attribution inside any of them

---

### 2. No firm templates attached

**Attach:** `tests/skills/estate-documents/intake-no-templates/`
**Run:**
```
/estate-documents all
```

**Check:**
- Skill does **not** ask whether a firm template exists before proceeding
- Skill uses the plugin's bundled placeholder templates (`plugin/skills/estate-documents/templates/`) for all four documents
- Skill states plainly, for each document, that the placeholder is generic and not state-specific
- Organ donation preference for Harold Okonkwo is left as an attorney-to-confirm placeholder (intake explicitly leaves it undecided) rather than defaulted to yes/no

---

### 3. Missing required fields for two document types

**Attach:** `tests/skills/estate-documents/partial-intake/`
**Run:**
```
/estate-documents all
```

**Check:**
- Will drafts successfully (all required fields present, including both guardian primary and alternate)
- HIPAA Authorization drafts successfully
- Healthcare Power of Attorney is **not** drafted — skill names the missing field: no life-support/treatment preference on file
- Financial (Durable) Power of Attorney is **not** drafted — skill names the missing field: no agent named, no powers specified
- Skill does not invent a plausible-sounding treatment preference or agent to fill either gap
- Skill clearly separates "here's what I drafted" from "here's what's blocked and why" in the same response

---

### 4. Document(s) not specified

**Attach:** `tests/skills/estate-documents/full-intake-with-templates/`
**Run:**
```
/estate-documents
```

**Check:**
- Skill asks which document(s) to draft (will, healthcare POA, financial POA, HIPAA, or all) before doing anything else — does not default to drafting all four

---

### 5. Attorney asks which documents the client needs

**Continue from Scenario 1's draft in the same conversation. Ask:**
```
Does Diane need a trust too, given the minor's trust language in the will?
```

**Check:**
- Skill declines to decide
- Explains this is the attorney's judgment call, not something it will determine

---

### 6. Attorney asks about state execution requirements

**Continue from Scenario 1's draft. Ask:**
```
How many witnesses do I need for the will and the healthcare POA in my state?
```

**Check:**
- Skill declines to give a definitive answer
- Points to the execution-requirements placeholder already left in each draft, and confirms the attorney needs to verify this independently against governing state law

---

### 7. No folder attached

**Attach nothing. Run:**
```
/estate-documents
```

**Check:**
- Skill asks you to attach a folder or paste the relevant intake answers directly into the conversation
- Does not proceed with invented client or family details

---

### 8. Edit and revise loop

**Continue from Scenario 1's draft. Say:**
```
add a no-contest clause section to the will
```

**Check:**
- Skill produces a revised Will draft without asking for new inputs beyond the requested addition
- The other three documents and all previously drafted facts/placeholders carry over unchanged
- Cross-document consistency (names, agents) is preserved
- Re-invites confirmation

---

### 9. Confirmation gate

**Continue from Scenario 1 or 8's draft. Say:**
```
looks good
```

**Check:**
- Final set restated cleanly in its own blocks, still with no header/footer text inside them
- Skill does **not** notarize, file, record, or submit any document — no case management system, e-signature service, or state filing system is accessed
- Skill offers to draft any document type not yet requested for this client; does not close the conversation unilaterally

---

### 10. Cross-document consistency

**Using Scenario 1's completed draft set, check without re-running anything:**

- "Renee Castellano" is spelled identically in the Will (executor/guardian) and the Healthcare POA (alternate agent)
- "Thomas J. Whitfield" is spelled identically in the Will (residuary beneficiary), Healthcare POA (primary agent), Financial POA (primary agent), and HIPAA Authorization (authorized recipient)
- Primary/alternate ordering for Thomas J. Whitfield and Renee Castellano is consistent between the Healthcare POA and Financial POA
- Date format is consistent across all four documents

---

## Release build verification

```bash
npm run build
sha256sum -c estate-planning-document-assembler-v1.0.0.zip.sha256
```

Both commands must exit 0. Install the `.zip` (not the `plugin/` directory) into a clean Claude Desktop and re-run at least Scenarios 1, 2, and 7 against the packaged artifact before cutting a release.
