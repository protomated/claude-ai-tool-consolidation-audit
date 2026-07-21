# Court Deadline Reasoning & Calendar Drafting for Law Firms — Claude Desktop Plugin

A Claude Desktop plugin that computes court deadlines from the rule you supply, shows its step-by-step reasoning so every calculation is auditable, and offers to draft a calendar event — for solo and small-firm attorneys handling one-off or complex date logic.

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

### 2. This is a reasoning tool, not a docketing system

This plugin computes deadlines from the rule you provide. It does not know your jurisdiction's procedural rules, local court rules, or standing orders. You supply the applicable rule each time; it applies it. Verify every computed date independently before relying on it.

**Use docketing software for ongoing deadline management.** This plugin is for one-off and complex date logic — situations where you need to see the reasoning, not just the answer.

### 3. Calendar events require your explicit confirmation

The plugin will not create any calendar event without showing you the full event details first and receiving your explicit in-conversation confirmation.

---

## Installation (under 5 minutes)

### Step 1 — Download and install

1. Download `court-deadline-reasoning.zip` from the [Releases page](https://github.com/protomated/claude-court-deadline-reasoning/releases).
2. Double-click the `.zip` file, or drag it into Claude Desktop's **Extensions** panel.
3. Claude Desktop will install the plugin and prompt you to connect the required connector.

### Step 2 — Connect Google Calendar

1. Go to **Claude Desktop → Settings → Connectors**.
2. Find **Google Calendar** and click **Connect**.
3. Sign in with the Google account that holds your firm's calendar.
4. Authorize calendar access when prompted.

> **Tip:** Use a firm Google Workspace account rather than a personal Gmail if your firm runs on Google Workspace. That keeps deadline events inside your firm calendar.

### Step 3 — Verify

Open a new Claude Desktop chat. Type `/skills`. You should see `/court-deadline` listed. Run `/court-deadline` to start.

See [CONNECTORS.md](CONNECTORS.md) for troubleshooting.

---

## The Skill

### `/court-deadline` — Court Deadline Reasoning & Calendar Drafting

Provide a trigger date and the applicable rule in plain English. The skill computes the deadline with a full step-by-step reasoning chain, then offers to draft a calendar event.

**What it handles:**
- Calendar-day counts (21 days after service, 30 days from entry of judgment)
- Business-day counts (15 business days, excluding weekends and federal holidays)
- Week and month arithmetic (4 weeks from filing, 6 months from accrual)
- Federal holiday exclusions (built-in U.S. federal holiday calendar)
- Next-business-day rollover when deadlines land on weekends or holidays
- Multiple related deadlines from a single rule set

**What you supply each time:**
- The trigger date and what it represents
- The applicable rule in plain English
- Optionally: matter name, court name, opposing party, event type

**What it does not do:**
- It does not know your jurisdiction's rules — you supply the rule
- It does not know state or local holidays — supply those dates if the rule references them
- It does not create calendar events without your confirmation

```
/court-deadline
/court-deadline served June 15 2026, responsive pleading 21 calendar days after service
```

**Typical use time:** under 2 minutes once you have the rule in hand.
**Setup:** under 5 minutes.

---

## Why This Matters

Missed deadlines are the leading cause of legal malpractice claims — approximately 24.6% of all claims. Yet only 27% of solo practitioners use dedicated docketing software. The gap shows up most in one-off situations: an unusual service method, a complex statutory period, a deadline that chains off another deadline. That is where this plugin helps.

The differentiator from a spreadsheet formula is the reasoning trace. You can read every step, verify every exclusion, and catch any error before it becomes a missed deadline.

---

## Want a System That Knows Your Rules?

This plugin requires you to supply the rule each time. Protomated can build a custom deadline management system integrated with your case management software, pre-loaded with the rule sets for every jurisdiction you file in — $5,000–$15,000 depending on scope.

[Book a 30-minute call →](https://protomated.com/call)

---

## License

MIT. See [LICENSE](LICENSE).

## Feedback and Issues

[GitHub Issues](https://github.com/protomated/claude-court-deadline-reasoning/issues) | [hello@protomated.com](mailto:hello@protomated.com)
