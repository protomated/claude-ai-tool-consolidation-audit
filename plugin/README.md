# AI Tool Consolidation & Data-Hygiene Audit — Claude Desktop Plugin

A Claude Desktop / Cowork plugin that runs a short guided interview on which AI tools your firm uses, for what, and with what data — then produces a data-hygiene audit flagging data-handling risks and redundant tools, and recommending genuine consolidation candidates onto a governed Claude + MCP stack, for solo and small-firm attorneys managing an average of 18 different AI tools with no single source of truth.

**Distributed by [Protomated](https://protomated.com) as a free download.**

---

## ⚠️ Required: Read This Before You Run the Interview

**This section is not boilerplate. Read it before naming your firm's AI tools.**

### 1. You must be on a qualifying Claude plan

Do NOT run this interview on a consumer Claude plan (claude.ai Personal or Claude Pro) if it will touch anything about your firm's actual client data flows. Consumer plans do not provide a Data Processing Agreement (DPA) covering that.

Use one of the following:

- **Claude for Work** (formerly Claude.ai Teams)
- **Claude Team or Enterprise**
- **Claude API** (with a signed DPA from Anthropic)

> **If you're not sure which plan you're on:** Open Claude Desktop → Help → About. If it says "Claude Pro," you are on a consumer plan. Upgrade to Claude for Work first.

**Describe data categories, not real client names or matter numbers, while answering the interview.** The audit doesn't need actual client-identifying details to work.

### 2. This is not a security assessment or a compliance certification

This plugin builds its audit entirely from what you report about your own AI tools. It does not perform penetration testing, does not independently verify any vendor's claims, and does not certify that your current AI tool use complies with any bar rule, ethics opinion, or security standard. You're responsible for verifying anything the audit marks UNCONFIRMED, and for confirming compliance questions with ethics counsel or your state bar's AI guidance.

### 3. It never states a vendor's terms from its own knowledge

If you don't know a tool's data-handling terms — whether it's on a business tier, whether a DPA is signed, whether it trains on your inputs — say so. The skill marks that tool UNCONFIRMED and tells you to check with the vendor. It never asserts a specific product's current policy from its own training data; those terms change, and a wrong claim here is worse than an honest gap.

### 4. It recommends — it doesn't act

The plugin produces an audit and a recommendation. It never logs into, changes a setting on, migrates data from, or cancels anything at any vendor account, and it never carries out the consolidation it recommends. You, your staff, or a separate engagement do that.

---

## Installation (about 5 minutes)

### Step 1 — Download and install

1. Download `ai-tool-consolidation-audit.zip` from the [Releases page](https://github.com/protomated/claude-ai-tool-consolidation-audit/releases).
2. Double-click the `.zip` file, or drag it into Claude Desktop's **Extensions** panel.
3. Claude Desktop will install the plugin.

No connectors to authorize. No credentials to configure.

### Step 2 — (Optional) Attach an existing AI-tools list

If your firm already has a written list of the AI tools it uses, attach it as a workspace folder before running the skill — the skill will confirm it's complete and fill in any missing detail through the interview. If you don't have one, skip this step; the skill interviews you from scratch.

### Step 3 — Verify

Open a new Claude Desktop chat and type `/skills`. You should see `/ai-tool-audit` listed. Run `/ai-tool-audit` to start.

---

## The Skill

### `/ai-tool-audit` — AI Tool Consolidation & Data-Hygiene Audit

Runs a short guided interview on your firm's AI tools, then:

1. Builds a tool inventory — what each tool is used for, who uses it, what data it touches
2. Rates each tool's data-handling status: **confirmed appropriate**, **partial/mixed**, **confirmed risk**, or **UNCONFIRMED** (never guessed)
3. Flags redundant tools serving the same workflow
4. Recommends genuine consolidation candidates onto a governed Claude + MCP stack — and names, just as plainly, which tools should stay exactly where they are

**What you supply:**
- Answers to a short interview about which AI tools you use, for what, and with what data — about 10 minutes
- An existing AI-tools list, if you have one, to speed the interview up
- Verification of anything the audit marks UNCONFIRMED — the skill doesn't guess vendor terms

**What it produces:**
- A tool inventory table with a data-handling status per tool
- A plain flag for any tool carrying sensitive data without confirmed protection
- A redundancy note for any workflow covered by more than one tool
- A consolidation recommendation tied to your firm's actual workflows — with genuinely specialized tools named as "keep," not silently ignored

**What it does not do:**
- It does not certify your firm's AI tool use as compliant with any bar rule, ethics opinion, or security standard
- It does not perform a security assessment or independently verify a vendor's claims
- It does not draft your firm's actual AI-use policy or a client-facing AI-disclosure clause — that's a separate scope this skill doesn't cover
- It does not access, change, migrate, or cancel anything at any vendor, and doesn't carry out its own recommendation
- It does not invent an inventory, a use case, or a data-handling status you didn't report

**Example inputs:**

```
/ai-tool-audit
/ai-tool-audit [attach your existing AI-tools list first]
```

**Typical use time:** about 10 minutes for the interview and first audit.
**Setup:** about 5 minutes (install plugin).

---

## FAQ

**Does this just pitch Protomated's services?**
No — the audit is built from what you report, and it names tools to keep as plainly as tools to consolidate. If your firm's specialized practice management system or docketing engine is doing its job, the audit says so and doesn't recommend touching it. The consolidation recommendation only applies where a redundancy or a data-handling gap actually shows up in what you reported. Protomated does offer a paid Fractional Advisory engagement to actually carry out a consolidation, but the free audit works the same way whether or not you ever book that call.

**Does this replace our firm's AI-use policy?**
No. This skill audits your current tool landscape and recommends a stack — it does not draft an internal AI-use policy or a client-facing AI-disclosure clause. Drafting those is outside this skill's scope.

**Is this a security or compliance certification?**
No. It's built entirely from what you report in the interview, not an independent technical or legal review. Anything you don't know is marked UNCONFIRMED, not assumed safe.

**Does it change anything about our actual tools?**
No. It never logs into, changes, migrates, or cancels anything at any vendor. Every output is a recommendation for you to act on.

---

## Testing guide

Run these inputs to verify the plugin is working correctly. Use synthetic or anonymized firm/tool details for every test.

1. **Full audit, existing inventory attached** — attach a folder with an AI-tools list covering several tools, run `/ai-tool-audit` → expect: skill confirms the list is complete before proceeding, fills in missing detail through a short interview, and produces a full audit
2. **Interview only, no attachment** — attach nothing, run `/ai-tool-audit` and answer the interview questions as they're asked → expect: skill builds the same kind of inventory purely from chat answers
3. **Specialized tool, no consolidation recommended** — include a practice-management system or a court-deadline/docketing tool in the inventory → expect: skill rates it normally but explicitly recommends keeping it as specialized, naming the tool and the reason, rather than folding it into a consolidation recommendation
4. **Data-handling risk flagged** — report a tool that handles client-identifying data on a consumer tier with no DPA → expect: skill rates it 🔴 confirmed risk and includes it in the Data-Handling Flags section with the specific reason
5. **Unconfirmed data-handling status** — say "I don't know" when asked about a tool's data-handling terms → expect: skill marks it ⚪ UNCONFIRMED and tells you to verify with the vendor — it does not guess or assume it's fine
6. **Redundant tools** — report two different tools used for the same workflow (e.g., two summarization tools) → expect: skill flags the overlap in Redundant/Overlapping Tools and considers it as a consolidation candidate, tied to that specific workflow
7. **Firm has no AI tools** — respond "we don't use any AI tools" when asked → expect: skill asks you to reconsider common ones (dictation, research-tool AI features), and if you confirm there's genuinely nothing, it says so plainly and stops rather than inventing an inventory
8. **Attorney asks for the firm's AI-use policy** — after an audit, ask "can you draft our AI-use policy from this?" → expect: skill declines, explains that's a separate skill's job
9. **Attorney asks it to act** — ask "can you just cancel the redundant tool and set up the new stack for us?" → expect: skill declines, explains it produces the audit and recommendation only, and never accesses or changes anything at any vendor
10. **Confirmation gate** — after any audit, say "looks good" → expect: findings restated cleanly; skill does not claim to have accessed, changed, or migrated anything anywhere

---

### Release build verification

```bash
npm run build
sha256sum -c ai-tool-consolidation-audit-v1.0.0.zip.sha256
```

Both commands must exit 0. Install the `.zip` (not the `plugin/` directory) into a clean Claude Desktop to confirm the packaged artifact works end to end.

---

## Why This Matters

Firms are accumulating AI tools faster than they're tracking what those tools do with client data — an average of about 18 different tools per firm, often adopted informally, each with its own data-handling terms nobody has fully reviewed. This plugin gives a firm a fast, honest first look at that sprawl: what's actually being used, what's carrying real data-hygiene risk, what's simply duplicated effort, and what's already working fine and shouldn't be touched.

---

## Want the Consolidation Actually Carried Out?

This plugin produces the audit and the recommendation. Protomated can carry out the actual migration to a governed Claude + MCP stack — staff training, a rollout plan, and ongoing oversight — as a Fractional Advisory engagement.

[Book a 30-minute call →](https://protomated.com/call)

---

## License

MIT. See [LICENSE](LICENSE).

## Feedback and Issues

[GitHub Issues](https://github.com/protomated/claude-ai-tool-consolidation-audit/issues) | [hello@protomated.com](mailto:hello@protomated.com)
