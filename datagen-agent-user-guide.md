---
title: "Cordage Outbound Lead Agent -- User Guide"
status: active
created: 2026-03-20
updated: 2026-03-20
---

# Cordage Outbound Lead Agent

## What This Does

This agent monitors FDA warning letters for pharmaceutical manufacturing companies cited for document control, data integrity, and CGMP violations. These companies have 15 days to respond with a corrective action plan and face re-inspection -- creating an urgent buying window for Cordage's compliance platform.

## How It Works

### Weekly Lead Digest

The agent:

1. **Scans** FDA.gov for new CGMP warning letters (updated weekly)
2. **Fetches** each letter and classifies violations (document control, data integrity, batch records, CAPA)
3. **Filters** for Cordage-relevant violations (skip labeling-only, adulteration-only letters)
4. **Enriches** companies with headcount and location data
5. **Finds contacts**: VP Quality, Director of Compliance, Head of Regulatory Affairs
6. **Scores timing**: urgent (<15 days), active (15-90), prep (90-180)
7. **Drafts emails** matched to the timing window
8. **Delivers** a numbered digest

### Replying to Action

| Command | Example | What happens |
|---------|---------|-------------|
| Approve | `approve 1, 3` or `approve all` | Leads saved to CSV |
| Reject | `reject 2 -- too large` | Lead skipped, reason logged |
| Deep dive | `more on 1` | Agent fetches full letter text + deeper company research |
| Monitor | `watch Acme Pharma` | Agent checks for future FDA actions against this company |

### Timing-Aware Messaging

Each lead gets a cold email draft matched to how long ago the letter was issued:

- **Urgent (< 15 days)**: "FDA just cited you -- the clock is loud"
- **Active (15-90 days)**: "The 483 response bought time, but re-inspection is coming"
- **Prep (90-180 days)**: "Re-inspection ready?"

## Signal Types

1. **Document control failures** -- missing, incomplete, or falsified records
2. **Data integrity violations** -- retroactive entries, destroyed originals, uncontrolled records
3. **Batch record deficiencies** -- incomplete production records, missing QC review
4. **CAPA failures** -- no corrective action process or inadequate follow-through
5. **Laboratory control failures** -- incomplete test data, missing specifications
6. **21 CFR Part 11 violations** -- electronic record/signature compliance gaps
