# Estate Planning Document Assembly Skill for Law Firms — Claude Desktop Plugin

A Claude Desktop / Cowork plugin that populates a basic will, healthcare power of attorney, financial power of attorney, and HIPAA authorization from one intake pass — using your firm's own state-specific templates, or generic placeholders if you haven't attached your own — for solo and small-firm estate planning attorneys who assemble near-identical document sets by hand, client after client.

**Distributed by [Protomated](https://protomated.com) as a free download.**

---

## ⚠️ Required: Read This Before You Install

**This section is not boilerplate. Read it before attaching any intake files.**

### 1. You must be on a qualifying Claude plan

Do NOT use this plugin on a consumer Claude plan (claude.ai Personal or Claude Pro) with any confidential client or matter information. Consumer plans do not provide a Data Processing Agreement (DPA) covering privileged content.

Use one of the following:

- **Claude for Work** (formerly Claude.ai Teams)
- **Claude Team or Enterprise**
- **Claude API** (with a signed DPA from Anthropic)

> **If you're not sure which plan you're on:** Open Claude Desktop → Help → About. If it says "Claude Pro," you are on a consumer plan. Upgrade to Claude for Work before attaching any confidential intake files.

### 2. Every draft is a first pass — you are the author of record

This plugin drafts from the intake answers and templates you provide. It does not verify accuracy, completeness, or your state's execution requirements, and it never decides which documents a client needs. You review every document, verify your state's witnessing and notarization rules, and finalize before your client signs anything.

### 3. FREE-tier placeholder templates are generic, not state-specific

If you don't attach your own template for a document type, the plugin uses its own bundled placeholder for that type. That placeholder makes no claim of state-specific legal accuracy — swap in your own state's form, or independently verify the placeholder, before any client signs.

### 4. This plugin does not file, notarize, or submit anything

The plugin reads only the workspace folder you explicitly attach, and it never notarizes, files, records, or submits a document anywhere. You and your client handle execution yourselves.

---

## Installation (about 10 minutes)

### Step 1 — Download and install

1. Download `estate-planning-document-assembler.zip` from the [Releases page](https://github.com/protomated/claude-estate-planning-document-assembler/releases).
2. Double-click the `.zip` file, or drag it into Claude Desktop's **Extensions** panel.
3. Claude Desktop will install the plugin.

No connectors to authorize. No credentials to configure.

### Step 2 — Attach an intake folder

Before running the skill, attach a workspace folder containing:
- Your client's intake answers (family structure, assets, beneficiaries, healthcare wishes — whatever you've gathered)
- Your firm's own state-specific templates, for any of the four document types you have one for. Skip this for any document type you don't have a template for — the plugin will use its own generic placeholder instead.

### Step 3 — Verify

Open a new Claude Desktop chat, attach your folder, and type `/skills`. You should see `/estate-documents` listed. Run `/estate-documents` to start.

---

## The Skill

### `/estate-documents` — Estate Planning Document Assembly

Populates up to four documents from one intake pass:

1. **Last Will and Testament** (basic)
2. **Healthcare Power of Attorney** (Advance Directive for Health Care)
3. **Financial (Durable) Power of Attorney**
4. **HIPAA Authorization**

The skill asks which document(s) you want if you don't say, uses your firm's own template per document type where you've attached one (and the bundled placeholder — clearly labeled — where you haven't), and only drafts a document once its required intake fields are present. A gap in one document's fields never blocks the others — it tells you exactly what's missing and drafts the rest.

**What you supply:**
- Intake answers, via an attached folder or pasted directly
- Your firm's own state-specific templates, for any document type you have one for
- Anything requiring your judgment, when you're ready to add it — the skill leaves a placeholder rather than guessing

**What it produces:**
- Ready-to-review documents, each in a clean copy-only block with no Protomated branding inside it
- Consistent names, agents, and dates across every document in the set
- A placeholder for anything requiring your judgment (state execution requirements, whether additional documents are needed)

**What it does not do:**
- It does not decide which documents a client needs, resolve a family conflict, or advise on tax strategy — that's yours to decide
- It does not determine your state's execution requirements — verify these independently
- It does not notarize, file, record, or submit anything — you and your client handle execution yourselves
- It does not invent facts, family details, or asset information not in your intake answers

**Example inputs:**

```
/estate-documents all
/estate-documents will
/estate-documents healthcare poa
/estate-documents
```

**Typical use time:** a few minutes per document set once your intake folder and any of your own templates are attached.
**Setup:** about 10 minutes (install plugin, attach intake folder and any firm templates).

---

## Testing guide

Run these inputs to verify the plugin is working correctly. Use synthetic or anonymized client details, and attach a test folder with sample intake answers and, optionally, sample firm templates.

1. **All four documents, firm templates attached, complete intake** — attach a folder with templates and complete intake, run `/estate-documents all` → expect: each draft follows its attached template's structure, cites only facts present in the intake, and keeps names/agents consistent across all four documents
2. **No firm templates attached** — attach a folder with complete intake but no templates, run `/estate-documents all` → expect: skill uses its bundled placeholder templates and says plainly that they're generic, not state-specific — it does not ask you to supply a template first
3. **Missing required fields for one document type** — attach a folder where, say, the financial POA has no named agent → expect: skill drafts the other complete documents and lists exactly what's missing for the blocked one, rather than guessing
4. **Document(s) not specified** — run `/estate-documents` with no argument → expect: skill asks which document(s) you want before drafting anything
5. **Attorney asks which documents the client needs** — after a draft, ask "does this client need a trust too?" → expect: skill declines, explains it's the attorney's call
6. **Attorney asks about state execution requirements** — ask "how many witnesses does my state require?" → expect: skill declines to state a definitive answer and points to the placeholder in the draft for the attorney to verify
7. **No folder attached** — run `/estate-documents` with nothing attached → expect: skill asks the attorney to attach a folder or paste the intake answers directly
8. **Edit and revise loop** — after a draft, say "add a section" → expect: revised draft, same facts, re-invites confirmation
9. **Confirmation gate** — after any draft set, say "looks good" → expect: final set restated cleanly with no header/footer text inside any copy block, skill does not notarize, file, or submit anything, offers to draft any remaining document type
10. **Cross-document consistency** — check that the same person named as healthcare agent in the Healthcare POA appears with identically spelled name and matching role in the HIPAA Authorization

---

### Release build verification

```bash
npm run build
sha256sum -c estate-planning-document-assembler-v1.0.0.zip.sha256
```

Both commands must exit 0. Install the `.zip` (not the `plugin/` directory) into a clean Claude Desktop to confirm the packaged artifact works end to end.

---

## Why This Matters

Estate planning attorneys assemble a near-identical document set — will, healthcare POA, financial POA, HIPAA authorization — for client after client, spending 2-3 hours per client on assembly alone. Existing free tools are scattered and incomplete, often stuck in spreadsheets pulled from state bar sites. This plugin closes the gap between "I have the client's intake answers" and "I have a first-pass document set in front of me," without touching the two decisions that are actually yours: what your state requires for execution, and what this client actually needs.

---

## Want More Than the Four Core Documents?

This plugin still requires you to attach your intake and templates and review every document by hand. Protomated can build a system that extends this into trust documents, pour-over wills, beneficiary deed templates, state funding checklists, and automated signing-ceremony scheduling — scoped to your firm's workflow.

[Book a 30-minute call →](https://protomated.com/call)

---

## License

MIT. See [LICENSE](LICENSE).

## Feedback and Issues

[GitHub Issues](https://github.com/protomated/claude-estate-planning-document-assembler/issues) | [hello@protomated.com](mailto:hello@protomated.com)
