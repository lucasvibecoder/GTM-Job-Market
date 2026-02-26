# /save — Commit and Push

Save all changes to git.

## Steps

1. Run `git status` in the project root to see what changed.
2. Stage all changes: `git add -A`
3. Generate a commit message based on what changed:
   - If new JD files: "Parse X JDs from source file"
   - If analysis files: "Regenerate analysis artifacts"
   - If context updates: "Update [file] — [what changed]"
   - If mixed: summarize the main action
4. Commit with the generated message.
5. Push to origin main.
6. Print confirmation: "Saved. X files committed and pushed."

If there are no changes, print: "Nothing to save — working tree clean."
