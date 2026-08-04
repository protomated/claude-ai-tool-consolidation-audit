# Testing Guide — `/ai-tool-audit`

Internal QA doc for CP18. This is the detailed companion to the quick 10-scenario guide in `plugin/README.md` — that one is end-user facing and ships inside the plugin zip; this one is for build verification and uses a real attached-folder fixture for the inventory-attached path, plus scripted chat answers for the interview-only path, since this skill's primary input is a guided chat interview with an optional workspace-folder inventory, not a single file to read cold.

All fixture data below is **synthetic** — a fictional firm (Hartwell & Voss LLP) and fictional AI-tool product names. Do not substitute real client, matter, or vendor account information into these fixtures; if you need to test against a real firm's actual tool landscape, anonymize it first the same way you would for any other testing.

## Fixtures

```
tests/skills/ai-tool-audit/
  full-audit-with-inventory-attached/   ai-tools-inventory.md — Hartwell & Voss LLP's rough written list of 7 AI
                                          tools, phrased the way a partner would actually describe them (not
                                          pre-formatted to the skill's own rubric) — tests the attached-inventory
                                          path, the "confirm it's complete" ambiguity rule, and every rating tier
  interview-only-no-attachment/         Empty except for a README — tests the from-scratch guided-interview path
                                          with scripted chat answers given below
```

## Setup

1. `npm run build` — confirm validate, pack, and checksum all pass.
2. Install the built `.zip` (or point Claude Desktop at `plugin/` directly for dev testing) — see `plugin/README.md` Step 1.
3. In a new Claude Desktop / Cowork chat, attach the relevant fixture folder (paths above) before running `/ai-tool-audit` for a scenario that needs one. Attaching is per-scenario.
4. Type `/skills` and confirm `/ai-tool-audit` is listed before running any scenario.

---

### 1. Full audit, existing inventory attached

**Attach:** `tests/skills/ai-tool-audit/full-audit-with-inventory-attached/`
**Run:**
```
/ai-tool-audit
```

**Check:**
- Skill asks whether `ai-tools-inventory.md` is the firm's complete list before treating it as final (does not silently assume 7 tools is everything)
- Once confirmed, fills in any remaining detail through a short interview rather than re-asking fully-described facts already in the file
- **QuickBrief AI** rates **🔴 confirmed risk** — client names are typed into a free/consumer-tier tool with no DPA
- **Clio Manage** rates **🟢 confirmed appropriate** — business tier with a signed DPA — and is explicitly named a **keep-as-specialized** recommendation (practice management)
- **DeadlineGuard** rates **🟢 confirmed appropriate** and is also named **keep-as-specialized** (jurisdiction-specific docketing/deadline logic)
- **NotesApp Free** rates **🔴 confirmed risk** — client-identifying call notes on a free consumer app with no DPA
- **SummarizeIt** rates **⚪ UNCONFIRMED** — the firm doesn't know its data-handling terms; skill does not guess, and does not assert what a real product with this description would actually do
- **TranscribeFast** rates **🟡 partial/mixed** — paid/business tier confirmed, but the signed DPA itself is not confirmed
- **MarketingBot** rates **🟢 confirmed appropriate** — the reason cites the no-sensitive-data branch of the rubric (never touches client information), not a confirmed tier, since the tier is free/consumer
- Redundancy flagged: QuickBrief AI's research-summary use and SummarizeIt both serve the same case-research-summary workflow
- Consolidation recommendation ties each candidate to a specific workflow: (a) client email drafting + case research summaries (QuickBrief AI + SummarizeIt) and (b) client intake call notes (NotesApp Free) — not a blanket "replace everything"
- Clio Manage and DeadlineGuard are recommended to **keep**, named explicitly, not just absent from the consolidation list
- Summary count matches: 7 tools audited — 3 green, 1 yellow, 2 red, 1 unconfirmed; 2 consolidation-candidate workflows; 2 tools recommended to keep as specialized
- Compliance header appears once in chat, above the audit; footer once, below it — neither embedded inside a table row or section

---

### 2. Interview only, no attachment

**Attach:** `tests/skills/ai-tool-audit/interview-only-no-attachment/` (or attach nothing)
**Run:**
```
/ai-tool-audit
```

**When asked, answer with:**
```
We use three things: a free AI chatbot called DraftPal for writing client letters, sometimes with 
client names in the prompt; our billing software's built-in AI feature for generating time-entry 
narratives (paid tier, not sure about their DPA); and a free app one of our assistants uses to 
transcribe internal staff meeting notes — no client info in those, just internal planning.
```

**Check:**
- Skill builds a full inventory purely from the chat answers, asking follow-up questions for anything not covered (e.g., who uses each tool, if not stated)
- "DraftPal" rates **🔴 confirmed risk** (client names, free tier)
- The billing AI feature rates **🟡 partial/mixed** (paid tier confirmed, DPA unconfirmed)
- The internal-meeting-notes app rates **🟢 confirmed appropriate** — reason cites the no-sensitive-data branch of the rubric (no client info at all), not a confirmed tier, since it's a free consumer app
- Skill does not ask for a written list before proceeding — the interview alone is sufficient

---

### 3. Specialized tool, no consolidation recommended

**Continue from Scenario 1's audit, or re-run with the same fixture. Confirm:**

**Check:**
- Clio Manage and DeadlineGuard are each named individually with a specific reason to keep them, not lumped into a generic "these are fine" line
- Neither appears in the Consolidation Recommendation section as a candidate for replacement
- If asked directly — `should we move Clio and DeadlineGuard onto the new stack too?` — skill explains they're specialized/integrated tools outside what a general Claude + MCP setup would replace, and reaffirms the keep recommendation

---

### 4. Unconfirmed data-handling status is never guessed

**Continue from Scenario 1. Ask:**
```
What's SummarizeIt's actual data policy — do they train on our inputs?
```

**Check:**
- Skill does not answer with a specific claim about a real product's policy
- Reiterates that this is UNCONFIRMED and the firm needs to check directly with the vendor
- Does not change SummarizeIt's rating based on the question alone

---

### 5. Redundant tools

**Continue from Scenario 1. Ask:**
```
Do QuickBrief AI and SummarizeIt actually overlap?
```

**Check:**
- Skill confirms the overlap (both used for case-research summaries) and explains why it's flagged
- Does not claim one tool must be cancelled — presents it as a fact for the firm to weigh, tied to the consolidation recommendation already given

---

### 6. Firm has no AI tools

**Attach nothing. Run:**
```
/ai-tool-audit
```
**When asked what AI tools the firm uses, respond:**
```
Honestly, none that I can think of.
```

**Check:**
- Skill asks the firm to reconsider common ones (dictation, a research tool's AI feature, an email client's AI drafting assistant) before accepting that
- If the firm still confirms nothing, skill says so plainly and stops — does not invent an inventory to audit

---

### 7. Attorney asks for the firm's AI-use policy

**Continue from Scenario 1's audit. Ask:**
```
Can you draft our firm's AI-use policy based on this audit?
```

**Check:**
- Skill declines
- Explains it audits and recommends a stack — drafting policy documents is outside what it produces

---

### 8. Attorney asks the skill to act on its own recommendation

**Continue from Scenario 1's audit. Ask:**
```
Can you go ahead and cancel our SummarizeIt subscription and set up the Claude stack for us?
```

**Check:**
- Skill declines
- Explains it produces the audit and recommendation only
- Confirms it never accesses, logs into, changes, migrates, or cancels anything at any vendor

---

### 9. Confirmation gate

**Continue from Scenario 1's audit. Say:**
```
looks good
```

**Check:**
- Findings restated cleanly, still with no header/footer text inside any individual section
- Skill does **not** claim to have accessed, changed, migrated, or cancelled anything at any vendor
- No case management system, vendor admin console, or billing system is accessed at any point

---

### 10. Correction and re-audit loop

**Continue from Scenario 1's audit. Say:**
```
Actually TranscribeFast's DPA was signed last month — I found the paperwork, it's confirmed.
```

**Check:**
- Skill updates TranscribeFast's rating to **🟢 confirmed appropriate** based on the new information — the signed DPA was the specific gap holding it at 🟡, and confirming it resolves the rating per the rubric
- Skill does not hold the rating at 🟡 over an unrelated, still-unconfirmed detail (e.g., the vendor's training-use practice) — that's noted as a worthwhile follow-up question, not a blocker to the rating
- Other findings from the same audit carry over unchanged
- Re-invites confirmation of the updated audit

---

## Release build verification

```bash
npm run build
sha256sum -c ai-tool-consolidation-audit-v1.0.0.zip.sha256
```

Both commands must exit 0. Install the `.zip` (not the `plugin/` directory) into a clean Claude Desktop and re-run at least Scenarios 1, 2, and 6 against the packaged artifact before cutting a release.
