---
name: fda-letter-scanner
description: "Scan FDA.gov for new CGMP warning letters, fetch each letter, classify violations for Cordage relevance (document control, data integrity, batch records, CAPA, 21 CFR Part 11). Returns structured data per letter. TRIGGERS: Called by cordage-agent orchestrator, or directly with 'scan FDA letters'."
---

# FDA Letter Scanner

> Finds and classifies FDA warning letters for Cordage-relevant compliance violations.

## Input

```json
{
  "date_range": "string (e.g. '2026', '2025', or 'last 6 months')",
  "already_processed": ["list of letter URLs to skip"]
}
```

## Output

```json
[
  {
    "company_name": "string",
    "location": "string",
    "letter_date": "YYYY-MM-DD",
    "letter_url": "string",
    "violations": {
      "document_control": true/false,
      "data_integrity": true/false,
      "batch_records": true/false,
      "capa": true/false,
      "cfr_part_11": true/false,
      "laboratory_controls": true/false
    },
    "cordage_relevant": true/false,
    "violation_summary": "string -- 1-2 sentences",
    "days_since_letter": N,
    "timing_window": "URGENT|ACTIVE|PREP|STALE"
  }
]
```

## How to Execute

### Step 1: Search for new letters

Use WebSearch:
```
Query: "site:fda.gov/inspections-compliance-enforcement-and-criminal-investigations/warning-letters CGMP {year}"
```

Run for current year and previous year. Collect all letter URLs.

Remove any URLs in `already_processed`.

### Step 2: Fetch and classify each letter

For each new letter URL, use WebFetch with prompt:

```
Extract from this FDA warning letter:
1. Company name and location
2. Date of letter
3. List all violations cited
4. For each of these, answer yes or no:
   - document control issues
   - data integrity issues
   - batch record / production record issues
   - CAPA / corrective action issues
   - 21 CFR Part 11 issues
   - laboratory control issues
5. One sentence summary of the most serious violation
Return as JSON.
```

### Step 3: Classify Cordage relevance

A letter is **Cordage-relevant** if it mentions ANY of:
- document control
- data integrity
- batch records
- CAPA
- 21 CFR Part 11

**Not relevant** if it ONLY mentions:
- labeling
- adulteration without documentation issues
- unapproved drug claims
- advertising

### Step 4: Score timing

Calculate business days from letter_date to today:
- 0-15 business days = URGENT
- 16-90 days = ACTIVE
- 91-180 days = PREP
- > 180 days = STALE (skip unless requested)

### Rate Limiting

- Max 10 WebFetch calls per run (FDA letters are long)
- Process newest letters first (most urgent)
- If > 10 new letters, process top 10 and note remainder for next run
