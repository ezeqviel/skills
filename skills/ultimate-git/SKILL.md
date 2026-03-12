---
name: ultimate-git
description: 'GitHub & Git operations specialist with opinionated workflows. Use when the user asks to create, review, or merge PRs, manage branches, create releases or tags, work with issues, configure GitHub Actions, manage repo settings, scaffold new repositories, or perform any gh CLI operation. Triggers on requests involving pull requests, branch strategy, releases, GitHub workflows, repo configuration, conventional commits, pushing code, committing changes, creating tags, reviewing PRs, stashing changes, git blame, git log, or git bisect. Also triggers on Spanish equivalents: commitear, pushear, subir cambios, crear PR, hacer merge, crear tag, crear repo, revisar PR, ramas, guardar cambios, ver historial.'
---

# Ultimate Git

Opinionated Git & GitHub workflow skill built on the `gh` CLI. Provides branch strategy, PR conventions, repo scaffolding, and common workflows.

## Operating Rules

1. **Confirm repo context** before any operation: `gh repo view --json nameWithOwner -q .nameWithOwner`
2. **JSON output**: Use `--json` fields for programmatic output
3. **Heredoc for bodies**: Always use heredoc for PR/issue bodies to preserve formatting
4. **Non-destructive**: Never force-push to main/develop. Confirm before any destructive operation
5. **Pull before branching**: Never create a branch without pulling latest from parent first
6. **Conventional Commits**: All commits and PR titles follow the spec. See [references/conventional-commits.md](references/conventional-commits.md) for the full type table and examples

## Branch Strategy

Two modes — auto-detect before any branching or PR operation:

```bash
git branch -r | grep -q 'origin/develop' && echo "Gitflow" || echo "GitHub Flow"
```

**GitHub Flow** (no `develop` branch):
```
main ← production (protected, PR only)
  feat/*  ← from main, PR to main
  fix/*   ← from main, PR to main
```

**Gitflow** (`develop` branch exists):
```
main         ← releases + tags (protected, PR only)
  develop    ← stable dev (protected, PR only)
    feat/*   ← from develop, PR to develop
    fix/*    ← from develop, PR to develop
    hotfix/* ← from main, PR to main + develop
```

For detailed workflows per strategy (feature, release, hotfix), see [references/branch-strategies.md](references/branch-strategies.md).

## Merging a PR

Before merging, review commit history: `git log --oneline BASE..HEAD`

Use `AskUserQuestion` to let the user choose the merge strategy. Always explain the reasoning behind each option:

```
header: "Merge strategy"
question: "How to merge PR #NUM? (X commits: [summarize commit quality])"
options:
- { label: "--merge (Recommended)", description: "Preserves the full commit history — useful for traceability and understanding what happened" }
- { label: "--squash", description: "Condenses all commits into one clean commit — useful when the intermediate history (wip, fix typo, etc.) doesn't add value" }
```

Default recommendation is `--merge`. Only suggest `--squash` first when commits are clearly messy (wip, wip2, fix typo, aaa, etc.).

Then run: `gh pr merge PR_NUM --[strategy] --delete-branch`

## Repo Init

1. **Ask**: Language? Test command? Install command? Public or private? GitHub Flow or Gitflow?
2. **Init**: `git init && git checkout -b main` (+ `git checkout -b develop` if Gitflow)
3. **Generate**: `.gitignore` (language-appropriate) + `.github/workflows/ci.yml` (see [references/ci-templates.md](references/ci-templates.md))
4. **Commit**: `git add -A && git commit -m "chore: initial project scaffold"`
5. **Remote**: `gh repo create NAME --source=. --push --public|--private`
6. **Default branch**: if Gitflow, `gh api repos/{owner}/{repo} --method PATCH -f default_branch=develop`
7. **Protection**: Apply rules for main (and develop if Gitflow). See [references/branch-protection.md](references/branch-protection.md)

## Common Workflows

For issues, PR review, stash, and investigation (log/blame/bisect), see [references/workflows.md](references/workflows.md).

## Error Resolution

| Error                   | First Action                                                 |
| ----------------------- | ------------------------------------------------------------ |
| `403 Forbidden`         | Check token scopes: `gh auth status`                         |
| `Merge conflicts`       | Resolve locally: `git pull origin BASE && git merge --no-ff` |
| `Status checks failing` | `gh pr checks PR_NUM` — wait or investigate CI               |
| `Branch protected`      | Create PR instead of direct push                             |
| `Not found`             | Verify repo: `gh repo view` and branch exists                |

Diagnose systematically: auth → repo context → branch state → permissions.
