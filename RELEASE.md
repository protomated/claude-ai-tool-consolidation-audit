# Estate Planning Document Assembler v1.0.0

Initial release.

## What's included

### `/estate-documents` — Estate Planning Document Assembly

An estate planning document assembler for solo and small-firm attorneys:

- **One intake pass, four documents:** the skill reads a client's intake answers — family structure, assets, beneficiaries, healthcare wishes — from an attached workspace folder and populates a basic will, healthcare power of attorney, financial power of attorney, and HIPAA authorization consistently from that single pass.
- **Uses your own templates, or a generic fallback:** for each document type, the skill follows your firm's own state-specific template if you've attached one, or its own bundled generic placeholder if you haven't — clearly labeled as generic, not state-specific.
- **Never determines execution requirements or which documents a client needs:** the skill leaves an explicit placeholder for state-specific witnessing and notarization requirements, and declines to decide whether a client needs documents beyond these four — that stays the attorney's call.
- **Flags gaps per document, never invents facts:** if a document type's required intake fields are missing, the skill names exactly what's missing and drafts the other complete document types anyway — one gap never blocks the whole set.
- **Keeps names and agents consistent across the set:** the same person's name, role, and ordering match across every document drafted in the same session.
- **Attorney review gate:** presents every draft with an explicit confirmation step before marking a set ready. Never notarizes, files, records, or submits anything anywhere.

Handles: basic wills, healthcare powers of attorney, financial powers of attorney, and HIPAA authorizations — the document set estate planning attorneys assemble nearly identically, client after client.

## Setup

Install time: about 10 minutes. Download the zip, drag it into Claude Desktop's Extensions panel, and attach a workspace folder with your client's intake answers and (optionally) your firm's own state-specific templates. No connectors to authorize. Open a new chat, type `/skills`, and verify `/estate-documents` appears.

## Compliance

Requires Claude for Work, Claude Team, or Claude Enterprise. Do not use a consumer Claude plan (Claude Pro or Personal) with confidential client or matter information. Every output carries an "ASSISTED DRAFT — ATTORNEY REVIEW & STATE-SPECIFIC VERIFICATION REQUIRED" header and footer, shown around each draft — never inside a document your client might sign. The skill never marks a document set ready without your explicit confirmation, never determines your state's execution requirements or which documents your client needs, and never notarizes, files, or submits anything itself. All drafts are populated from your intake answers only — no facts are invented.
