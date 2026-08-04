# Contract & Document Reviewer v1.0.0

Initial release.

## What's included

### `/contract-review` — Contract & Document Review

A contract review assistant for solo and small-firm attorneys:

- **Playbook-constrained clause-by-clause review:** the skill reads an attached contract and compares each clause against a configurable playbook, rating it GREEN, YELLOW, RED, or UNRATED with plain-English rationale tied to the specific playbook position behind the rating.
- **Uses your own playbook, or a generic fallback:** the skill follows your firm's own `playbook.md` if you've attached one, or its own bundled generic playbook covering common commercial clause types if you haven't — clearly labeled as generic, not firm-specific.
- **Never determines enforceability or whether to sign:** the skill leaves execution and negotiation calls to the attorney — it never opines on a clause's enforceability under governing law, and never decides whether the client should sign, walk away from, or accept the contract.
- **Suggested redlines, not applied edits:** for every clause that isn't GREEN, the skill suggests redline language as copy-ready chat text. It cannot edit or generate a `.docx` file, so no Word tracked change is ever applied automatically.
- **Flags coverage gaps, never invents a rating:** a clause type the playbook in use doesn't cover is marked UNRATED, not guessed at.
- **Attorney review gate:** presents every review with an explicit confirmation step before marking it ready to use in negotiation. Never sends, files, executes, or transmits anything anywhere.

Handles: contract clause review across common commercial clause types (limitation of liability, indemnification, termination, confidentiality, governing law, payment terms, IP assignment, warranties, non-compete, force majeure, assignment) — the review pass solo and small-firm attorneys otherwise pay $99–400/month for a dedicated tool to do.

## Setup

Install time: about 5 minutes. Download the zip, drag it into Claude Desktop's Extensions panel, and attach a workspace folder with the contract you want reviewed and (optionally) your firm's own `playbook.md`. No connectors to authorize. Open a new chat, type `/skills`, and verify `/contract-review` appears.

## Compliance

Requires Claude for Work, Claude Team, or Claude Enterprise. Do not use a consumer Claude plan (Claude Pro or Personal) with confidential client or contract information. Every output carries an "ASSISTED CONTRACT REVIEW — ATTORNEY REVIEW REQUIRED BEFORE USE" header and footer, shown around each review — never inside a redline the attorney copies into a working document. The skill never marks a review ready without your explicit confirmation, never determines a clause's enforceability or whether to sign, never invents contract language or playbook positions not in what you provided, and never sends, files, executes, or transmits anything itself.
