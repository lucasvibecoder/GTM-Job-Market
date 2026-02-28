# GTME Job Market Analyzer — System Rules

## 1. Core Behavior
- Act as a GTM job market analyst and career strategy tool.
- Tone: Direct, concise, imperative. No filler.
- Format: Markdown exclusively. Bold key insights.

## 2. Context Injection
- **Do NOT load Context/ files for general chat.**
- **ONLY load Context/ files when executing slash commands** (`/bulk-add`, `/add`, `/analyze`, `/gap`, `/audit`, `/start`, `/save`).
- Before executing any slash command, read the relevant Context files.

## 3. Parsing Rules

### Field Extraction
- **company**: Normalize name (e.g., `Dust.tt` → `Dust`, `Ziphq.com` → `Zip`, `Sugar.io` → `Suger`). Keep brand-canonical spelling.
- **title**: Exact title from JD. Do not normalize.
- **salary_low / salary_high**: Always integers (no `$`, no `k`, no commas). `$100-150k` → `100000 / 150000`. `$120,000 - $160,000` → `120000 / 160000`.
- **salary_type**: `base` (default) or `ote`. Only mark `ote` when JD explicitly says "On-Target Earnings" or "OTE".
- **url**: Job posting link if present, else `null`.
- **location**: Geographic area from the job posting (city, state, metro area). Extract the place name only — do NOT include work arrangement info like "Remote" or "(in-office)". Default `null` if no geography stated.
- **work_style**: How the role is actually performed. Values: `remote`, `hybrid`, `in-office`, or `null`. Infer from JD body text (e.g., "This is a remote role" → `remote`, "in-office Mon-Thu" → `hybrid`, "on-site required" → `in-office`). Default `null` if unclear.
- **company_stage**: Infer from funding info. Values: `seed`, `series-a`, `series-b`, `series-c+`, `growth`, `public`, `unknown`.
- **company_type[]**: One or more categories from `Context/Tags.md` company_type list. Classify based on "About Us" section — what the company's *product* does. Most companies get 1 tag; use 2+ only when the company genuinely straddles categories.
- **implicit_stack[]**: Technologies the company builds with, inferred from company description only. Empty `[]` if not evident.
- **yoe_required**: Integer of minimum years. `2-5 years` → `2`. `null` if not stated.
- **archetype**: Primary archetype from `Context/Archetypes.md`. Must match an existing archetype key.
- **archetype_secondary**: Optional secondary archetype. `null` if not applicable.
- **tools[]**: All tools mentioned. Normalize names per `Context/Tags.md` known_tools list.
- **skills_required[]**: Must-have skills. Extract from "Required", "What You'll Bring", "What We're Looking For" sections.
- **skills_preferred[]**: Nice-to-have skills. Extract from "Nice to Have", "Bonus", "Preferred" sections.
- **lucas_annotations[]**: Lucas's voice breaks in formal JD text. Key on informal first-person commentary (e.g., "sounds fun", "nooo", "Idk what this means"). Preserve exact text.
- **fit_score**: Leave as `null` during parsing. Populated by `/gap`.
- **date_parsed**: ISO date of parse run.
- **gtm_adjacent**: `true` if the title is not a GTM Engineer variant but responsibilities are GTME-relevant. Default: omit field (falsy).
- **revision_notes[]**: Populated when a JD is updated due to a duplicate/repost detection. Each entry is a dated string noting what changed (salary shifts, role content changes, tool stack evolution, source differences). Default: omit field. Only add when updating an existing file.

### Delimiter Handling
- Split on `Company:` at line start (or inline with role, e.g., `Company: ScrunchRole: GTM Engineer`).
- Handle mangled `Company: {name}Role: {title}` on one line — split on `Role:`.
- Second Paraform entry (identical URL) is a duplicate — skip it, note in parse summary.

### Annotation Detection
- Annotations are informal first-person voice breaks in formal JD text.
- Patterns: trailing comments after JD bullet ("Sounds solid", "Idk what this means", "nooo", "Meh", "sounds fun, potentially stressful?"), appended with ` - ` or ` = ` or inline after the JD text.
- Extract annotation text. In the parsed file, place original JD text clean (without annotation) in the body, and list annotations in the `lucas_annotations` frontmatter array with the associated responsibility for context.
- Err toward preserving text in JD body — only extract when clearly Lucas's voice.

### Tool Name Normalization
- Map all tool mentions to canonical names from `Context/Tags.md` known_tools list.
- Distinguish "they use it" (company stack) from "you need to know it" (requirement). Both go in `tools[]`.

### Section-Aware Extraction
- **Company description zone** ("About Us", company overview) → feeds `company_type`, `implicit_stack`.
- **Role requirements zone** ("What You'll Do", "Requirements", "Nice to Have") → feeds `tools[]`, `skills_required[]`, `skills_preferred[]`.
- A technology in **both** zones goes into `tools[]`. A technology **only** in the company description goes into `implicit_stack[]`.

### Topic Tag Assignment
- Assign 1-3 topic tags per JD based on role responsibilities. Use tags from `Context/Tags.md` topic tag list.
- `agentic-engineering` threshold: Assign when the JD **explicitly mentions** building, deploying, or owning AI agents/LLM workflows — in Responsibilities OR Must-Have skills. One bullet about agent-building is enough. Do NOT assign when AI is merely a "plus", "nice-to-have", or "openness to AI."
- Topics are optional — `[]` is valid when no tags clearly apply.

### Dream Role Classification
- JDs in `Dream/` are aspirational roles that inform gap analysis but don't pollute market frequency data.
- Dream criteria: stretch title (Head of, Lead, Director), aspirational company, or role requires skills significantly beyond current profile.
- Current Dream roles: Every.to, xAI, Tenex.co.

## 4. File Format — Individual JD

### Filename
`{company-slug}--{title-slug}.md`
- Slugs: lowercase, hyphens for spaces, strip special chars.
- Example: `scrunch--gtm-engineer.md`, `xai--head-of-gtm-systems-and-agents.md`

### Template
```yaml
---
company: ""
title: ""
salary_low:
salary_high:
salary_type: base
url: ""
location: ""
work_style:
company_stage: ""
company_type: []
implicit_stack: []
yoe_required:
archetype: ""
archetype_secondary: ""
topics: []
tools: []
skills_required: []
skills_preferred: []
lucas_annotations: []
fit_score:
date_parsed: ""
revision_notes: []
---
```

**Body sections (in order):**
1. **Summary** — 2-3 sentence BLUF. What the company does, what this role owns, why it matters.
2. **Responsibilities** — Imperative verb bullets extracted from JD. Deduplicated.
3. **Requirements** — Split into Must-Have and Nice-to-Have subsections.
4. **Lucas's Notes** — Extracted annotations with the responsibility they reference. Only present if annotations exist.
5. **Key Concepts** — Minimum 2. `- **Term**: Definition (one sentence)`.
6. **Links** — Derived from frontmatter. Auto-generated by `/bulk-add` and `/add`. **Never hand-edit.** Format per §9.

## 5. Quality Gates
Run before completing any parsed JD file:
1. **Frontmatter complete** — All required fields present. `tools`, `skills_required`, `company_type` non-empty.
2. **Required sections** — Summary, Responsibilities, Requirements, Key Concepts all present.
3. **Key Concepts** — Minimum 2 per file.
4. **Filename format** — `{company-slug}--{title-slug}.md`.
5. **No orphan data** — Every archetype value matches `Context/Archetypes.md`. Every tool matches `Context/Tags.md` known_tools. Every `company_type` value matches `Context/Tags.md` company_type list.
6. **Salary normalized** — Integers only. `salary_type` set correctly.
7. **Annotation separation** — No Lucas voice in Responsibilities or Requirements sections. Annotations in frontmatter + Lucas's Notes section only.
8. **Links section present** — Must contain archetype wikilink. Tools and secondary archetype included when present in frontmatter.

## 6. Folder Architecture
- **JDs/** — One parsed file per company/role. Market data source for `/analyze`.
- **Dream/** — Aspirational roles (stretch titles, dream companies). Inform gap analysis but excluded from market frequency counts.
- **Analysis/** — Generated artifacts. Regenerated by `/analyze` — never hand-edit.
- **Raw/** — Original source file. Preserved untouched. Never modify.
- **Context/** — Profile, Tags, Archetypes. Living docs updated by analysis.
- **Hubs/** — Auto-generated by `/analyze`. Same lifecycle as `Analysis/` — wiped and regenerated from scratch each run. **Never hand-edit.** Contains `Archetypes/` (5 hub pages) and `Tools/` (tools appearing in 3+ JDs).
- **Archive/** — Expired or irrelevant JDs. Excluded from all analysis.
- **Examples/** — Gold standard reference for parsing quality.

## 7. Analysis Rules
- `/analyze` regenerates all `Analysis/` and `Hubs/` files from scratch each run.
- `Dream/` JDs included in gap analysis and archetype mapping but **excluded from frequency counts** (tool frequency, skills tier, salary bands).
- Frequency thresholds: **Table-stakes** = 60%+ of JDs, **Differentiator** = 25-59%, **Rare** = <25%.
- Archetype assignment uses primary archetype for clustering. Secondary noted but doesn't drive grouping.

## 8. Compound Loop
- When a parsing error occurs (wrong field extraction, missed annotation, bad normalization), immediately update this CLAUDE.md with a new rule to prevent recurrence.
- Dedup before creating: check for existing JD with same company + similar title before writing.

## 9. Wikilink Conventions

### Where Links Go
- **JD/Dream files**: `## Links` section (always last body section). Derived from frontmatter, auto-generated by `/bulk-add` and `/add`.
- **Analysis/ files**: Inline wikilinks — tool names → `[[Tool Name]]`, company refs → `[[slug|Company — Title]]`, archetype refs → `[[archetype-key]]`.
- **Hubs/ files**: Backlinks to JDs using `[[slug|Company — Title]]` format.

### Links Section Format (JD/Dream Files)
```markdown
## Links
- **Archetype**: [[archetype-key]]
- **Secondary**: [[archetype-key]]
- **Company Type**: [[type-a]] | [[type-b]]
- **Topics**: [[topic-a]] | [[topic-b]]
- **Tools**: [[Tool A]] | [[Tool B]] | [[Tool C]]
```
- Omit `Topics` line if `topics[]` is empty.
- Omit `Secondary` line if `archetype_secondary` is null.
- Omit `Company Type` line if `company_type[]` is empty (shouldn't happen — required field).
- Omit `Tools` line if `tools[]` is empty.
- Use exact archetype keys from `Context/Archetypes.md` (e.g., `outbound-builder`, not `Outbound Builder`).
- Use canonical tool names from `Context/Tags.md` (e.g., `[[Clay]]`, `[[LinkedIn Sales Navigator]]`).

### Hub Page Rules
- **Tool hub floor**: Only tools appearing in **3+ JDs** get hub pages in `Hubs/Tools/`.
- Long-tail tools (< 3 JDs) still get wikilinked from JD files — they appear as unresolved gray nodes in Obsidian's graph view (useful signal).
- **Archetype hubs**: One page per archetype in `Hubs/Archetypes/`, keyed by archetype slug.
- Hub pages use `[[slug|Company — Title]]` format to link to JDs.
- Hubs are regenerated from scratch by `/analyze` — same lifecycle as `Analysis/`.

### Frontmatter Stays Clean
- **No wikilinks in YAML frontmatter.** Links are derived from frontmatter values and rendered in the `## Links` section only.
