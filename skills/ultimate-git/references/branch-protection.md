# Branch Protection

## Modern Rulesets (preferred)

```bash
gh ruleset list
gh ruleset view RULESET_ID
```

## Legacy API (full control)

```bash
# Main branch protection
gh api --method PUT repos/{owner}/{repo}/branches/main/protection \
  --input - <<'EOF'
{
  "required_status_checks": {"strict": true, "contexts": []},
  "enforce_admins": true,
  "required_pull_request_reviews": {
    "required_approving_review_count": 1,
    "dismiss_stale_reviews": true
  },
  "restrictions": null,
  "allow_force_pushes": false,
  "allow_deletions": false
}
EOF

# Develop branch protection
gh api --method PUT repos/{owner}/{repo}/branches/develop/protection \
  --input - <<'EOF'
{
  "required_status_checks": {"strict": true, "contexts": []},
  "enforce_admins": true,
  "required_pull_request_reviews": {
    "required_approving_review_count": 1,
    "dismiss_stale_reviews": true
  },
  "restrictions": null,
  "allow_force_pushes": false,
  "allow_deletions": false
}
EOF

# View current protection
gh api repos/{owner}/{repo}/branches/main/protection --jq '.'
```
