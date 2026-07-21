# Court Deadline Reasoning & Calendar Drafting — Claude Desktop Plugin

A Claude Desktop plugin for solo and small-firm attorneys. One skill (`/court-deadline`) takes a trigger date and the applicable rule in plain English, computes the deadline step by step with an auditable reasoning chain, and drafts a Google Calendar event on explicit attorney confirmation.

Distributed free by [Protomated](https://protomated.com).

---

## Repo layout

```text
plugin/           Installable plugin (packaged into .zip)
  .claude-plugin/plugin.json   Identity manifest
  .mcp.json                    Declares Google Calendar connector requirement
  manifest.json                Display metadata
  prompts/system-prompt.md     Master system prompt — compliance guardrails live here
  skills/court-deadline/
    SKILL.md                   The single skill (deadline reasoning + calendar drafting)

scripts/
  validate-plugin.mjs          Validates plugin/ structure before packing

docs/
  Court Deadline Reasoning - Technical.md   Technical specification

.github/workflows/
  validate.yml     Runs on every push/PR — validates plugin structure
  release.yml      Runs on vX.Y.Z tags — builds, checksums, and publishes a GitHub Release
```

---

## Skill

| Skill | What it does |
|---|---|
| `/court-deadline` | Takes a trigger date + rule in plain English → resolves any ambiguities → computes deadline step by step → shows full reasoning chain → offers to draft a Google Calendar event (created only on attorney confirmation) |

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

This is a content plugin — testing is manual inside Claude Desktop. There is no test runner.

### Setup

1. **Build:** `npm run build` — confirm all three steps pass (validate, pack, checksum).
2. **Install:** Claude Desktop → Customize → Personal Plugins → `+` → point at `plugin/` directory (dev) or drag in the `.zip` (release test).
3. **Connect Google Calendar:** Claude Desktop → Settings → Connectors → Google Calendar → Connect → sign in.
4. **Verify skill loads:** type `/skills` in a new chat — `/court-deadline` must appear.

---

### Test inputs and what to check

Run each input below and verify the expected behaviour. Every test case exercises a different branch in the skill logic.

---

#### 1. Basic calendar-day count

```
/court-deadline served via personal service on June 15 2026 (Monday), responsive pleading due 21 calendar days after service, if deadline falls on weekend or federal holiday the next business day applies
```

**Check:**
- Reasoning chain shows Day 0 = June 15, count starts June 16
- Raw deadline computed as July 6 (Monday) — no rollover needed
- Compliance header present: `⚠️ NOT A SUBSTITUTE FOR DOCKETING SOFTWARE`
- Compliance footer present: `— Prepared with Protomated Court Deadline Reasoning`
- Skill offers to draft a calendar event and does not create one without confirmation

---

#### 2. Deadline rolls over a weekend

```
/court-deadline order entered June 10 2026 (Wednesday), notice of appeal due 30 calendar days from entry, rolls to next business day if deadline falls on a weekend or federal holiday
```

**Check:**
- Raw deadline: July 10 (Friday) — no rollover
- Change to June 11 as trigger to push raw deadline to July 11 (Saturday) → rolled to Monday July 13
- Reasoning chain names the Saturday explicitly and states the rollover

---

#### 3. Deadline lands on a federal holiday

```
/court-deadline complaint filed June 5 2026, defendant's answer due 21 calendar days after service completed June 5, rolls to next business day if weekend or federal holiday
```

Adjust trigger date to land the raw deadline on July 4 (Independence Day) — e.g. trigger June 13 puts Day 21 at July 4.

**Check:**
- Skill identifies July 4 as Independence Day (federal holiday)
- Rolls to July 6 (Monday) — confirms July 6 is not itself a weekend or holiday
- Holiday is named in the reasoning chain, not just skipped silently

---

#### 4. Business-day count

```
/court-deadline motion filed July 1 2026, opposition due 15 business days after filing, excluding weekends and federal holidays
```

**Check:**
- Skill counts forward in business days, not calendar days
- Labor Day (first Monday of September) skipped if it falls in the window
- Final date is further out than a raw calendar-day count would produce
- Reasoning chain names each weekend block and any holiday skipped

---

#### 5. Ambiguous rule — no day type specified

```
/court-deadline served July 7 2026, responsive pleading due 21 days after service
```

**Check:**
- Skill does **not** compute immediately
- Asks: "Does this rule count calendar days or business days?"
- Only proceeds after attorney answers
- Does not guess

---

#### 6. Ambiguous rule — no rollover stated

```
/court-deadline judgment entered August 3 2026, notice of appeal due 30 calendar days from entry
```

**Check:**
- Skill computes the raw date
- If raw date is a weekday and not a holiday, states the result and notes no rollover rule was supplied (or asks if one applies — either is acceptable; it must not silently apply a rollover)

---

#### 7. Month arithmetic

```
/court-deadline accrual date January 31 2026, statute of limitations 2 years from accrual
```

**Check:**
- Skill asks or states its interpretation of "2 years" (calendar years vs. exact day count)
- Handles January 31 + 2 years = January 31 2028 correctly
- Then try: "one month from January 31 2026" — skill must address the end-of-month edge case (February 28 vs. March 2) and ask or state which interpretation it applies

---

#### 8. Confirmation gate — decline

Reach the calendar event offer and respond **no** or **skip**.

**Check:**
- No calendar event is created
- Skill closes with the final deadline date and the "verify independently" reminder
- Nothing is written to Google Calendar

---

#### 9. Confirmation gate — confirm

Reach the calendar event offer, review the draft, respond **yes**.

**Check:**
- Event draft shown in full before creation (title, date, time, description)
- Event appears in Google Calendar after confirmation
- Confirmation message tells you to verify the event in Google Calendar
- Event description contains the compliance note

---

#### 10. Multiple deadlines from one rule

```
/court-deadline summary judgment motion filed August 10 2026, opposition due 21 calendar days after filing, reply due 14 calendar days after opposition, all deadlines roll to next business day if weekend or federal holiday
```

**Check:**
- Two separate numbered computation blocks — one for opposition, one for reply
- Each has its own weekend/holiday check
- Both carry the compliance wrapper

---

### Release build verification

```bash
npm run build
sha256sum -c court-deadline-reasoning-v1.0.0.zip.sha256
```

Both commands must exit 0. Install the `.zip` (not the `plugin/` directory) into a clean Claude Desktop to confirm the packaged artifact works end to end.

---

## Cutting a release

Update `RELEASE.md` at the repo root, then push a semver tag — CI does the rest:

```bash
git tag v1.0.1
git push origin v1.0.1
```

The release workflow validates, builds, checksums, and publishes a GitHub Release with `court-deadline-reasoning-v1.0.1.zip` and `.sha256` attached.

---

## Compliance

The plugin enforces five non-negotiable rules, defined in `plugin/prompts/system-prompt.md` and `SKILL.md`:

1. **Confirmation gating** — Claude must show the full calendar event draft and get explicit in-conversation confirmation before creating any event.
2. **Required output wrapper** — every skill output begins with `⚠️ NOT A SUBSTITUTE FOR DOCKETING SOFTWARE` and ends with the `— Prepared with Protomated...` footer.
3. **Plan-tier warning** — consumer Claude (Personal/Pro) must not be used with confidential matter information.
4. **Hard compliance note** — all outputs carry verbatim: "computes from the rule you provide; does not know your jurisdiction's rules; not a substitute for docketing software or your own verification."
5. **Ambiguity resolution** — the skill must ask before computing if the rule is ambiguous on day type, counting anchor, rollover, or holiday scope. It must never guess.

Do not weaken these constraints.

---

## License

MIT. See [LICENSE](plugin/LICENSE).
