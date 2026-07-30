# Demand Letter & Client Correspondence Drafter for Law Firms — Claude Desktop Plugin

A Claude Desktop / Cowork plugin that drafts a first-pass demand letter from your firm's own template and case facts, or a plain-English client status-update email — for solo and small-firm attorneys who draft near-identical correspondence by hand, matter after matter.

**Distributed by [Protomated](https://protomated.com) as a free download.**

---

## ⚠️ Required: Read This Before You Install

**This section is not boilerplate. Read it before attaching any case files.**

### 1. You must be on a qualifying Claude plan

Do NOT use this plugin on a consumer Claude plan (claude.ai Personal or Claude Pro) with any confidential matter or client information. Consumer plans do not provide a Data Processing Agreement (DPA) covering privileged content.

Use one of the following:

- **Claude for Work** (formerly Claude.ai Teams)
- **Claude Team or Enterprise**
- **Claude API** (with a signed DPA from Anthropic)

> **If you're not sure which plan you're on:** Open Claude Desktop → Help → About. If it says "Claude Pro," you are on a consumer plan. Upgrade to Claude for Work before attaching any confidential case files.

### 2. Every draft is a first pass — you are the author of record

This plugin drafts from the case facts and template you provide. It does not verify accuracy, completeness, or legal sufficiency, and it never sets a demand amount, apportions liability, or reaches a legal conclusion. You review every draft, fill in the demand figure yourself, and confirm it's accurate before sending.

### 3. This plugin does not send anything

The plugin reads only the workspace folder you explicitly attach, and it never emails, files, submits, or transmits a letter anywhere. You copy the final draft and send it yourself.

---

## Installation (about 5 minutes)

### Step 1 — Download and install

1. Download `demand-letter-drafter.zip` from the [Releases page](https://github.com/protomated/claude-demand-letter-drafter/releases).
2. Double-click the `.zip` file, or drag it into Claude Desktop's **Extensions** panel.
3. Claude Desktop will install the plugin.

No connectors to authorize. No credentials to configure.

### Step 2 — Attach a case folder

Before running the skill, attach a workspace folder containing:
- Your case facts (intake notes, correspondence, treatment summaries, incident reports — whatever you have)
- Your firm's demand-letter template, if you're drafting a demand letter

If you skip this, the skill will ask you to attach a folder or paste the facts directly.

### Step 3 — Verify

Open a new Claude Desktop chat, attach your folder, and type `/skills`. You should see `/demand-letter` listed. Run `/demand-letter` to start.

---

## The Skill

### `/demand-letter` — Demand Letter & Client Correspondence Drafter

Drafts one of two things from your attached case folder:

1. **A first-pass demand letter** — your firm's template, populated with the case facts you provided.
2. **A plain-English client status-update email** — where the matter stands, in language a client can follow.

The skill asks which one you want if you don't say, asks for your template if it can't find one, and asks before drafting if the facts on hand aren't enough — it never fills a gap with a plausible-sounding guess.

**What you supply:**
- Case facts, via an attached folder or pasted directly
- Your firm's demand-letter template, for demand letters
- The demand amount, when you're ready to add it — the skill leaves a placeholder rather than suggesting a figure

**What it produces:**
- A ready-to-review demand letter or client status-update email, in a clean copy-only block with no Protomated branding inside it
- A placeholder for anything requiring your judgment (demand amount, liability position)

**What it does not do:**
- It does not set a demand amount, apportion liability, or reach a legal conclusion — that's yours to decide
- It does not send, file, or submit anything — you send it yourself
- It does not invent facts, treatment details, or damages figures not in your case folder

**Example inputs:**

```
/demand-letter demand letter
/demand-letter status update
/demand-letter
```

**Typical use time:** a few minutes per draft once your case folder and template are attached.
**Setup:** about 5 minutes (install plugin, attach case folder and template).

---

## Testing guide

Run these inputs to verify the plugin is working correctly. Use synthetic or anonymized matter details, and attach a test folder with sample intake notes and a sample demand-letter template.

1. **Demand letter with template and full facts** — attach a folder with a template and complete facts, run `/demand-letter demand letter` → expect: draft follows the template structure, cites only facts present in the folder, and leaves `[DEMAND AMOUNT — attorney to set]` rather than a figure
2. **Demand letter with no template attached** — attach a folder with facts but no template, run `/demand-letter demand letter` → expect: skill asks whether a template exists rather than generating a generic structure
3. **Demand letter with sparse damages facts** — attach a folder with minimal treatment/damages detail → expect: skill asks what to include rather than inferring plausible treatment details
4. **Client status-update email** — run `/demand-letter status update` → expect: plain-English draft, no legal jargon, structured as what's happened / what's next / any client action needed
5. **Output type not specified** — run `/demand-letter` with no argument → expect: skill asks whether this is a demand letter or a status update before doing anything else
6. **Attorney requests a demand figure** — after a draft, say "what should I demand?" → expect: skill declines, explains it's the attorney's call, leaves the placeholder in place
7. **Attorney requests a liability assessment** — ask "who's at fault here?" → expect: skill declines to draw a legal conclusion
8. **No folder attached** — run `/demand-letter` with nothing attached → expect: skill asks the attorney to attach a folder or paste the facts directly
9. **Edit and revise loop** — after a draft, say "more formal" → expect: revised draft, same facts, re-invites confirmation
10. **Confirmation gate** — after any draft, say "looks good" → expect: final draft restated cleanly with no header/footer text inside the copy block, skill does not send anything, offers to draft the other correspondence type for the same matter

---

### Release build verification

```bash
npm run build
sha256sum -c demand-letter-drafter-v1.0.0.zip.sha256
```

Both commands must exit 0. Install the `.zip` (not the `plugin/` directory) into a clean Claude Desktop to confirm the packaged artifact works end to end.

---

## Why This Matters

Poor or slow client communication is the single largest category of bar grievances against attorneys. Personal-injury and general-practice attorneys draft near-identical demand letters and status updates matter after matter — the template and the facts rarely change shape, only the details. This plugin closes the gap between "I know what needs to go in this letter" and "I have a first pass in front of me," without touching the two decisions that are actually yours: what to demand, and who's at fault.

---

## Want the Full Matter-Milestone Pipeline?

This plugin still requires you to attach your case folder and review every draft by hand. Protomated can build a system that drafts and queues client updates automatically as your matter's status changes in your practice management system — a Quick-Win Build scoped to your firm's workflow.

[Book a 30-minute call →](https://protomated.com/call)

---

## License

MIT. See [LICENSE](LICENSE).

## Feedback and Issues

[GitHub Issues](https://github.com/protomated/claude-demand-letter-drafter/issues) | [hello@protomated.com](mailto:hello@protomated.com)
