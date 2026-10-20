---
name: ai-sales-team
description: "Use when researching prospects, qualifying leads (BANT/MEDDIC), finding decision makers, writing outreach/follow-up emails, preparing meetings, building proposals, handling objections, ICP building, competitive intel, or sales pipeline reports. Adapted from zubair-trabzada/ai-sales-team-claude for Hermes Agent."
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [sales, prospecting, b2b, outreach, lead-qualification]
    related_skills: [article-writing]
---

# AI Sales Team — Hermes Agent

## Overview

Sales intelligence system adapted for Hermes Agent (origin: Claude Code repo
`zubair-trabzada/ai-sales-team-claude`). Research any company, qualify leads with
BANT + MEDDIC, map the buying committee, generate personalized outreach, prepare
meetings, build proposals, and produce pipeline reports (Markdown or PDF).

Subagent role files live in `agents/` (spawn with `delegate_task`).
Email/proposal/objection templates live in `templates/`.
Helper scripts live in `scripts/` (optional; see Dependencies).

## Command Reference

User says anything like "sales prospect https://acme.com" or "riset prospect acme.com".
Router below — Hermes has no slash commands, so the orchestrator routes by intent:

| Intent (user phrasing) | Flow | Output file |
|---|---|---|
| "prospect X" / full audit | 5 parallel agents (below) | PROSPECT-ANALYSIS.md |
| "quick X" / snapshot 60 detik | single agent, condensed, no file | terminal reply |
| "research X" | agent: `agents/sales-company.md` | COMPANY-RESEARCH.md |
| "qualify X" / BANT / MEDDIC | agent: `agents/sales-opportunity.md` | LEAD-QUALIFICATION.md |
| "contacts X" / decision maker | agent: `agents/sales-contacts.md` | DECISION-MAKERS.md |
| "competitors X" | agent: `agents/sales-competitive.md` | COMPETITIVE-INTEL.md |
| "outreach X" / cold email | `templates/outreach-*.md` style | OUTREACH-SEQUENCE.md |
| "followup X" | follow-up sequence template | FOLLOWUP-SEQUENCE.md |
| "prep X" / meeting brief | `templates/meeting-prep.md` | MEETING-PREP.md |
| "proposal X" | `templates/proposal-template.md` | CLIENT-PROPOSAL.md |
| "objections X" | `templates/objection-playbook.md` | OBJECTION-PLAYBOOK.md |
| "ICP X" / ideal customer | ICP builder (below) | IDEAL-CUSTOMER-PROFILE.md |
| "sales report" / pipeline | report builder (below) | SALES-REPORT.md |

## Full Prospect Flow (`/sales prospect` equivalent)

1. Load `agents/` role file content into 5 `delegate_task` calls (toolsets: `['web','terminal']`),
   all in one batch:
   - **sales-company** → firmographics, growth signals, tech stack
   - **sales-contacts** → decision makers, org mapping, personalization anchors
   - **sales-opportunity** → BANT qualification, pain points, budget signals
   - **sales-competitive** → current solutions, switching costs, positioning
   - **sales-strategy** → outreach strategy, messaging, channel recommendation
2. Aggregate into Prospect Score 0-100:

| Category | Weight |
|---|---|
| Company Fit | 25% |
| Decision Maker Access | 20% |
| Opportunity Strength (BANT) | 25% |
| Competitive Position | 15% |
| Timing/Strategy | 15% |

   Grade: A 85+ (invest significant effort), B 70-84 (worth pursuing), C 55-69
   (nurture passively), D <55 (deprioritize).
3. Write `PROSPECT-ANALYSIS.md` with score bar, per-agent sections, recommended next action.

Subagents have no conversation memory — pass the URL, the ICP context (if known),
and the full role-file text in `context`. Ask each to return its section in Markdown,
ready to paste.

## Quick Snapshot (no file)

Single web search + read of the site. Reply in terminal only:
company summary (2-3 lines), fit guess, 3 personalization hooks, one outreach angle.
Skip agents and files.

## ICP Builder

Interview-lite: from user's description produce
target segments, firmographics ranges, trigger events, disqualifiers,
value hypothesis, outreach angles. Write `IDEAL-CUSTOMER-PROFILE.md`.
Then reuse it as context for every later prospect run.

## Pipeline Report

- Scan current dir for `*-ANALYSIS.md`, `DECISION-MAKERS.md`, `LEAD-QUALIFICATION.md`.
- If none, ask user for pipeline data (or a CRM export file path).
- Output `SALES-REPORT.md`: stage table, totals, weighted forecast, risks, next actions.
- PDF: `python scripts/generate_pdf_report.py` (see Dependencies) — or fall back to Markdown-only and say why.

## Templates

Read the matching `templates/*.md` for structure before writing output files.
Language: match user's language for chat replies; default output files to the
prospect's likely business language (or user's language if unspecified).

## Scripts (optional accelerators)

| Script | Purpose | Usage |
|---|---|---|
| `lead_scorer.py` | deterministic BANT scoring from JSON answers | `python scripts/lead_scorer.py answers.json` |
| `contact_finder.py` | query-based decision-maker search helpers | `python scripts/contact_finder.py "company"` |
| `analyze_prospect.py` | scrape site → structured summary JSON | `python scripts/analyze_prospect.py https://acme.com` |
| `generate_pdf_report.py` | Markdown report → PDF | `pip install reportlab` first |

Prefer the agent/web flow when scripts' deps are missing — scripts are accelerators,
not requirements.

## Dependencies

| Dependency | Install | Fallback |
|---|---|---|
| `web` toolset | enabled | cannot research; ask user for data |
| `delegate_task` | enabled | run agents sequentially in same session |
| `reportlab` | `pip install reportlab` | skip PDF, deliver Markdown |
| requests/bs4 (scripts) | `pip install -r` from upstream repo | skip scripts, use web tools |

## Portability

Self-contained. Install on any Hermes instance:
```bash
cp -r ai-sales-team ~/.hermes/skills/custom/
```

## Common Pitfalls

1. **Don't invent contact emails.** Agents may hallucinate addresses; verify via
   website/LinkedIn search, mark unverified contacts explicitly.
2. **Scoring is a rubric, not a measurement.** State assumptions behind each
   category score; never present the number as fact.
3. **Five parallel agents on one small site** can duplicate fetches — pass the
   site content you already have in context when cheap to do so.
4. **PDF deps missing** — check `import reportlab` before promising PDF output.

## Verification Checklist

- [ ] All output files written to cwd with exact names from Command Reference
- [ ] Prospect Score breakdown shows all 5 category weights
- [ ] Unverified contact info marked as such
- [ ] User's language used in chat reply
