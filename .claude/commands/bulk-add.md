# /bulk-add — Bulk Import JDs from Clay CSV Export

Import multiple job descriptions from a Clay CSV export into the vault.

## Prerequisites
Read these files before starting:
- `CLAUDE.md` (parsing rules, quality gates, file format)
- `Context/Tags.md` (tool normalization, allowed tags)
- `Context/Archetypes.md` (archetype definitions)
- `Examples/Gold_Standard_JD.md` (reference output)

## Input
`$ARGUMENTS` = path to CSV file. The CSV must have a `Full Jd` column (match flexibly on casing/spacing: `full_jd`, `Full JD`, `full jd`, etc.). Each cell contains a structured text blob from Clay.

## Clay Format Parsing

Each `Full Jd` cell looks like:
```
CompanyName / domain.com
title: Actual Title (Fallback Title if "title" not present)
job url: https://...
salary: 130000 - 160000 currency: USD YEAR
date added: 2026-02-18T21:51:49.489Z
location: City or Metro Area
LinkedIn job id: 12345
Description: [full JD body text]
```

### Header Field Extraction
- **company**: Text before ` / ` on line 1. Normalize per CLAUDE.md §3.
- **title**: After `title: `. Strip Clay fallback suffix — remove `(... if "title" not present)` parenthetical from end.
- **url**: After `job url: `. If empty/missing → `null`.
- **salary_low / salary_high**: Parse `X - Y` from `salary:` line. Ignore `currency: USD YEAR` suffix. If `0 - 0` or missing → both `null`.
- **location**: After `location: `. Geographic only (per CLAUDE.md §3 — no "Remote" here).
- **date added**: Noted but not used as `date_parsed`. Use today's date for `date_parsed`.
- **LinkedIn job id**: Ignored (not in schema).
- **Description**: Everything after `Description: ` = JD body text. Feed into parse logic.

### Fields Inferred from Body Text
Same extraction as `/add` Step 2 and CLAUDE.md §3:
`work_style`, `company_stage`, `company_type`, `implicit_stack`, `yoe_required`, `archetype`, `archetype_secondary`, `tools`, `skills_required`, `skills_preferred`, `lucas_annotations`, `salary_type` (OTE only if body says so), `fit_score` (always `null`).

## Pipeline (per entry)

1. **Extract** `Full Jd` cell from CSV row.

2. **Parse Clay header** — extract company, title, url, salary, location from structured header lines.

3. **Within-batch dedup** — if the same company + similar title already appeared earlier in the CSV, skip. First one wins.

4. **Dedup check** — search `JDs/`, `Dream/`, `Archive/` for existing file with same company + similar title. If found → skip, count as dupe.

5. **Raw preservation** — append raw text to `Raw/source-jds.md` with delimiter:
   ```
   ---
   <!-- Imported: {ISO timestamp} via /bulk-add -->
   {raw cell text}
   ```

6. **Dream classification** — auto-classify to `Dream/` if title contains stretch keywords: `Head of`, `Director`, `VP`. Everything else → `JDs/`.

7. **Agency detection** — flag if body contains: `our client`, `on behalf of`, `confidential client`, `undisclosed company`. Don't skip — just flag in summary.

8. **Parse body** — extract all remaining fields from Description text using CLAUDE.md §3 rules (archetypes, tools, skills, company_type, annotations, etc.).

9. **Write file** — filename + frontmatter + body per CLAUDE.md §4. Include `## Links` section per CLAUDE.md §9.

10. **Quality gates** — run CLAUDE.md §5 checks. If a gate fails, count as failed and note the reason.

11. **Track orphan tools** — any tool not in Tags.md known_tools → add to orphan list.

## Summary Output

After processing all rows, print:
```
## /bulk-add Summary

Imported: X files (Y → JDs/, Z → Dream/)
Skipped: N duplicates
Failed: N (quality gate failures)

⚠ Possible agency postings:
- company--title.md — detected "on behalf of" in body

🏷 Dream-classified (stretch title):
- company--title.md — title matched "Head of" / "Director" / "VP"

🔧 Unrecognized tools (not in Tags.md):
- ToolName (seen in: company1, company2)
- ToolName2 (seen in: company3)
→ Add these to Tags.md? [list for bulk addition]

Analysis is now stale — run /analyze to refresh.
```

Omit any section that has zero entries (e.g., no agency postings → skip that block).

## Edge Cases
- **No salary listed or `0 - 0`**: Set `salary_low` and `salary_high` to `null`.
- **OTE salary**: Only when body text explicitly says "On-Target Earnings" or "OTE". Otherwise default `base`.
- **Missing sections**: If JD has no clear "Nice to Have" section, set `skills_preferred: []`.
- **Empty `Full Jd` cell**: Skip row, don't count as failure.
- **No `Full Jd` column found**: Error out immediately with: "CSV has no `Full Jd` column. Expected a column named `Full Jd` (case-insensitive)."
