# /start — Session Launcher

Quick orientation for a new session. Show what's in the vault and what you can do.

## Steps

1. **Count files**:
   - `JDs/` — count .md files
   - `Dream/` — count .md files
   - `Analysis/` — count .md files
   - `Archive/` — count .md files

2. **Staleness check**:
   - If `Analysis/` files exist, compare their last-modified date to the most recent JD file. If any JD is newer than Analysis, flag: "Analysis is stale — run `/analyze` to refresh."
   - If `JDs/` is empty, flag: "No JDs parsed yet — run `/bulk-add` to import JDs."

3. **Print dashboard**:
```
## GTME Job Market Analyzer

| Folder | Count |
|---|---|
| JDs/ | X |
| Dream/ | X |
| Analysis/ | X |
| Archive/ | X |

[Staleness warnings if any]

## Commands
| Command | What it does |
|---|---|
| /bulk-add | Bulk import JDs from Clay CSV export |
| /add | Add a single new JD |
| /analyze | Regenerate Analysis/ artifacts |
| /gap | Personal fit/gap analysis |
| /save | Git commit + push |
```
