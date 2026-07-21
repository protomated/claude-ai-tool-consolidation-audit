# Court Deadline Reasoning & Calendar Drafting v1.0.0

Initial release.

## What's included

### `/court-deadline` — Court Deadline Reasoning & Calendar Drafting

A step-by-step deadline calculator for solo and small-firm attorneys:

- **Trigger date + rule in plain English:** supply the applicable procedural rule; the skill applies it. No jurisdiction-wide rule database — you own the rule; it does the arithmetic.
- **Auditable reasoning chain:** every computation is shown step by step — counting anchor, day type, weekend and federal holiday exclusions, rollover check — so you can verify the logic, not just the result.
- **Ambiguity resolution before computing:** if the rule leaves anything implicit (day type, counting anchor, rollover behavior, holiday scope), the skill asks before computing rather than guessing.
- **Calendar event drafting:** after computing the deadline, the skill shows a complete Google Calendar event draft and creates it only after you confirm.

Handles: service-response windows, appeal periods, statute-of-limitations landmarks, summary-judgment deadlines, discovery cutoffs, and any one-off date logic where showing the work matters.

## Setup

Install time: approximately 5 minutes. Connect the Google Calendar connector once in Claude Desktop → Settings → Connectors → Google Calendar, sign in with your Google account, and authorize calendar access. See `plugin/CONNECTORS.md` for step-by-step instructions.

## Compliance

Requires Claude for Work, Claude Team, or Claude Enterprise. Do not use a consumer Claude plan (Claude Pro or Personal) with confidential matter information. Every output carries a "NOT A SUBSTITUTE FOR DOCKETING SOFTWARE" header. The skill never creates calendar events without your explicit in-conversation confirmation. Computed deadlines must be verified independently before reliance.
