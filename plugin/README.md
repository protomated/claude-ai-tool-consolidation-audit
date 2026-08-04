# Contract & Document Review Skill for Law Firms — Claude Desktop Plugin

A Claude Desktop / Cowork plugin that reviews a contract clause by clause against a configurable playbook — flagging each clause GREEN, YELLOW, or RED with plain-English rationale and suggested redline language — using your firm's own playbook, or a generic clause playbook if you haven't attached your own, for solo and small-firm attorneys who review contracts occasionally and can't justify a dedicated $99–400/mo contract-review tool.

**Distributed by [Protomated](https://protomated.com) as a free download.**

---

## ⚠️ Required: Read This Before You Install

**This section is not boilerplate. Read it before attaching any contract.**

### 1. You must be on a qualifying Claude plan

Do NOT use this plugin on a consumer Claude plan (claude.ai Personal or Claude Pro) with any confidential client or contract information. Consumer plans do not provide a Data Processing Agreement (DPA) covering privileged content.

Use one of the following:

- **Claude for Work** (formerly Claude.ai Teams)
- **Claude Team or Enterprise**
- **Claude API** (with a signed DPA from Anthropic)

> **If you're not sure which plan you're on:** Open Claude Desktop → Help → About. If it says "Claude Pro," you are on a consumer plan. Upgrade to Claude for Work before attaching any confidential contract.

### 2. Every review is a first pass — you are responsible for every position

This plugin compares contract language to playbook positions and reports the result. It does not verify enforceability under any governing law, does not know your deal's business context beyond what's in the contract, and never decides whether to sign, negotiate, or reject anything. You review every rating and redline before using this review in negotiation.

### 3. FREE-tier playbook is generic, not your firm's positions

If you don't attach your own `playbook.md`, the plugin uses its own bundled generic playbook — a conservative, general-purpose starting point for common commercial clause types, not this firm's actual negotiation positions. Treat every rating from it as something to confirm, not a final answer.

### 4. This plugin does not edit your document or send anything

The plugin reads only the workspace folder you explicitly attach. It cannot edit or generate a `.docx` file, so it never applies a Word tracked change — every suggested redline is chat text you copy into your own document. It never sends, files, or transmits your contract or the review to anyone.

---

## Installation (about 5 minutes)

### Step 1 — Download and install

1. Download `contract-document-reviewer.zip` from the [Releases page](https://github.com/protomated/claude-contract-document-reviewer/releases).
2. Double-click the `.zip` file, or drag it into Claude Desktop's **Extensions** panel.
3. Claude Desktop will install the plugin.

No connectors to authorize. No credentials to configure.

### Step 2 — Attach a contract (and, optionally, your firm's playbook)

Before running the skill, attach a workspace folder containing:
- The contract you want reviewed
- Your firm's own `playbook.md`, if you have one — skip this and the plugin will use its own bundled generic playbook instead, clearly labeled as generic

### Step 3 — Verify

Open a new Claude Desktop chat, attach your folder, and type `/skills`. You should see `/contract-review` listed. Run `/contract-review` to start.

---

## The Skill

### `/contract-review` — Contract & Document Review

Reviews an attached contract clause by clause against a playbook:

1. Matches each clause to the applicable playbook position
2. Rates it **GREEN** (meets the playbook position), **YELLOW** (within an acceptable fallback range), **RED** (conflicts with a must-have or trips a red-flag trigger), or **UNRATED** (not covered by the playbook in use)
3. Suggests redline language for anything that isn't GREEN

**What you supply:**
- The contract, via an attached folder or pasted directly
- Your firm's own `playbook.md`, if you have one — the skill uses its bundled generic playbook (clearly labeled) if you don't
- Anything requiring your judgment — the skill leaves a clause UNRATED rather than guessing

**What it produces:**
- A clause-by-clause review with plain-English rationale tied to specific playbook positions
- Suggested redline language for YELLOW and RED clauses, as copy-ready chat text
- A clear UNRATED flag for anything the playbook in use doesn't cover

**What it does not do:**
- It does not decide whether to sign, negotiate, or reject the contract — that's yours to decide
- It does not determine enforceability under any governing law — verify this independently
- It does not edit or generate a `.docx` file, or apply a Word tracked change — you apply every redline yourself
- It does not send, file, or transmit the contract or the review to anyone
- It does not invent contract language or firm positions not present in what you provide

**Example inputs:**

```
/contract-review
/contract-review vendor-services-agreement.docx
```

**Typical use time:** a few minutes per contract once it's attached and, if you have one, your firm's playbook is attached alongside it.
**Setup:** about 5 minutes (install plugin, attach a contract).

---

## Testing guide

Run these inputs to verify the plugin is working correctly. Use synthetic or anonymized contract details, and attach a test folder with a sample contract and, optionally, a sample firm playbook.

1. **Full review, firm playbook attached** — attach a folder with a contract and a firm `playbook.md`, run `/contract-review` → expect: every rating is tied to a specific entry in the attached playbook, not the bundled generic one
2. **No firm playbook attached** — attach a folder with just the contract, run `/contract-review` → expect: skill uses the bundled generic playbook and says plainly that it's generic, not this firm's positions — it does not ask you to supply a playbook first
3. **Clause type not covered by playbook** — include a clause type the playbook doesn't address → expect: skill marks it UNRATED and says so, rather than guessing GREEN/YELLOW/RED
4. **More than one contract attached** — attach two contracts, run `/contract-review` with no argument → expect: skill asks which one to review, or whether to review both, before starting
5. **Attorney asks whether to sign** — after a review, ask "should we sign this?" → expect: skill declines to decide, explains it's the attorney's call
6. **Attorney asks about enforceability** — ask "is this limitation of liability clause enforceable in my state?" → expect: skill declines to give a definitive answer, points to the rating already given as a starting point
7. **No contract attached** — run `/contract-review` with nothing attached → expect: skill asks the attorney to attach the contract or paste its text
8. **Edit and revise loop** — after a review, say "re-check the termination clause" → expect: skill re-reviews just that clause without asking for unrelated new inputs, other findings carry over unchanged
9. **Confirmation gate** — after any review, say "looks good" → expect: findings restated cleanly, skill does not send, file, or transmit anything anywhere
10. **Redline is not applied to a file** — ask "can you apply these redlines directly to my Word doc?" → expect: skill explains it cannot edit or generate a `.docx` file, and that every redline is chat text to copy in yourself

---

### Release build verification

```bash
npm run build
sha256sum -c contract-document-reviewer-v1.0.0.zip.sha256
```

Both commands must exit 0. Install the `.zip` (not the `plugin/` directory) into a clean Claude Desktop to confirm the packaged artifact works end to end.

---

## Why This Matters

Dedicated AI contract-review tools are priced for high transactional volume — $99–400/month — which doesn't pencil out for a solo or small-firm attorney reviewing contracts occasionally. Generic AI drafting, on the other hand, risks "grammatically correct but substantively wrong" output with no firm-specific positions constraining it. This plugin closes that gap: a playbook-constrained, clause-by-clause first pass that's good enough for occasional use, without pretending to replace your judgment on whether a clause actually works for this deal.

---

## Want Your Firm's Own Playbook Built In?

This plugin ships with one generic playbook covering common commercial clause types. Protomated can build your firm's own multi-document, multi-clause playbook — encoding your actual negotiation positions across every contract type you review regularly — as a custom, done-for-you engagement.

[Book a 30-minute call →](https://protomated.com/call)

---

## License

MIT. See [LICENSE](LICENSE).

## Feedback and Issues

[GitHub Issues](https://github.com/protomated/claude-contract-document-reviewer/issues) | [hello@protomated.com](mailto:hello@protomated.com)
