---
name: court-deadline
description: Compute a court or filing deadline from a trigger date and the applicable rule you supply in plain English. Shows step-by-step reasoning so every calculation is auditable. Then offers to draft a calendar event. Use for service-response windows, appeal periods, statute-of-limitations landmarks, summary-judgment deadlines, discovery cutoffs, and any one-off or complex date logic where you need to see the work — not just the answer.
argument-hint: "[optional: trigger date + rule, e.g. 'served June 15 2026, responsive pleading 21 calendar days after']"
---

# /court-deadline — Court Deadline Reasoning & Calendar Drafting

> ⚠️ NOT A SUBSTITUTE FOR DOCKETING SOFTWARE
> This skill computes deadlines from the rule you provide. It does not know your jurisdiction's rules, local court rules, or standing orders. Verify every computed date independently before relying on it. Not legal advice.

Provide a trigger date and the applicable rule in plain English. The skill reasons through the computation step by step — showing every decision in the chain — and states the deadline. Then it offers to draft a calendar event.

**This skill does not maintain a rule database.** You supply the rule each time. It applies it.

## Invocation

```
/court-deadline
/court-deadline served June 15 2026, responsive pleading 21 calendar days after service
```

---

## Workflow

### Step 1 — Collect inputs

Ask for the following. Accept inputs supplied as arguments; ask for anything missing.

**Required:**
- **Trigger date** — the event the deadline runs from (service date, filing date, judgment date, entry of order, etc.). Ask for the date and what it represents.
- **Rule** — the applicable deadline rule in plain English. Example: "21 calendar days after service; if the deadline falls on a weekend or federal holiday, the next business day applies." If the attorney is unsure how to phrase the rule, ask: "What does the applicable procedural rule say about when this is due?"

**Optional (include in the calendar event if provided):**
- Matter name or number
- Court name
- Opposing party
- Event type (e.g., "Responsive Pleading," "Notice of Appeal," "Summary Judgment Opposition")

---

### Step 2 — Resolve ambiguity before computing

Before performing any calculation, check the rule for the following ambiguities. If any are present, ask the attorney to clarify. Do not assume. One question at a time.

**Day type:** If the rule says "days" without specifying "calendar" or "business," ask:
> "Does this rule count calendar days (every day including weekends and holidays) or business days (excluding weekends and applicable holidays)?"

**Counting anchor:** If the rule says "after" or "from" without specifying whether the trigger date is included:
> "Is [trigger date] Day 0 (excluded — counting begins the following day) or Day 1 (included in the count)? Federal rules generally treat the trigger date as Day 0, but local rules vary."

**Holiday scope:** If the rule excludes holidays without specifying which:
> "Does this rule exclude federal holidays only, or also state or local holidays? If state or local holidays apply, please supply the relevant dates — I know U.S. federal holidays but not state or local ones."

**Rollover:** If the rule does not address what happens when the deadline falls on a weekend or holiday:
> "If the computed deadline falls on a weekend or holiday, does it roll to the next business day, the preceding business day, or stay fixed on that date?"

**Month arithmetic:** If the rule uses "month" or similar:
> "Does 'one month' mean the same date in the following calendar month, or 30 calendar days? For example, one month from January 31 could be February 28/29 or March 2, depending on interpretation."

---

### Step 3 — Compute the deadline step by step

Perform the computation in clearly numbered steps. Every step must be shown explicitly. The reasoning chain is the primary deliverable — not a footnote.

**Standard computation structure:**

```
DEADLINE COMPUTATION — REASONING CHAIN

Trigger event: [what happened and on what date]
Rule applied:  [the rule as the attorney stated it; note any assumptions made]

Step 1 — Anchor date
[State the trigger date and whether it is Day 0 or Day 1. Identify which day counting begins.]

Step 2 — Count [N] [calendar / business] days
[Calendar days: compute the endpoint arithmetically. For counts ≤ 14, you may enumerate each day. For larger counts, compute directly and state the method.]
[Business days: identify each weekend and holiday that extends the count and name any federal holidays skipped.]
Raw deadline (before rollover check): [date] ([day of week])

Step 3 — Weekend / holiday check
[State what day of the week the raw deadline falls on.]
[State whether it is a federal holiday — name the holiday if so.]
[Apply the rollover rule and show the result, or confirm no rollover is needed.]
[If rolled: confirm the new date is itself not a weekend or holiday.]

DEADLINE: [Final date] ([Day of week])
```

**Month/year arithmetic:** Show the calendar-month math explicitly — trigger month and day, add specified months, note any end-of-month adjustment (e.g., one month from January 31 = February 28/29; state the rule you applied).

**Multiple deadlines:** If the rule produces more than one deadline (e.g., 30 days to file, 14 days to reply to opposition), compute each in its own numbered block and label them clearly.

---

### Step 4 — Present the deadline

Present the result in this exact format:

```
⚠️ NOT A SUBSTITUTE FOR DOCKETING SOFTWARE
This deadline was computed from the rule you provided. It does not reflect jurisdiction-specific rules, local court rules, or standing orders you did not supply. Verify this date independently before relying on it. Not legal advice.

---

DEADLINE COMPUTATION — REASONING CHAIN

[Full step-by-step reasoning chain from Step 3]

---

RESULT

Deadline:  [Final date] ([Day of week])
Event:     [What this deadline is for]
[Matter / Court / Opposing Party if provided]

---

— Prepared with Protomated Court Deadline Reasoning (Claude Desktop) | Verify independently before use | Not legal advice
```

---

### Step 5 — Offer to draft a calendar event

After presenting the deadline, ask:

> "Would you like me to draft a calendar event for this deadline? I'll show you the full event details before creating anything."

If the attorney says yes, draft the following and show it completely before taking any action:

```
Calendar event draft:

Title:       [Event type] — [Matter name or opposing party]
             Example: "Responsive Pleading Deadline — Smith v. Jones"
Date:        [Computed deadline date]
Time:        [Ask if not specified; suggest 9:00 AM as a default]
Description:
  Matter:    [Matter name or number, if provided]
  Court:     [Court name, if provided]
  Deadline:  [Computed deadline date] ([Day of week])
  Trigger:   [What event started the clock and on what date]
  Rule:      [Rule as attorney stated it]
  Source:    Protomated Court Deadline Reasoning (Claude Desktop)
  ⚠️ Verify this date independently. Not a substitute for docketing software.

Shall I create this event in your Google Calendar? (yes / no)
```

Do not create the event until the attorney explicitly confirms.

---

### Step 6 — Create the calendar event (only after confirmation)

If the attorney confirms, create the calendar event via the Google Calendar connector with the details shown in Step 5.

After creating the event, confirm:
> "Calendar event created: '[Event title]' on [date]. Check your Google Calendar to confirm it appears correctly. Remember to verify the deadline date independently."

If the attorney declines, close with:
> "No event created. The computed deadline is [final date]. Verify it independently before relying on it."

---

## What This Skill Does Not Do

- It does not know your jurisdiction's court rules, local rules, standing orders, or state procedural requirements. You supply the applicable rule; it applies it.
- It does not interpret whether a particular rule applies to your matter. The attorney confirms applicability.
- It does not create calendar events without your explicit confirmation after reviewing the event details.
- It does not know state or local holidays. If the rule references them, supply those dates.
- It does not produce a verified docket entry or replace a docketing system. Every computed date requires your independent verification.
- It does not provide legal advice.

---

— Prepared with Protomated Court Deadline Reasoning (Claude Desktop) | Verify independently before use | Not legal advice
