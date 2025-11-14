---
agent: agent
---
Perform git commit, using `COMMIT_MESSAGE_TEMPLATE` as the commit message template.

# Commit Workflow
1. identify all file changes pending commit (both staged and unstaged)
2. determine logical groupings of changes that should be committed together (e.g., related to a single feature or fix)
    - for each grouping, generate a concise, descriptive commit message following conventional commit style (e.g., feat:, fix:, docs:, style:, refactor:, test:, chore:)
    - Refer to `.github/COMMIT_MESSAGE_TEMPLATE.md` for commit message guidelines.
3. execute the git commit command(s) with the generated message(s)