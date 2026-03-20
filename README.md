# Cordage FDA Warning Letter Lead Agent

A proactive lead-finding agent for [Cordage](https://cordage.io), an AI-powered compliance and document management platform for regulated industries. The agent monitors FDA warning letters for CGMP violations related to document control and data integrity, identifies cited companies, finds quality/compliance contacts, and delivers timing-aware lead digests.

## How It Works

```
1. WebSearch FDA.gov for new CGMP warning letters
2. WebFetch each letter, classify violations (document control, data integrity, batch records, CAPA)
3. Filter for Cordage-relevant citations (skip labeling-only, adulteration-only)
4. Enrich companies (headcount, location)
5. Find VP Quality / Director of Compliance via DataGen LinkedIn tools
6. Score timing window: URGENT (<15 days) / ACTIVE (15-90) / PREP (90-180)
7. Draft cold emails matched to timing window
8. Output numbered lead digest
```

## Why FDA Warning Letters

Unlike most outbound agents that use proxy signals, Cordage's signal is **the actual pain event** -- an FDA warning letter IS the trigger. No inference needed.

- FDA publishes warning letters as public record on fda.gov
- CDER issued 111 GMP warning letters in FY2025 (up 50% YoY)
- Companies have **15 business days** to respond with a corrective action plan
- Regulatory violations are the #1 reason CDMOs lose bids (26%)
- 3-5 new Cordage-relevant letters per week

## Pipeline

| Step | Tool | What it does |
|------|------|-------------|
| **FDA letter scanner** | WebSearch + WebFetch | Finds new CGMP letters, extracts violation types from full text |
| **Violation classifier** | Text analysis | Flags document control, data integrity, batch records, CAPA, 21 CFR Part 11 |
| **Company enrichment** | Speaker / WebSearch | Headcount, location, confirms pharma manufacturing |
| **Contact finder** | DataGen LinkedIn tools | VP Quality, Director Compliance, Head Regulatory Affairs |
| **cordage-agent** | Orchestrator | Chains pipeline, scores timing, drafts emails, processes feedback |

## Digest Format

```
LEAD 1: Chemspec Chemicals [ACTIVE]

Letter Date: December 23, 2025
Violations: Document control, data integrity, batch records, CAPA
FDA Letter: https://fda.gov/...
Company: Navi Mumbai, India | ~200 employees
Contact: [VP Quality], Quality Assurance
LinkedIn: https://linkedin.com/in/...

Draft email:
> Subject: Before re-inspection
>
> The 483 response bought Chemspec time, but the re-inspection clock is ticking.
> Most quality teams underestimate how long it takes to close every finding
> with evidence trails that hold up.
>
> Am I close, or are you already ahead of it?
```

## Timing-Aware Messaging

| Window | Days Since Letter | Email Angle |
|--------|------------------|-------------|
| **URGENT** | 0-15 business days | "The 15-day clock is loud" |
| **ACTIVE** | 15-90 days | "Re-inspection is coming" |
| **PREP** | 90-180 days | "Re-inspection ready?" |

## Feedback Loop

Reply to the digest to take action:

| Command | Example | What happens |
|---------|---------|-------------|
| Approve | `approve 1, 3` | Saved to approved-leads.csv |
| Reject | `reject 2 -- too large` | Logged, agent learns |
| Deep dive | `more on 1` | Fetches full letter text + deeper research |
| Monitor | `watch Acme Pharma` | Tracks future FDA actions |

The agent learns from feedback -- rejection patterns become filter rules, approved profiles sharpen targeting.

## Deployment

Deployed on [DataGen](https://datagen.dev) as a Claude Code agent.

```bash
# Deploy
datagen agents deploy <agent-id>

# Configure secrets
datagen agents config <agent-id> --secrets "DATAGEN_API_KEY,CLAUDE_CODE_OAUTH_TOKEN"

# Trigger a run
datagen agents run <agent-id> --payload '{"message": "Scan for new FDA CGMP warning letters"}'

# Schedule weekly
datagen agents schedule <agent-id> --cron "0 9 * * 1" --timezone "America/Los_Angeles" --name "Cordage weekly FDA scan"

# Check logs
datagen agents logs <agent-id>
```

## Repo Structure

```
cordage/
├── CLAUDE.md                     # Agent navigation guide
├── README.md                     # This file
├── context.md                    # Cordage company context
├── datagen-agent-user-guide.md   # Business context
├── approved-leads.csv            # Approved leads output
└── .claude/
    ├── agents/
    │   └── fda-letter-scanner.md # Agent definition (DataGen discovers this)
    ├── cordage-agent/            # Orchestrator
    │   ├── SKILL.md
    │   ├── references/
    │   │   ├── cordage-context.md
    │   │   ├── digest-email-template.md
    │   │   └── fallback-responses.md
    │   └── memory/
    │       ├── icp-preferences.md
    │       └── SUMMARY.md
    └── fda-letter-scanner/       # Subagent skill
        └── SKILL.md
```
