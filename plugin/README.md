# Billing Narrative & Time-Entry Drafter for Law Firms — Claude Desktop Plugin

A Claude Desktop plugin that converts rough time-entry notes into professional billing narratives with suggested time increments — for solo and small-firm attorneys who lose billable hours because writing the entry after the fact is too slow.

**Distributed by [Protomated](https://protomated.com) as a free download.**

---

## ⚠️ Required: Read This Before You Install

**This section is not boilerplate. Read it before entering any matter information.**

### 1. You must be on a qualifying Claude plan

Do NOT use this plugin on a consumer Claude plan (claude.ai Personal or Claude Pro) with any confidential matter or client information. Consumer plans do not provide a Data Processing Agreement (DPA) covering privileged content.

Use one of the following:

- **Claude for Work** (formerly Claude.ai Teams)
- **Claude Team or Enterprise**
- **Claude API** (with a signed DPA from Anthropic)

> **If you're not sure which plan you're on:** Open Claude Desktop → Help → About. If it says "Claude Pro," you are on a consumer plan. Upgrade to Claude for Work before entering any confidential matter details.

### 2. Every narrative is a draft — you confirm before billing

This plugin drafts from the notes you provide. It does not verify accuracy, completeness, or billing compliance. You review every entry and confirm it is correct before pasting it into your billing system.

### 3. This plugin does not connect to your billing system

The plugin operates entirely within your Claude Desktop conversation. It never submits, records, or transmits entries anywhere. You copy the final narrative and paste it into Clio, MyCase, PracticePanther, or wherever you bill.

---

## Installation (under 3 minutes)

### Step 1 — Download and install

1. Download `billing-narrative-drafter.zip` from the [Releases page](https://github.com/protomated/claude-billing-narrative-drafter/releases).
2. Double-click the `.zip` file, or drag it into Claude Desktop's **Extensions** panel.
3. Claude Desktop will install the plugin.

No connectors to authorize. No credentials to configure.

### Step 2 — Verify

Open a new Claude Desktop chat. Type `/skills`. You should see `/billing-narrative` listed. Run `/billing-narrative` to start.

---

## The Skill

### `/billing-narrative` — Billing Narrative & Time-Entry Drafter

Paste your rough notes. The skill asks one clarifying question at a time if anything is unclear, then drafts a clean, professional billing narrative with a suggested time increment. You review, edit if needed, and paste into your billing system.

**What you supply:**
- Rough notes — abbreviations, shorthand, and fragments are fine
- Optionally: matter name, date, time estimate, billing increment style (0.1 hr or 0.25 hr), UTBMS code preference

**What it produces:**
- A ready-to-paste billing narrative in professional past-tense billing language
- A suggested time increment (rounded to your increment style)
- UTBMS/ABA task and activity codes if you want them

**What it does not do:**
- It does not submit entries to any billing system — you paste it yourself
- It does not invent facts not in your notes
- It does not verify the accuracy of your time record

**Example inputs:**

```
/billing-narrative tc w client 30 min re PI settlement, reviewed demand, advised to counter at 85k
/billing-narrative drafted MSJ, reviewed 3 supporting cases, added argument re proximate cause
/billing-narrative attended scheduling conference Judge Smith, Smith v Jones, approx 45 min
/billing-narrative responded to 4 emails from opp counsel re discovery schedule and doc production
```

**Typical use time:** under 2 minutes per entry once you have your notes in hand.
**Setup:** under 3 minutes.

---

## Testing guide

Run these inputs to verify the plugin is working correctly. Use synthetic or anonymized matter details.

1. **Basic conference call** — paste: `tc w client 30 min re PI settlement, reviewed demand letter, advised to counter` → expect: professional narrative, 0.5 hr suggestion, ask about increment style first session
2. **Email exchange** — paste: `responded to 3 emails from opp counsel re discovery schedule` → expect: narrative leading with "Corresponded with opposing counsel," time suggestion ~0.3 hr
3. **Document drafting** — paste: `drafted motion for summary judgment, incorporated 3 cases, added proximate cause argument` → expect: specific narrative naming the motion and work done, time suggestion 1.0–2.0 hr depending on context
4. **Court appearance** — paste: `attended scheduling conference, Judge Smith, Smith v Jones, 45 min` → expect: narrative with "Attended scheduling conference," 0.8 hr (nearest 0.1), or asks increment style
5. **Research session** — paste: `researched TX statute of limitations for negligence claims, reviewed 2 cases, drafted memo section` → expect: narrative naming the research and output, flags time as estimate
6. **Ambiguous notes (skill asks, does not guess)** — paste: `worked on Smith file` → expect: skill asks what was done before drafting, does not invent activity
7. **UTBMS codes** — paste same as #1, answer "yes" to UTBMS question → expect: L160 + A106 or equivalent suggested, with explanation
8. **Multi-activity entry** — paste: `reviewed contract, drafted client email summary, then researched indemnity clause law` → expect: skill asks whether to combine or split into separate entries
9. **Edit loop** — after first draft, say "shorter" → expect: revised narrative, same time suggestion, re-invites confirmation
10. **Confirmation gate** — after any draft, say "looks good" → expect: final narrative restated cleanly as ready-to-paste, offer to draft another entry

---

## Why This Matters

Research consistently shows that 14% of billable hours go unrecorded — most of it because writing the narrative after the fact is slow, tedious, and gets skipped when attorneys are busy. For a solo attorney billing 1,200 hours a year at $300/hr, that is $50,000 left on the table annually. For a firm doing $1M in fees, it is closer to $140,000.

The bottleneck is the narrative, not the time. Attorneys know what they did. They just don't have a fast way to turn that into billing-appropriate language. This plugin closes that gap in under 2 minutes per entry.

---

## Want a System That Captures Time Automatically?

This plugin still requires you to paste your notes. Protomated can build a custom time-capture system that pulls from your calendar and email automatically, drafts narratives in bulk, and syncs directly to your billing software — $5,000–$15,000 depending on scope.

[Book a 30-minute call →](https://protomated.com/call)

---

## License

MIT. See [LICENSE](LICENSE).

## Feedback and Issues

[GitHub Issues](https://github.com/protomated/claude-billing-narrative-drafter/issues) | [hello@protomated.com](mailto:hello@protomated.com)
