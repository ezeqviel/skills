# Common Workflows

## Issues

```bash
# Create an issue
gh issue create --title "bug: login breaks on Safari" --body "$(cat <<'EOF'
## Description
What's happening and how to reproduce it.

## Expected behavior
What should happen instead.
EOF
)"

# List open issues
gh issue list

# List issues with filters
gh issue list --label bug --assignee @me

# View an issue
gh issue view 42

# Close an issue
gh issue close 42
```

### Linking PRs to Issues

Use `closes #N` in the PR body to auto-close the issue when the PR is merged:

```bash
gh pr create --title "fix(auth): validate session token" --body "$(cat <<'EOF'
## What changed
- Added token validation on login

closes #42
EOF
)"
```

Also works with: `fixes #N`, `resolves #N`.

---

## PR Review

When reviewing someone else's PR:

```bash
# See what changed
gh pr diff 123

# Check CI status
gh pr checks 123

# Approve
gh pr review 123 --approve --body "LGTM"

# Request changes
gh pr review 123 --request-changes --body "The endpoint doesn't validate input"

# Just comment (no approval/rejection)
gh pr review 123 --comment --body "Why did this change?"

# View all comments on a PR
gh api repos/{owner}/{repo}/pulls/123/comments
```

---

## Stash

Save work-in-progress without committing — useful when you need to switch branches mid-task:

```bash
# Save current changes with a descriptive message
git stash push -m "halfway through login refactor"

# List all stashes
git stash list

# Recover the most recent stash (and remove it from the list)
git stash pop

# Recover a specific stash
git stash pop stash@{2}

# See what's in a stash without applying it
git stash show -p stash@{0}
```

---

## Investigation

Tools for understanding what happened and when.

### History

```bash
# Visual commit history
git log --oneline --graph --all

# History of a specific file
git log --oneline -- path/to/file

# Search commit messages
git log --grep="login" --oneline

# Commits by author
git log --author="eze" --oneline --since="2 weeks ago"
```

### Blame

Find who changed each line of a file and when:

```bash
git blame src/api.ts

# Blame a specific line range
git blame -L 10,20 src/api.ts
```

### Bisect

Binary search to find exactly which commit introduced a bug:

```bash
git bisect start
git bisect bad                # current commit is broken
git bisect good v1.0.0        # this version worked fine
# Git checks out a middle commit — test it and tell git:
git bisect good               # if this commit works
git bisect bad                # if this commit is broken
# Repeat until git finds the exact commit
git bisect reset              # go back to where you started
```
