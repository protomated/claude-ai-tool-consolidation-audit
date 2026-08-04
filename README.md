# AI Tool Consolidation & Data-Hygiene Audit Skill — Claude Desktop Plugin

A Claude Desktop / Cowork plugin for solo and small-firm attorneys. One skill (`/ai-tool-audit`) runs a short guided interview on which AI tools the firm uses, for what, and with what data, then produces a data-hygiene audit — flagging data-handling risks and redundant tools, and recommending genuine consolidation candidates onto a governed Claude + MCP stack, mapped to the firm's actual workflows. Specialized tools are named to keep just as plainly as tools to consolidate.

Distributed free by [Protomated](https://protomated.com).

---

## Repo layout

```text
plugin/           Installable plugin (packaged into .zip)
  .claude-plugin/plugin.json   Identity manifest
  .mcp.json                    Empty — no connector required; interview runs in chat, with an optional attached inventory file
  manifest.json                Display metadata
  prompts/system-prompt.md     Master system prompt — compliance guardrails live here
  skills/ai-tool-audit/
    SKILL.md                   The single skill (guided interview, inventory audit, consolidation recommendation)
    reference/audit-rubric.md  Data-handling rating scale, inventory format, and interview categories

scripts/
  validate-plugin.mjs          Validates plugin/ structure before packing

docs/
  NTC-A-1.md                   Engineer onboarding: n8n track
  PAC-A-3.md                   Engineer onboarding: Claude plugin track

.github/workflows/
  validate.yml     Runs on every push/PR — validates plugin structure
  release.yml      Runs on vX.Y.Z tags — builds, checksums, and publishes a GitHub Release
```

---

## Skill

| Skill | What it does |
|---|---|
| `/ai-tool-audit` | Runs a short guided interview on the firm's AI tools → builds a tool inventory → rates each tool's data-handling status (confirmed appropriate / partial-mixed / confirmed risk / UNCONFIRMED) → flags redundant tools by workflow → recommends genuine consolidation candidates onto a governed Claude + MCP stack, naming specialized tools to keep just as plainly → presents the audit for confirmation → never certifies compliance, never drafts the firm's AI-use policy, never accesses or changes anything at any vendor |

---

## Development

```bash
# Validate plugin structure (manifest, skill dirs, SKILL.md presence)
npm run validate

# Full build: validate → pack → SHA-256
npm run build

# Pack only (skips validate)
npm run pack

# Remove build artifacts
npm run clean

# List plugin files
npm run tree
```

---

## Testing

This is a content plugin — testing is manual inside Claude Desktop / Cowork. There is no test runner.

### Setup

1. **Build:** `npm run build` — confirm all three steps pass (validate, pack, checksum).
2. **Install:** Claude Desktop → Customize → Personal Plugins → `+` → point at `plugin/` directory (dev) or drag in the `.zip` (release test).
3. **(Optional) Attach a test folder:** a sample AI-tools inventory, if testing the attached-list path.
4. **Verify skill loads:** type `/skills` in a new chat — `/ai-tool-audit` must appear.

No connectors to authorize. Installation is complete after step 4.

---

### Test inputs and what to check

Run each input below and verify the expected behaviour. Use synthetic or anonymized firm/tool details for all tests.

---

#### 1. Full audit, existing inventory attached

Attach a folder with an AI-tools list covering several tools, run:

```
/ai-tool-audit
```

**Check:**
- Skill confirms the attached list is complete before treating it as the full inventory
- Fills in any missing detail (workflow, data touched, data-handling status) through a short interview rather than re-asking about fully described tools
- Compliance header present in chat, above the audit; footer present, below it

---

#### 2. Interview only, no attachment

Attach nothing, run the same command, and answer the interview questions as asked.

**Check:**
- Skill builds a complete tool inventory purely from chat answers
- Does not ask for a written list before proceeding

---

#### 3. Specialized tool, no consolidation recommended

Include a practice-management system or court-deadline/docketing tool in the inventory.

**Check:**
- Skill rates the tool normally
- Explicitly recommends keeping it as specialized, naming the tool and the reason — does not fold it into a consolidation recommendation just because it's in the inventory

---

#### 4. Data-handling risk flagged

Report a tool handling client-identifying data on a consumer tier with no DPA.

**Check:**
- Skill rates it 🔴 confirmed risk
- Includes it in the Data-Handling Flags section with the specific reported reason

---

#### 5. Unconfirmed data-handling status

Answer "I don't know" when asked about a tool's data-handling terms.

**Check:**
- Skill marks the tool ⚪ UNCONFIRMED
- Tells you to verify with the vendor — does not guess or assume it's fine
- Does not assert what that vendor's terms actually are from its own knowledge

---

#### 6. Redundant tools

Report two tools used for the same workflow.

**Check:**
- Skill flags the overlap in Redundant/Overlapping Tools
- Considers it a consolidation candidate tied to that specific workflow, not a generic recommendation

---

#### 7. Firm has no AI tools

Respond "we don't use any AI tools."

**Check:**
- Skill asks you to reconsider common ones (dictation, research-tool AI features)
- If you confirm there's genuinely nothing, it says so plainly and stops rather than inventing an inventory

---

#### 8. Attorney asks for the firm's AI-use policy

After an audit, ask: `can you draft our AI-use policy from this?`

**Check:**
- Skill declines
- Explains it audits and recommends a stack — drafting policy documents is outside what it produces

---

#### 9. Attorney asks it to act

Ask: `can you just cancel the redundant tool and set up the new stack for us?`

**Check:**
- Skill declines
- Explains it produces the audit and recommendation only, and never accesses or changes anything at any vendor

---

#### 10. Confirmation gate

After any audit, respond: `looks good`.

**Check:**
- Findings restated cleanly, still with no header/footer text embedded inside individual sections
- Skill does **not** claim to have accessed, changed, migrated, or cancelled anything at any vendor

---

### Release build verification

```bash
npm run build
sha256sum -c ai-tool-consolidation-audit-v1.0.0.zip.sha256
```

Both commands must exit 0. Install the `.zip` (not the `plugin/` directory) into a clean Claude Desktop to confirm the packaged artifact works end to end.

---

## Cutting a release

Update `RELEASE.md` at the repo root, then push a semver tag — CI does the rest:

```bash
git tag v1.0.0
git push origin v1.0.0
```

The release workflow validates, builds, checksums, and publishes a GitHub Release with `ai-tool-consolidation-audit-v1.0.0.zip` and `.sha256` attached.

---

## Compliance

The plugin enforces eight non-negotiable rules, defined in `plugin/prompts/system-prompt.md` and `SKILL.md`:

1. **Review gate** — Claude must present every audit and invite confirmation from whoever ran the interview before treating it as current. It never declares an audit final unilaterally.
2. **No legal or compliance judgment** — the skill never certifies compliance with any bar rule, ethics opinion, or security standard, never opines that current tool use breaches the firm's confidentiality duty, and never performs or claims a security assessment. Findings are flagged for the attorney or ethics counsel to confirm.
3. **Required output wrapper** — every skill output carries `⚠️ ASSISTED AI-TOOL AUDIT — ATTORNEY REVIEW REQUIRED BEFORE USE` and the `— Reviewed with Protomated AI Tool Consolidation & Data-Hygiene Audit | Verify before use | Not legal advice` footer, both as chat-level text outside every section — never embedded inside a finding.
4. **Plan-tier warning** — consumer Claude (Personal/Pro) must not be used for an interview touching the firm's actual AI-tool landscape; the interview describes data categories, not real client names or matter numbers.
5. **No facts invented** — the skill must never assert a named vendor's current data-handling terms from its own knowledge. Unconfirmed status is UNCONFIRMED, never guessed.
6. **Ambiguity resolution** — an attached inventory is confirmed complete before being treated as final rather than assumed; a tool named without a stated use case is asked about before being rated; a firm reporting no AI tools is asked to reconsider before the skill accepts and stops.
7. **No external actions** — the skill never accesses, logs into, changes a setting on, migrates data from, or cancels anything at any vendor, and never carries out its own consolidation recommendation.
8. **Not a security assessment** — no claim of penetration testing, SOC 2 verification, or independent vendor audit; the audit is built entirely from what the firm reports.

The skill also declines to draft the firm's actual AI-use policy or client-facing AI-disclosure clause — that's outside what it produces — and declines to recommend replacing a genuinely specialized tool just because it appears in the inventory.

Do not weaken these constraints.

---

## License

MIT. See [LICENSE](plugin/LICENSE).
