# Branch Strategies

Two supported strategies. The skill auto-detects which one the repo uses based on whether a `develop` branch exists.

## GitHub Flow (simple)

Best for: personal projects, small teams, continuous deployment, anything without formal releases.

```
main ← production (protected, PR only)
  feat/*  ← features (from main, PR to main)
  fix/*   ← bugfixes (from main, PR to main)
```

### New Feature (GitHub Flow)

```bash
git checkout main && git pull origin main
git checkout -b feat/feature-name
# ... work + commit using Conventional Commits ...
git push -u origin feat/feature-name
gh pr create --base main --title "feat(scope): description" --body "$(cat <<'EOF'
## What changed
- Bullet list of concrete changes

## Why
- Business/technical reason

## Impact
- What works differently now
EOF
)"
```

### Release (GitHub Flow)

No release branch needed — tag directly on main:

```bash
git checkout main && git pull origin main
gh release create vX.Y.Z --target main --generate-notes --title "vX.Y.Z"
```

---

## Gitflow (full)

Best for: apps with formal release cycles, teams that need a staging area before production, projects where `main` = what's deployed.

```
main         ← releases + tags (protected, PR only)
  develop    ← stable dev (protected, PR only)
    feat/*   ← features (from develop, PR to develop)
    fix/*    ← bugfixes (from develop, PR to develop)
    hotfix/* ← urgent (from main, PR to main + develop)
```

### New Feature (Gitflow)

```bash
git checkout develop && git pull origin develop
git checkout -b feat/feature-name
# ... work + commit using Conventional Commits ...
git push -u origin feat/feature-name
gh pr create --base develop --title "feat(scope): description" --body "$(cat <<'EOF'
## What changed
- Bullet list of concrete changes

## Why
- Business/technical reason

## Impact
- What works differently now
EOF
)"
```

### Release (Gitflow)

Merge develop into main, then tag:

```bash
git checkout main && git pull origin main
git merge develop
git push origin main
gh release create vX.Y.Z --target main --generate-notes --title "vX.Y.Z"
```

### Hotfix (Gitflow only)

For urgent fixes that can't wait for the next release cycle:

```bash
git checkout main && git pull origin main
git checkout -b hotfix/fix-name
# ... fix + commit ...
git push -u origin hotfix/fix-name
gh pr create --base main --title "fix: hotfix description" --body "$(cat <<'EOF'
## What changed
- Description of the fix

## Why
- Urgency / impact of the bug

## Impact
- What this resolves
EOF
)"
# After merge to main, also merge main back to develop
```

---

## How to detect which strategy

```bash
git branch -r | grep -q 'origin/develop' && echo "Gitflow" || echo "GitHub Flow"
```

If `develop` exists → Gitflow. Otherwise → GitHub Flow. This check should happen before any branching or PR operation.
