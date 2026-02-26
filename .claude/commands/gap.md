# /gap — Personal Fit & Gap Analysis

Generate fit scores and gap analysis using `Context/Profile.md` as the personal layer.

## Prerequisites
Read these files before starting:
- `Context/Profile.md` (skills, gaps, preferences, salary targets)
- `Context/Archetypes.md` (archetype definitions)
- `Context/Tags.md` (tool normalization)
- All files in `JDs/` and `Dream/` (read all frontmatter + requirements)

## Steps

### Step 1 — Score Each JD
For each JD (including Dream/), calculate a fit score (1-10) based on:

**Skill match (0-4 points)**:
- Count required skills Lucas has (from Profile.md Core Skills) vs total required
- Partial credit for "developing" skills (Growth Edges)
- 4 = matches all required, 0 = matches none

**Tool match (0-3 points)**:
- Count required tools Lucas knows vs total tools listed
- Weight by proficiency level (Expert=1.0, Advanced=0.8, Proficient=0.6, Basic=0.3, None=0)
- 3 = full stack coverage, 0 = no overlap

**Preference alignment (0-2 points)**:
- Building-focused role? (+0.5)
- AI-forward company? (+0.5)
- Early-stage/startup? (+0.5)
- No heavy dashboard/reporting requirement? (+0.5)

**Salary fit (0-1 point)**:
- Salary range overlaps with target ($140-180k)? = 1
- Below floor ($130k)? = 0
- Above target (stretch)? = 0.5

### Step 2 — Write fit_score to each JD
Update each JD file's frontmatter with the calculated `fit_score`.

### Step 3 — Generate Fit_Gap_Analysis.md
Write `Analysis/Fit_Gap_Analysis.md` with:

**Fit Leaderboard**:
Table of all JDs ranked by fit_score (descending):
```
| Rank | Company | Title | Fit Score | Top Gaps |
```

**Gap Frequency Analysis**:
Count how many JDs require each skill/tool Lucas lacks. Sort descending.
```
| Gap | JDs Requiring It | % | Priority |
```
Priority = High (60%+), Medium (25-59%), Low (<25%)

**"If You Learned X" Analysis**:
For each high-priority gap, calculate:
- How many JDs would see a score increase
- Which specific JDs unlock
- Estimated effort to close the gap (from Profile.md Growth Edges)

Format:
```
### If you learned [Skill/Tool]
- Unlocks: X additional JDs
- Score boost: +Y average across affected JDs
- Top unlocks: [list top 3 JDs by score increase]
- Effort estimate: [Low/Medium/High] — [reason]
```

**Dream Role Gap Map**:
For each Dream/ role, list every gap between Lucas's profile and the requirements. Be specific — not "needs coding" but "requires Python proficiency for building automation scripts + SQL for data modeling."

### Step 4 — Print Summary
```
Fit scores written to X JDs.

Top 5 by fit:
1. Company — Title (score)
...

Top 3 gaps to close:
1. [Gap] — unlocks X JDs
...

Dream role gap summary:
- Every.to: [primary gaps]
- xAI: [primary gaps]
- Tenex.co: [primary gaps]
```
