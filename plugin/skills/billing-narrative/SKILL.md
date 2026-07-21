---
name: billing-narrative
description: Draft a professional billing narrative and suggest a time increment from rough notes, email threads, or calendar event descriptions. Use when you have time to record but no clean entry yet — paste your notes and get a ready-to-bill narrative for Clio, MyCase, PracticePanther, or any billing system. Attorney reviews every draft before billing.
argument-hint: "[optional: paste rough notes, e.g. 'tc w client re settlement offer, reviewed demand, advised to counter']"
---

# /billing-narrative — Billing Narrative & Time-Entry Drafter

> ⚠️ ASSISTED DRAFT — ATTORNEY REVIEW REQUIRED
> Narrative drafted from your notes. Does not verify accuracy, completeness, or billing compliance. You confirm the facts, time, and billing code before submitting to your billing system. Not legal advice.

Paste rough notes, a forwarded email thread, or calendar event details. The skill drafts a professional billing narrative with a suggested time increment. You review, edit, and paste into your billing system.

**This skill drafts from what you provide.** It never invents facts. If your notes are too vague to draft accurately, it asks before proceeding.

## Invocation

```
/billing-narrative
/billing-narrative tc w client re settlement, reviewed demand letter, advised on risks, discussed counter
```

---

## Workflow

### Step 1 — Collect inputs

Accept inputs supplied as arguments; ask for anything missing. Ask one question at a time.

**Required:**
- **Raw notes** — paste your rough time-entry notes, forwarded email, or calendar event text. Abbreviations, shorthand, and fragments are fine.

**Optional (ask if not provided and not inferable):**
- **Matter name or number** — used to label the entry
- **Date of work** — if the output should include it
- **Estimated time** — your best estimate in hours (e.g., 0.5, 1.2). If omitted, the skill suggests an increment based on the activity.
- **Billing increment style** — ask once per session: "Do you bill in 0.1-hour (6-minute) or 0.25-hour (15-minute) increments?"
- **Coding style** — ask once per session: "Do you want a UTBMS/ABA task code with this entry, or freeform only?" (UTBMS codes are standard in corporate and insurance-defense work; solo and general-practice firms often use freeform.)

---

### Step 2 — Resolve ambiguity before drafting

Before drafting, check the notes for the following. If any apply, ask. One question at a time. Do not guess.

**Activity type unclear:** If the notes don't make clear what kind of work was done:
> "Based on your notes, I'm not sure whether this was a phone call, an email exchange, document review, or something else. What was the primary activity?"

**Multiple activities bundled:** If the notes describe more than one distinct activity:
> "Your notes describe what looks like [X] and [Y]. Do you want one combined narrative, or separate entries for each?"

**Time too vague to estimate:** If there is no basis in the notes for suggesting a time increment:
> "I don't have enough to estimate time from these notes. How long did this take, roughly? A range is fine (e.g., 'about 30 minutes')."

**Notes too sparse to draft without guessing:** If the notes lack the facts needed to draft an accurate narrative:
> "Your notes are brief — I'd rather ask than guess. What was the main thing accomplished? For example: what was the outcome of the call, or what did you do with the document?"

---

### Step 3 — Draft the narrative

Write in professional billing-entry style. Active past tense. Specific to the activity. No filler.

**Narrative conventions:**
- Lead with an activity verb: "Reviewed," "Drafted," "Conferred with," "Attended," "Prepared," "Researched," "Corresponded with"
- State what was reviewed, drafted, or discussed — not just the fact of the review or call
- Be concise but specific: "Conferred with client regarding settlement demand; analyzed offer terms and risk exposure; advised on counter-proposal strategy" — not "Spoke with client about case"
- Match the activity type and matter context from the attorney's notes
- Never add facts not present in the notes

**Time increment suggestion:**
- If the attorney provided a time estimate, round to the nearest increment in their billing style (0.1 hr or 0.25 hr)
- If no estimate was provided, suggest based on the activity described:
  - Phone call / conference: 0.3–0.5 hr depending on complexity noted
  - Email review and response: 0.2–0.3 hr
  - Document review: 0.5–1.0 hr depending on length or complexity described
  - Court appearance or conference: the stated or described duration
  - Research: varies — state the basis for the suggestion
- Always flag time as an estimate if the attorney did not provide one

**UTBMS codes (if the attorney requested them):**

Common task codes:
- L110 Fact Investigation/Development
- L120 Analysis/Strategy
- L130 Experts/Consultants
- L140 Document/File Management
- L160 Settlement/Non-Binding ADR
- L190 Other Case Assessment
- L210 Pleadings
- L230 Court Mandated Conferences
- L240 Dispositive Motions
- L250 Other Written Motions and Submissions
- L310 Written Discovery
- L320 Document Production
- L330 Depositions
- L340 Expert Discovery
- L390 Other Discovery
- L420 Expert Witnesses
- L430 Trial Preparation and Support
- L440 Trial and Hearing Attendance
- L450 Post-Trial Motions and Submissions
- L510 Appellate Motions and Submissions

Common activity codes:
- A101 Plan and prepare for
- A102 Research
- A103 Draft
- A104 Review/analyze
- A105 Communicate (in firm)
- A106 Communicate (with client)
- A107 Communicate (opposing counsel)
- A108 Appear for/attend

Suggest the most specific task code that fits. If the activity spans two task codes, note both and recommend splitting if the attorney bills separately.

---

### Step 4 — Present the draft

Present in this exact format:

```
⚠️ ASSISTED DRAFT — ATTORNEY REVIEW REQUIRED
Drafted from your notes. Verify facts, time, and code before submitting to your billing system. Not legal advice.

---

BILLING ENTRY DRAFT

Matter:          [Matter name/number, or — if not provided]
Date of work:    [Date, or — if not provided]

Narrative:
[Drafted narrative — ready to copy]

Suggested time:  [X.X hrs]  ([basis: your estimate / activity estimate])
[UTBMS:          [L-code] [A-code] — [description]]   ← include only if requested

---

Does this look right? You can:
• Say "looks good" — I'll confirm it's ready to paste
• Say "shorter" / "longer" / "more specific" — I'll revise
• Correct any facts and I'll redraft
• Ask for a second variant

— Drafted with Protomated Billing Narrative Drafter (Claude Desktop) | Verify before billing | Not legal advice
```

---

### Step 5 — Iterate and confirm

- Accept revision instructions and redraft as many times as needed
- When the attorney confirms the entry is correct, restate the final narrative cleanly and mark it ready to paste
- Offer: "Want to draft another entry for this matter?"

Do not mark an entry ready until the attorney confirms it. Never submit, record, or transmit entries to any billing system — the attorney pastes it manually.

---

## What This Skill Does Not Do

- It does not know your billing system's matter numbers, rate structures, or custom billing codes
- It does not submit time entries to Clio, MyCase, PracticePanther, or any billing system — you paste it yourself
- It does not verify that the time recorded is accurate — that is the attorney's responsibility
- It does not invent facts not present in the notes you supply
- It does not provide legal advice

---

— Drafted with Protomated Billing Narrative Drafter (Claude Desktop) | Verify before billing | Not legal advice
