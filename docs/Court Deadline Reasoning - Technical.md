# Court Deadline Reasoning & Calendar Drafting — Technical

**Product:** Court Deadline Reasoning & Calendar Drafting (Claude Desktop plugin)
**Audience:** Solo and small-firm attorneys using Claude Desktop
**Distribution model:** Free download (lead magnet) → consulting upsell ($5K–$15K)
**Team size assumed:** 1 developer + 1 content/legal SME
**Monthly infrastructure budget assumed:** $0 at MVP scale

---

## Architectural Context

This product is a **Claude Desktop plugin** — a small folder structure containing skill content and a manifest, packaged as a `.zip` file the attorney installs into Claude Desktop with a one-click drag-and-drop. There is no backend, no MCP server, and no auth layer beyond the Google Calendar OAuth flow managed by Claude Desktop.

(`.zip` is a ZIP archive that Claude Desktop recognises natively. The format has two variants: **standalone** bundles a single MCP server, and **plugin** bundles skills plus declared connector requirements. We use the plugin variant.)

The plugin reduces to two layers:

1. **Skill content** — one `SKILL.md` file and a master system prompt. This is the entirety of what we produce.
2. **Distribution infrastructure** — a landing page (WordPress on protomated.com), email capture form, and `.zip` hosting. The landing page is managed separately from this repo.

Marginal cost per installed attorney is zero. Infrastructure cost is flat regardless of download volume up to ~10K email subscribers.

---

## 1. Stack Summary

| Layer | Technology | Notes |
|---|---|---|
| **Plugin Runtime** | Claude Desktop (managed by Anthropic) | We write no runtime code. |
| **Plugin Format** | Claude Plugin (`.claude-plugin/plugin.json` + `.mcp.json` + `skills/`), packaged as `.zip` (plugin variant) | Open spec at code.claude.com/docs/en/plugins. |
| **Skills** | `SKILL.md` (Anthropic spec) | YAML frontmatter + markdown body. One file. |
| **Calendar Integration** | Claude Desktop built-in Google Calendar connector | Declared as required in `.mcp.json`. Anthropic handles OAuth. Used to create deadline calendar events. |
| **Landing Page** | WordPress on protomated.com | `/templates/court-deadline-reasoning-skill/` — managed outside this repo. |
| **Plugin Hosting** | GitHub Releases | Free unlimited bandwidth for public `.zip` release assets. |
| **Email Capture** | Kit (formerly ConvertKit) — free Newsletter plan | Up to 10,000 subscribers free. |
| **Code Hosting / CI** | GitHub + GitHub Actions | Free; CI packs the `.zip` on tag push. |

**Total monthly cost at MVP scale: $0.**

---

## 2. Key Architectural Decisions

### 2.1 Use Claude Desktop's built-in Google Calendar connector; build no MCP servers

**Decision:** The plugin declares only Google Calendar as a required connector in `.mcp.json`. No custom MCP server.

**Why:** The `/court-deadline` skill works entirely from the attorney's in-conversation inputs — it conducts no interview beyond collecting the trigger date and rule, and the only external operation is optionally creating a calendar event in the attorney's Google Calendar.

Claude Desktop ships a first-party managed Google Calendar connector. Declaring it as a requirement via `.mcp.json` causes Claude Desktop to prompt the attorney with a "Connect" button and a Google sign-in flow; Anthropic handles OAuth token management. We never touch attorney credentials or write any integration code.

**Trade-off:** We are coupled to whatever tool names and capabilities the Google Calendar connector exposes. If Anthropic renames the connector identifier or changes tool signatures, the skill may need a one-line update. Mitigation: monitor Anthropic's changelog; verify the connector identifier `google-calendar` against the live Connectors panel before each release.

### 2.2 Plugin variant of `.zip`, not standalone

**Decision:** Package as the **plugin variant** of `.zip`, not the standalone variant.

**Why:** The standalone variant bundles a single MCP server — appropriate for shipping custom integration code. The plugin variant bundles skills + connector requirement declarations + manifests — appropriate when reusing Anthropic's built-in connectors. Since we ship no MCP server, the plugin variant is the right choice.

**Both variants give the attorney the same one-click `.zip` install experience** in Claude Desktop. The choice is purely about what's inside the bundle.

### 2.3 Compliance language lives in skills, not in a custom UI

**Decision:** Required guardrails (the "not a substitute for docketing software" warning, the rule-supply-each-time requirement, and the confirmation gate before any calendar event creation) are enforced through the master system prompt and through mandatory headers/footers in every skill output.

**Why:** Plugins do not execute code on install — they cannot show custom UI. The compliance layer must be content-driven. Claude enforces the system prompt on every conversation, making it the right enforcement surface.

### 2.4 Reasoning chain as the primary output

**Decision:** The skill structures every response so the step-by-step computation precedes the deadline summary — not the other way around. The reasoning chain is the deliverable; the date is the conclusion.

**Why:** The differentiator from a spreadsheet formula (PAC-16, PAC-22) is auditability. An attorney can follow the reasoning chain, catch a wrong assumption, and fix it before missing a deadline. Burying the reasoning in a footnote defeats the purpose. This is also the CLE talk angle: "I can show you the work" is a professional competence argument, not just a feature.

---

## 3. Infrastructure

### 3.1 Hosting & Deployment

| Environment | Purpose | Host |
|---|---|---|
| **Dev** | Local plugin development and skill testing | Developer's laptop; plugin loaded into Claude Desktop via Personal Plugins panel |
| **Distribution** | Landing page + `.zip` download | WordPress/protomated.com (landing) + GitHub Releases (`.zip` artifact) |
| **CI/CD** | Validate skill, pack, and publish on git tag | GitHub Actions |

**CI flow on tag push (`v1.0.0`, etc.):**
1. Run `npm run validate` (`scripts/validate-plugin.mjs`) against `plugin.json`, `.mcp.json`, and `SKILL.md` frontmatter
2. Run `npm run pack` to produce `court-deadline-reasoning-v1.0.0.zip`
3. Run `npm run checksum` to compute SHA-256
4. Publish `.zip` + checksum to GitHub Releases

### 3.2 "Database" Schema (local-only state)

There is no central database. State lives entirely on the attorney's machine and inside Claude Desktop:

```
PLUGIN_DIR
  ├── .claude-plugin/plugin.json
  ├── .mcp.json                  (declares: google-calendar)
  └── skills/court-deadline/SKILL.md

GOOGLE_CALENDAR
  └── Calendar events created by the attorney during sessions
      (managed entirely by Google; not accessible to Protomated)
```

**What this means:**
- The plugin directory lives wherever Claude Desktop stores Personal Plugins.
- The Google Calendar connector is authorized by the attorney — the plugin can only create events in the calendar the attorney authorized.
- No cloud-side records, ever. We do not know who has installed the plugin.

### 3.3 Background Jobs

There are no background jobs anywhere. The plugin is static content.

| Job | Schedule / Trigger | Purpose |
|---|---|---|
| **CI: build & release** | On git tag push | GitHub Actions validates the skill and publishes the `.zip`. |
| **Email drip sequence** | Triggered by download/opt-in | Kit sends a nurture sequence ending in a consulting CTA. |

### 3.4 Third-Party Integrations

| Service | Purpose | Tier / Cost at MVP |
|---|---|---|
| **Anthropic Claude Desktop** | The runtime | Paid by attorney (Claude for Work / Team / Enterprise) |
| **Google Calendar** | Calendar event creation | Managed by Anthropic's connector; attorney's Google account |
| **GitHub Releases** | `.zip` artifact distribution | Free (unlimited bandwidth on public release assets) |
| **Kit (ConvertKit)** | Email capture + nurture sequence | Free Newsletter plan up to 10,000 subscribers |
| **GitHub Actions** | CI/CD pipeline | Free on public repos |

**Total third-party cost at MVP: $0/month.**

---

## 4. Authentication & Security

### 4.1 Auth approach

**We have no authentication system.** The only connector this plugin requires is Google Calendar, which uses OAuth managed entirely by Claude Desktop and Google. The attorney signs in with their Google account once; Anthropic handles token storage and refresh. No credentials leave the attorney's machine to Protomated.

The only auth-adjacent thing we ship is the connector declaration in `.mcp.json`:

```json
{
  "mcpServers": {
    "google-calendar": { "type": "http", "url": "" }
  }
}
```

When the attorney installs the plugin, Claude Desktop's Connectors panel shows Google Calendar with a "Connect" button and the note "Required by: Court Deadline Reasoning." One click — completing the Google sign-in flow — completes setup.

### 4.2 Data handling policies

| Data class | Where it lives | Retention |
|---|---|---|
| Skill files | Plugin directory inside Claude Desktop | Until attorney uninstalls the plugin |
| Trigger dates and rule text | Transient — held only in Claude's conversation context | Subject to attorney's Claude plan retention |
| Calendar events (if created) | Attorney's Google Calendar, in the account they authorized | Google's standard retention; never copied or transmitted by us |
| Landing-page email | Kit subscriber list | Until subscriber unsubscribes; deletable on request |

**Nothing the plugin processes is ever transmitted to Protomated infrastructure.**

### 4.3 Compliance considerations

- **ABA Model Rule 1.6 (Confidentiality):** The README's first section is a hard compliance gate. Attorneys must be on Claude for Work / Team / Enterprise (or Claude API with a DPA) before entering confidential matter information.
- **Competence (Model Rule 1.1):** The plugin enforces the "not a substitute for docketing software" warning in every output — attorneys must verify computed dates independently.
- **No auto-filing:** The plugin never creates calendar events without explicit in-conversation attorney confirmation. Calendar events are the only external write operation; they require affirmative consent every time.
- **Heppner (SDNY, Feb. 2026):** README and master system prompt warn explicitly about the privilege-waiver risk of using consumer-tier Claude with confidential matter information.

### 4.4 Security measures

- **Bundle integrity:** SHA-256 checksum published alongside every `.zip` release on GitHub.
- **No custom code surface:** We ship no executables, no MCP servers, no scripts. Reviewers can audit the entire plugin by reading the markdown files and two JSON manifests.
- **No network egress from our code:** Because we have no code. The only external operation is calendar event creation via the Google Calendar connector, which is scoped to the calendar the attorney authorized.
- **Confirmation gating:** All calendar event creation operations are enforced in the master system prompt — Claude must obtain explicit attorney confirmation before creating any event.

---

## 5. API Architecture

### 5.1 Internal API

**The plugin exposes no API.** It declares which connector it needs (Google Calendar) and provides skill content that Claude consumes when that connector is active.

### 5.2 Tools consumed (provided by Claude Desktop's built-in Google Calendar connector)

Tool names should be **verified against the live Claude Desktop Google Calendar connector** before finalising the skill.

| Connector | Expected operations |
|---|---|
| **Google Calendar** | Create event (within the authorized calendar) |

The `SKILL.md` instructs Claude to use the Google Calendar connector to create events. If Anthropic changes the tool interface, the skill may need an update.

### 5.3 Key Abstractions

There are no abstractions in code because there is no code. The "abstraction" is the SKILL.md format: the skill describes how to collect inputs, resolve ambiguities, reason through the computation step by step, present the reasoning chain, and gate calendar event creation on attorney confirmation. Claude executes the abstraction.

---

## 6. Cost Projections

### 6.1 Per-unit cost breakdowns

- **Per-attorney compute cost: $0.** Plugin runs on the attorney's Claude subscription.
- **Per-attorney connector cost: $0.** The Google Calendar connector is managed by Anthropic as part of Claude Desktop.
- **Per-download bandwidth cost: $0.** GitHub Releases offers unlimited bandwidth for public `.zip` release assets.
- **Per-email-subscriber cost: $0** up to 10,000 subscribers on Kit's free Newsletter plan.

### 6.2 Monthly cost projections

| Stage | Subscribers / Downloads | Monthly Cost | Breakdown |
|---|---|---|---|
| **MVP** | 0–500 | **$0** | All free tiers — GitHub Releases, Kit free |
| **Growth** | 500–5,000 | **$0** | Same free tiers; no upgrades needed |
| **Scale** | 5,000–10,000 | **$0** | Kit still free under 10K subs |
| **Beyond 10K** | 10,000+ | **~$59/mo** | Kit Creator plan (~$59/mo at 3K–5K subs); scales with list size |

### 6.3 Unit economics

Infrastructure cost is effectively zero up to 10,000 subscribers. Worked example: 1,000 downloads → 200 opt-ins → 2% lead-to-paid = 4 consulting engagements at $10,000 average = $40,000 revenue against $0 infrastructure cost.

---

## 7. Environment Variables

The plugin itself has no environment variables. It is static content.

### CI/CD (GitHub Actions secrets)

| Variable | Required | Notes |
|---|---|---|
| `GITHUB_TOKEN` | Auto-provided | Used by `gh release create` to publish releases |

### Kit / email capture

Kit form configuration is managed in the Kit dashboard, not in this repo. The WordPress landing page's form points to the Kit form directly.

---

## 8. Development Setup

**Assumed already installed:** git, a text editor, Claude Desktop.

```bash
# 1. Clone
git clone https://github.com/protomated/claude-court-deadline-reasoning.git
cd claude-court-deadline-reasoning

# 2. Inspect the structure
npm run tree
# plugin/
# ├── .claude-plugin/plugin.json
# ├── .mcp.json
# ├── manifest.json
# ├── prompts/system-prompt.md
# ├── skills/
# │   └── court-deadline/SKILL.md
# ├── CONNECTORS.md
# ├── LICENSE
# └── README.md

# 3. Install the plugin into Claude Desktop for testing
# Open Claude Desktop → Customize → Personal plugins → "+"
# Point it at the plugin/ directory

# 4. In Claude Desktop → Connectors, click "Connect" on Google Calendar
# Sign in with your Google account and authorize calendar access
# (One-time setup; Anthropic handles OAuth)

# 5. Verify in a new chat:
#    - /skills lists /court-deadline
#    - Run /court-deadline and supply a trigger date + rule
#    - Confirm the reasoning chain is shown step by step
#    - Test the calendar event flow: review the draft, confirm, verify the event appears
```

**For packaging a release:**

```bash
# Validate, pack, checksum in one step
npm run build

# Or individually:
npm run validate   # validate plugin/ structure (scripts/validate-plugin.mjs)
npm run pack       # zip to court-deadline-reasoning-v1.0.0.zip
npm run checksum   # compute SHA-256

# Cut a GitHub release (runs build first, then gh release create)
npm run release
```

Attorneys install by double-clicking the `.zip` file or dragging it into Claude Desktop's Extensions panel — Claude Desktop handles the rest.

---

## 9. Third-Party Service Setup

### 9.1 Kit (formerly ConvertKit) — Email capture

- Sign up for the free **Newsletter plan** (10,000-subscriber limit)
- Create a form titled "Court Deadline Reasoning Download"
- Configure the success action to send an email containing the GitHub Releases download link
- Build a nurture sequence ending in a consulting CTA ($5K–$15K deadline management build)

### 9.2 GitHub — Source, Releases, CI

- Repository: `claude-court-deadline-reasoning` (private during dev, public for release)
- Configure GitHub Actions secrets per Section 7
- First release published manually via UI; subsequent via CI on tag push

---

## 10. Deployment Checklist

### 10.1 Pre-Launch

- [ ] `SKILL.md` validated against Anthropic's skill spec (`npm run validate` passes)
- [ ] Google Calendar connector identifier (`google-calendar`) verified against Claude Desktop's live Connectors panel
- [ ] Ambiguity-resolution prompts tested with realistic edge-case rules (ambiguous day type, missing rollover rule, month arithmetic)
- [ ] Federal holiday list in `prompts/system-prompt.md` verified for the current year
- [ ] Compliance language (README first section + skill headers/footers + "not a substitute for docketing software" caveat in all outputs) reviewed by a licensed attorney
- [ ] README and CONNECTORS.md proofread
- [ ] Loom walkthrough video recorded — showing install, connector connect, a complete deadline computation with reasoning chain, and a calendar event creation
- [ ] Plugin tested on a clean macOS Claude Desktop install
- [ ] Plugin tested on a clean Windows Claude Desktop install
- [ ] Kit form, success email, and nurture sequence configured
- [ ] GitHub Release v1.0.0 published with `.zip` + SHA-256 checksum
- [ ] WordPress landing page download link updated to point to the v1.0.0 release URL

### 10.2 Launch Day

- [ ] CI on `v1.0.0` tag completes; `.zip` is in GitHub Releases
- [ ] Landing page download link points to the correct release URL
- [ ] End-to-end test: landing page → opt-in → email → download → install → connect Google Calendar → run `/court-deadline` → verify reasoning chain → confirm calendar event → verify event in Google Calendar
- [ ] SHA-256 checksum verified
- [ ] Post LinkedIn launch announcement (Dele) linked to landing page

### 10.3 Post-Launch

- [ ] Kit broadcast to existing subscribers each time a new version ships
- [ ] Quarterly review of Anthropic's plugin spec and Google Calendar connector tool surface for breaking changes
- [ ] Verify federal holiday list each January for the new year
- [ ] Monthly check on Kit subscriber count vs. 10K free-tier ceiling
- [ ] Feedback channel (GitHub Issues + `mailto:` in the README) triaged weekly
- [ ] Track download-to-consulting-call conversion in Kit; target ≥3%

---

## Build Effort Estimate

| Deliverable | Hours |
|---|---|
| `plugin.json` + `manifest.json` manifests | 1 |
| `.mcp.json` connector declaration | 0.5 |
| `SKILL.md` — reasoning engine + step-by-step computation protocol (including compliance review + test) | 5 |
| `prompts/system-prompt.md` (master prompt + ambiguity resolution + confirmation gating) | 2 |
| `README.md` (with compliance gate first section) | 1.5 |
| `CONNECTORS.md` (Google Calendar connector setup) | 0.5 |
| Loom walkthrough video | 2 |
| QA on macOS + Windows | 2 |
| Legal/SME review pass | 3 |
| **Total** | **~17.5 hours** |

**Estimated timeline:** 2–3 days (1 developer + 1 content/legal SME working in parallel).
