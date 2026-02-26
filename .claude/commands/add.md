# /add — Add a Single JD

Add one new job description to the vault. Accepts pasted text or a URL.

## Prerequisites
Read these files before starting:
- `CLAUDE.md` (parsing rules, quality gates)
- `Context/Tags.md` (tool normalization)
- `Context/Archetypes.md` (archetype definitions)

## Input Modes

### Mode A — Pasted Text
User pastes raw JD text after invoking `/add`.
1. Parse the pasted text using CLAUDE.md §3 extraction rules.
2. If Company or Role is missing, ask.

### Mode B — URL
User provides a job posting URL.
1. Fetch the page content.
2. Extract the JD text from the page.
3. Parse using CLAUDE.md §3 extraction rules.
4. If the URL is from youtube.com, reject: "YouTube URLs aren't job postings."

## Pipeline

1. **Dedup check**: Search `JDs/`, `Dream/`, and `Archive/` for existing file with same company + similar title. If found, ask: "Found existing `[filename]`. Update it or create new?"

2. **Raw preservation**: Append the raw input text to `Raw/source-jds.md` with a timestamp delimiter:
   ```
   ---
   <!-- Imported: {ISO timestamp} via /add -->
   {raw input text}
   ```
   For Mode A, append the pasted text. For Mode B, append the fetched page text.

3. **Parse**: Extract all fields per CLAUDE.md §3. Classify archetype, normalize tools, detect annotations. Extract `company_type` and `implicit_stack` from company description section.

4. **Classify**: Ask user — "JDs/ or Dream/?" Default to JDs/ unless the role clearly matches Dream criteria (stretch title, aspirational company, significant skill gap).

5. **Write file**: Generate filename, write frontmatter + body per CLAUDE.md §4. Include `## Links` section derived from frontmatter per CLAUDE.md §9.

6. **Quality gates**: Run CLAUDE.md §5 checks.

7. **Staleness flag**: Print "Analysis is now stale — run `/analyze` to refresh."

8. **Print confirmation**:
```
Added: {filename}
Location: {JDs/ or Dream/}
Archetype: {primary} / {secondary}
Company Type: {values}
Tools: {count} detected
Salary: ${low} - ${high} ({type})
```
