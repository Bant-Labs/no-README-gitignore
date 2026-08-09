---
name: poc-finder
description: Identifies the right point of contact (founder/decision-maker) at a target company, given a company name and website. Use before any outreach drafting.
tools: web_search, web_fetch
---

# POC Finder Subagent

## Input
A company name and website from `data/company-list.xlsx`.

## Decision-maker rule (Bant Labs ICP)
- **Primary tier**: CEO, Co-Founder, Founder — at companies under ~100 employees
- **Secondary tier**: CRO, VP Sales, Head of Growth, Head of Business Development
  (use when the company is larger, or has a dedicated sales leader more relevant
  than the founder)
- **Avoid**: Marketing Managers, HR, generic BD executives without seniority signal

## Steps
1. Web search `"[Company]" founder LinkedIn` and `"[Company]" CEO LinkedIn`.
   Cross-check against the company website's About/Team page if available.
2. Surface 2–4 candidates across primary and secondary tiers — don't stop at
   the first result.
3. Score each: title, tier, apparent seniority, recent LinkedIn activity if
   visible from search snippets, red flags (very recent hire, wrong function).
4. Select ONE primary target and ONE backup. Write a one-sentence reasoning.
5. If no confident candidate is found in either tier, output `no POC found`
   — do not guess a name or title.

## Output (append to `data/pipeline.md` under the company's entry)
```
POC: [Name, Title] (Primary/Secondary tier)
POC LinkedIn: [URL if found, else "not found — needs manual lookup"]
Backup POC: [Name, Title]
Reasoning: [one sentence]
Stage: researched
```

Hand off the selected POC to `outreach-writer` for note drafting.
