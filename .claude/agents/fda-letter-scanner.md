---
name: fda-letter-scanner
description: Scan FDA.gov for CGMP warning letters with document control and data integrity violations. Classify, enrich, find quality/compliance contacts, and draft timing-aware cold emails for Cordage.
---

# FDA Letter Scanner Agent

Read and follow the orchestrator skill at `.claude/cordage-agent/SKILL.md` for the full pipeline.

## Quick Summary

1. WebSearch FDA.gov for new CGMP warning letters
2. WebFetch each letter, classify violations (document control, data integrity, batch records, CAPA)
3. Enrich companies (headcount, location)
4. Find VP Quality / Director Compliance via DataGen LinkedIn tools
5. Score timing window (URGENT / ACTIVE / PREP)
6. Draft cold emails matched to timing
7. Output numbered lead digest

## Entry Prompts

- "Run the agent" or "find leads" -- scan for new FDA letters
- "Watch {company}" -- monitor specific company for future FDA actions
- Feedback: "approve 1, 3", "reject 2 -- reason", "more on 1"

## Required Secrets

- DATAGEN_API_KEY
