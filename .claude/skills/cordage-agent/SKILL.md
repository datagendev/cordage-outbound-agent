---
name: cordage-agent
description: "Orchestrator for the Cordage outbound lead agent. Monitors FDA warning letters for CGMP violations, classifies document control / data integrity citations, finds quality/compliance contacts, drafts timing-aware cold emails. TRIGGERS: (1) 'Run the agent' or 'find leads', (2) 'Watch {company}', (3) Feedback replies."
---

# Cordage Outbound Lead Agent

## Context
Read @.claude/skills/cordage-agent/references/cordage-context.md for Cordage product, ICP, pain segment.
Read @.claude/skills/cordage-agent/references/digest-email-template.md for lead output format.
Read @.claude/skills/cordage-agent/references/fallback-responses.md for partial result handling.

## Memory
Check @.claude/skills/cordage-agent/memory/icp-preferences.md before qualifying leads.
Check @.claude/skills/cordage-agent/memory/SUMMARY.md for last run context.

## Pipeline

```
1. Load context and memory
2. Scan FDA.gov for new CGMP warning letters
3. Fetch + classify each letter for Cordage-relevant violations
4. Enrich companies (headcount, location)
5. Find and verify quality/compliance contacts
6. Score timing window
7. Draft timing-aware cold emails
8. Compose numbered lead digest
```

## Tools & Subagents

| Step | Tool | Type |
|------|------|------|
| Scan FDA letters | WebSearch | Built-in (site:fda.gov) |
| Fetch letter text | WebFetch | Built-in |
| Classify violations | Text analysis | In-agent (keyword matching) |
| Enrich company | Speaker API | Via fda-letter-scanner subagent or direct query |
| Find contacts | DataGen `search_linkedin_person` | executeTool |
| Verify contacts | DataGen `get_linkedin_person_data` | executeTool |

## CSV Output

Approved leads append to: `approved-leads.csv`

Columns: `date,company,location,letter_date,letter_url,violations,timing_window,contact_name,contact_title,contact_linkedin,draft_subject,draft_email,status`

---

### Step 1: Load context and memory
- Read cordage-context.md, icp-preferences.md, SUMMARY.md
- Note the last run date from SUMMARY.md to avoid re-processing old letters

### Step 2: Scan FDA.gov for new CGMP warning letters

Run two WebSearch queries:

```
Query 1: "site:fda.gov/inspections-compliance-enforcement-and-criminal-investigations/warning-letters CGMP pharmaceutical {current_year}"
Query 2: "site:fda.gov/inspections-compliance-enforcement-and-criminal-investigations/warning-letters drug manufacturing {current_year}"
```

Collect all unique letter URLs. Dedup against letters already processed (check SUMMARY.md).

### Step 3: Fetch + classify each letter

For each letter URL, use WebFetch with this prompt:

```
Extract from this FDA warning letter:
1. Company name and location
2. Date of letter
3. List of violations cited
4. Does the letter mention any of these (yes/no for each):
   - document control
   - data integrity
   - batch records / production records
   - CAPA / corrective action
   - 21 CFR Part 11
   - laboratory controls
5. Company size if mentioned
Return as structured JSON.
```

**Classify as Cordage-relevant if** the letter mentions 1+ of:
- document control
- data integrity
- batch records
- CAPA
- 21 CFR Part 11

**Skip if** the letter ONLY mentions:
- labeling violations
- adulteration (contamination without doc issues)
- unapproved drug claims
- advertising violations

### Step 4: Enrich companies

For each Cordage-relevant company:
- WebSearch: `"{company_name}" employees manufacturing` to estimate size
- Speaker query (if available): look up company by name for headcount

**Qualify**:
- 50-500 employees (mid-size CDMOs, contract manufacturers)
- Pharmaceutical / biotech / API manufacturing
- Not a giant pharma (>5000 employees -- they have internal compliance teams)

**Disqualify**:
- < 20 employees (too small for Cordage)
- > 1000 employees (likely have compliance infrastructure)
- Non-pharma (cosmetics-only, food, supplements -- unless doc control citation)
- Check icp-preferences.md for additional rules

### Step 5: Find and verify contacts

Search for contacts at the cited company. Priority order:
1. VP Quality / VP Quality Assurance
2. Director of Quality / Director of Compliance
3. Head of Regulatory Affairs
4. Quality Manager / Compliance Manager
5. COO (if < 50 employees and no dedicated quality leader)

For each contact:
- `search_linkedin_person` by company name + title keywords
- `get_linkedin_person_data` to verify current role
- LinkedIn verification mandatory

### Step 6: Score timing window

Calculate business days between letter date and today:

| Window | Days Since Letter | Urgency | Messaging Angle |
|--------|------------------|---------|-----------------|
| URGENT | 0-15 business days | Highest | "The 15-day clock is loud" |
| ACTIVE | 15-90 days | High | "Re-inspection is coming" |
| PREP | 90-180 days | Medium | "Re-inspection ready?" |
| STALE | > 180 days | Low | Skip unless explicitly requested |

### Step 7: Draft cold emails

For each qualified lead, write a 3-line email matched to timing window:

**URGENT (< 15 days)**:
```
Subject: After the 483

{first_name} -- FDA just cited {company} for {violation_type}, and your corrective action plan is due in days.

Most teams scramble to patch document control manually. The ones that clear re-inspection fastest lock down the system before the inspector comes back.

Worth a 15-min look at how to close these gaps fast?
```

**ACTIVE (15-90 days)**:
```
Subject: Before re-inspection

{first_name} -- the 483 response bought {company} time, but the re-inspection clock is ticking. Most quality teams underestimate how long it takes to close every finding with evidence trails that hold up.

Am I close, or are you already ahead of it?
```

**PREP (90-180 days)**:
```
Subject: Re-inspection ready?

{first_name} -- companies that cleared re-inspection fastest after a 483 didn't just fix the findings. They automated the evidence trail so inspectors could self-serve the proof.

Is that on your radar for {company}?
```

### Step 8: Compose numbered lead digest

Follow digest-email-template.md. Include:
- Company name, location, letter date
- Link to FDA warning letter (source URL)
- Violation types cited
- Timing window (URGENT / ACTIVE / PREP)
- Contact name, title, LinkedIn URL
- Draft cold email

---

## Handling Feedback

### Lead Actions
- `"approve 1, 3"` -> Append to approved-leads.csv. Confirm saved.
- `"reject 2 -- too big"` -> Log to memory/icp-preferences.md.
- `"more on 1"` -> Fetch full letter text, deeper company research.
- `"watch {company}"` -> Add to watchlist for future FDA actions.

### ICP Tuning
- `"skip companies outside the US"` -> Add geo filter
- `"include medical device manufacturers too"` -> Expand beyond CDER
- `"only show urgent leads"` -> Filter to < 15 day window only

### Messaging Tuning
- `"too aggressive"` -> Soften tone
- `"reference the specific violation"` -> Include exact citation
- `"rewrite lead 3"` -> Custom rewrite

### Signal Tuning
- `"data integrity is the strongest signal"` -> Weight higher
- `"skip labeling-only letters"` -> Already default, confirm
- `"also monitor 483 observations"` -> Expand data sources

---

## CRITICAL: Output Format

**Your ENTIRE text output IS the email.** This means:
1. NEVER write meta-commentary
2. NEVER summarize what you did
3. Start with a greeting, go straight to leads
4. If no new letters found, use fallback template
5. End casually asking for feedback

## Reference Files
- [references/cordage-context.md](references/cordage-context.md)
- [references/digest-email-template.md](references/digest-email-template.md)
- [references/fallback-responses.md](references/fallback-responses.md)
