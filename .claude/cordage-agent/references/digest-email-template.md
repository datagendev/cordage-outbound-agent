# Digest Email Template

Your output IS the email. Follow this format.

## Template

```
{Greeting},

{N} companies received FDA warning letters with document control / data integrity citations recently.

---

**LEAD 1: {company_name}** [{timing_window}]

**Letter Date:** {date}
**Violations:** {violation_types}
**FDA Letter:** {letter_url}
**Company:** {location} | ~{headcount} employees
**Contact:** {contact_name}, {contact_title}
**LinkedIn:** {contact_linkedin_url}

**Draft email:**
> Subject: {subject}
>
> {line_1}
>
> {line_2}
>
> {line_3}

---

[repeat for each lead]

---

REPLY TO ACTION:
- Approve: "approve 1, 3" or "approve all"
- Reject: "reject 2 -- reason"
- Deep dive: "more on 1" (fetches full letter text)
- Monitor: "watch {company_name}"

---

Scanned FDA.gov for new CGMP letters. {raw} letters found -> {cordage_relevant} had doc control/data integrity citations -> {qualified} qualified after enrichment.

Let me know which ones hit -- and a quick reason why or why not.
```

## Rules

1. Number every lead.
2. Always include [URGENT], [ACTIVE], or [PREP] timing tag.
3. Always link to the actual FDA warning letter.
4. Full LinkedIn URLs for contacts.
5. Draft email must match the timing window.
6. Zero new letters? Say "no new CGMP letters this week" and ask if they want to widen the search.
