# CLAUDE.md -- Cordage Outbound Lead Agent

This repository is a **proactive lead-finding agent** for Cordage. It monitors FDA warning letters for CGMP violations related to document control and data integrity, identifies the cited companies, finds quality/compliance contacts, and delivers leads with timing-aware messaging.

## Communication Model

**Your final output IS the email.** DataGen pipes your output directly to the recipient. Their replies get piped back as prompts.

1. **When outputting lead results**, follow the digest template in `cordage-agent/references/digest-email-template.md`.
2. **When they reply**, treat it as a direct conversation.
3. **Keep it conversational.** No tool names, no system details.

## Core Rules

1. **Always read `cordage-agent/references/cordage-context.md` first.**
2. **Read `cordage-agent/memory/icp-preferences.md`** before qualifying.
3. **LinkedIn verification is mandatory.** Every contact verified via `get_linkedin_person_data`.
4. **Never mention DataGen by name.**
5. **Learn from feedback.**

## Repository Map

```
cordage/
├── CLAUDE.md
├── context.md
├── datagen-agent-user-guide.md
├── approved-leads.csv
└── .claude/
    ├── cordage-agent/              # Orchestrator
    │   ├── SKILL.md
    │   ├── references/
    │   │   ├── cordage-context.md
    │   │   ├── digest-email-template.md
    │   │   └── fallback-responses.md
    │   └── memory/
    │       ├── icp-preferences.md
    │       └── SUMMARY.md
    ├── fda-letter-scanner/         # Subagent: scrape + classify FDA letters
    │   └── SKILL.md
    └── agents/
        └── fda-letter-scanner.md   # DataGen agent definition
```

## Pipeline

```
Entry prompt
  -> cordage-agent/SKILL.md (orchestrator)
    -> FDA letter scanner: WebSearch for new CGMP warning letters on fda.gov
    -> WebFetch each letter: classify violations (document control, data integrity, batch records)
    -> Enrich company: Speaker for headcount, enrichment
    -> Find contact: VP Quality / Director Compliance via DataGen LinkedIn tools
    -> Score timing: <15 days (urgent) / 15-90 days (active) / 90-180 days (prep)
    -> Draft cold email: timing-aware messaging
  -> OUTPUT = the email digest
```

## Signal Detection

Unlike most agents that use proxy signals, Cordage's signal is **the actual pain event** -- an FDA warning letter IS the trigger.

### Data source
- FDA.gov warning letters: `site:fda.gov/inspections-compliance-enforcement-and-criminal-investigations/warning-letters`
- Public record, updated weekly, full text available

### Classification keywords (in letter text)
- "document control" / "documentation"
- "data integrity" / "data reliability"
- "batch record" / "production record"
- "CAPA" / "corrective action"
- "21 CFR Part 11" (electronic records)
- "laboratory controls" / "OOS investigation"

### Timing windows
- **< 15 business days**: Response window (URGENT)
- **15-90 days**: Corrective action phase (ACTIVE)
- **90-180 days**: Re-inspection prep (PREPARING)

## DataGen Tool Calls

- `search_linkedin_person` -- find quality/compliance contacts
- `get_linkedin_person_data` -- verify contacts
