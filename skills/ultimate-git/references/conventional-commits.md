# Conventional Commits Reference

All commits and PR titles MUST follow the [Conventional Commits](https://www.conventionalcommits.org/) specification:

```
type(scope): description

[optional body]

[optional footer(s)]
```

## Types

| Type       | Purpose                         | Semver impact |
| ---------- | ------------------------------- | ------------- |
| `feat`     | New functionality               | MINOR         |
| `fix`      | Bug fix                         | PATCH         |
| `docs`     | Documentation only              | -             |
| `style`    | Formatting, no logic change     | -             |
| `refactor` | Code restructuring, no feat/fix | -             |
| `perf`     | Performance improvement         | PATCH         |
| `test`     | Adding or correcting tests      | -             |
| `ci`       | CI/CD configuration changes     | -             |
| `chore`    | Maintenance, deps, build        | -             |
| `revert`   | Revert a previous commit        | -             |

## Scope (optional)

Parenthetical context for the change: `feat(scraper):`, `fix(db):`, `refactor(session):`

## Breaking Changes

- Short form: `feat!: remove legacy API` (append `!` after type)
- Long form: add footer `BREAKING CHANGE: description of what breaks`

## Examples

```
feat(scraper): extract barcodes from Bultos tab
fix(db): prevent duplicate shipments on re-scrape
chore: upgrade electron to v38.2
ci: add playwright browser cache to workflow
refactor(session): simplify lazy initialization logic
feat!: change shipment archive threshold to 30 days
```
