# /audit — Vault Health Check

Scan all JD and Dream files for data quality issues, promotion candidates, and staleness. Terminal output only — no files generated.

## Prerequisites
Read these files before starting:
- `CLAUDE.md` (field definitions, quality gates, file format)
- `Context/Tags.md` (known_tools list, company_type list)
- `Context/Archetypes.md` (valid archetype keys)
- All files in `JDs/` and `Dream/`

## Checks

### Check 1 — Promotion Candidates (skills → tools)
Identify named technologies hiding in `skills_required` and `skills_preferred` that should be on the `known_tools` list.

1. Scan `skills_required[]` and `skills_preferred[]` across **JDs/ only** (Dream excluded from counts — consistent with existing frequency rules).
2. Look for **named technologies and platforms** embedded in free-text strings. Examples:
   - `"Python and/or JavaScript fluency"` → extract Python, JavaScript
   - `"SQL fluency"` → extract SQL
   - `"Experience with React or Next.js"` → extract React, Next.js
3. Use **word-boundary matching** to avoid false positives:
   - "SQL" must NOT match "MySQL" or "PostgreSQL"
   - "Go" must NOT match "Google" or "going"
   - "R" should only match when it's clearly the language (standalone, in a list of languages)
4. Compound strings count each tech separately — `"Python and/or JavaScript"` = 1 Python + 1 JavaScript.
5. **Skip abstract concepts**: AI, CRM, automation, LLM, API, APIs, webhooks, ETL, A/B testing, machine learning, data warehouse. Only flag proper-noun technologies with specific product/language names.
6. **Subtract anything already on the `known_tools` list** — don't flag Clay just because it appears in a skills string.
7. Count how many JDs mention each technology (deduplicate per-JD — a tech appearing in both skills_required and skills_preferred of the same JD counts once).
8. **Flag any technology appearing in 3+ JDs**.
9. Print: name, count, which JDs mention it, suggested canonical name.

### Check 2 — Orphan Data
Validate all frontmatter values against their reference lists.

1. **Archetypes**: Every `archetype` and `archetype_secondary` value (non-null) in JDs/ and Dream/ must match a key in `Context/Archetypes.md`. List any mismatches.
2. **Tools**: Every value in `tools[]` must match a name in `Context/Tags.md` known_tools table. List any that don't.
3. **Company types**: Every value in `company_type[]` must match a key in `Context/Tags.md` company_type table. List any that don't.

### Check 3 — Missing Fields
Verify every JD and Dream file has complete frontmatter per CLAUDE.md §4 template.

1. Required fields present: `company`, `title`, `salary_low`, `salary_high`, `salary_type`, `url`, `location`, `work_style`, `company_stage`, `company_type`, `implicit_stack`, `yoe_required`, `archetype`, `archetype_secondary`, `tools`, `skills_required`, `skills_preferred`, `lucas_annotations`, `fit_score`, `date_parsed`.
2. Non-empty array check: `tools[]`, `skills_required[]`, `company_type[]` must each have at least 1 element.
3. List files and specific missing/empty fields.

### Check 4 — Links Section Integrity
Verify the `## Links` section in every JD and Dream file matches frontmatter.

1. Every file has a `## Links` section.
2. Links section has an `Archetype` line.
3. Links section has a `Company Type` line.
4. Links section has a `Tools` line when `tools[]` is non-empty.
5. Links section has a `Secondary` line when `archetype_secondary` is not null.
6. List files and specific failures.

### Check 5 — Stale Analysis
Compare JD freshness against Analysis/ files.

1. Find `date_parsed` of the newest JD in `JDs/`.
2. Get the modification dates of files in `Analysis/`.
3. If any JD has a `date_parsed` newer than the oldest Analysis/ file modification date, flag it.
4. Also flag any JD file that has no `date_parsed` set.
5. Print how many JDs were added since last analysis run and list them.

## Output Format
Print all results to terminal using this format:

```
/audit — Vault Health Check
═══════════════════════════

Promotion Candidates (skills → tools)
──────────────────────────────────────
⚡ {Tech} — {N} JDs (not in known_tools)
   └ {company1}, {company2}, ...
   └ Suggested canonical: "{Tech}"
✓ No other candidates above 3-JD threshold

Orphan Data
───────────
✓ All archetypes valid        (or list mismatches)
✓ All tools valid             (or list orphans)
✓ All company types valid     (or list orphans)

Missing Fields
──────────────
✓ All {N} JDs + {M} Dream files have complete frontmatter
                              (or list files + missing fields)

Links Integrity
───────────────
✓ All files pass              (or list failures)

Stale Analysis
──────────────
✓ Analysis is current         (or flag stale JDs)
```

Use `⚡` for promotion candidates, `⚠` for warnings/issues, `✓` for passing checks.

## What This Command Does NOT Do
- Does not modify any files
- Does not add tools to Tags.md
- Does not regenerate Analysis/ or Hubs/
- Only prints findings — user decides what to act on
