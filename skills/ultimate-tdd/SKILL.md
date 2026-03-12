---
name: ultimate-tdd
description: "Test-Driven Development execution guide. Drives feature development through disciplined test-first cycles: write a failing test, make it pass with minimal code, then refactor. Use this skill when the user asks to build something with TDD, test-driven development, or the RED-GREEN-REFACTOR methodology. Does NOT apply to general testing requests like 'write tests' or 'add test coverage' — those are testing tasks, not TDD."
---

# Ultimate TDD

Build features test-first through disciplined RED-GREEN-REFACTOR cycles. Each cycle adds one small behavior, keeping the codebase working at every step.

## Operating Rules

1. **Always run tests** — never assume they pass. Execute the test command and read the output.
2. **One test at a time** — never write multiple tests before seeing them fail.
3. **Minimal production code** — write only what the current failing test demands. Nothing more.
4. **Respect existing structure** — before creating test files, find where tests already live. If a test file exists for the module you're working on, add to it.
5. **No automatic commits** — the user decides when to commit.
6. **Phase names stay out of git** — RED, GREEN, REFACTOR are useful conversation terms, but commit messages should describe actual changes (e.g. "add email validation", not "RED: write failing test for email").
7. **Not for retroactive testing** — this skill is for building new functionality test-first. Adding tests to existing untested code is a different activity.

## Phase 0: Project Reconnaissance

Before writing any test, understand the project's testing setup. This runs once at the start.

1. **Detect the test framework and runner.** Check config files (`package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`, `Gemfile`, etc.) and look for runner configs (`jest.config.*`, `vitest.config.*`, `pytest.ini`, `.rspec`). See [references/framework-detection.md](references/framework-detection.md) for the full detection guide.

2. **Find the test directory and naming conventions.** Look at existing test files to understand the pattern — colocated with source? Separate `tests/` or `__tests__/` directory? What's the file naming pattern (`*.test.ts`, `*_test.go`, `test_*.py`)?

3. **Check for existing tests on the target module.** If you're adding a feature to `src/auth/login.ts`, search for `login.test.ts`, `login.spec.ts`, or similar. If one exists, you'll add your tests there.

4. **If anything is unclear — ask.** Don't guess where tests should go. Ask the user.

## The Cycle

After reconnaissance, repeat these three phases for each behavior you need to implement. Start with the simplest possible case and increase complexity with each cycle.

### RED — Write a Failing Test

1. Identify the next behavior to test. If this is the first cycle, pick the simplest happy path.
2. Write exactly **one test** following the Arrange-Act-Assert pattern:
   - **Arrange** — set up test data and dependencies
   - **Act** — call the code under test
   - **Assert** — verify the expected outcome
3. Run the test suite.
4. Verify the test fails **for the expected reason** — a missing function, a wrong return value, an unmet condition. Not a syntax error, not an import failure, not a misconfigured test runner.
5. If it fails for the wrong reason, fix the infrastructure problem first and re-run. The test must fail because the behavior doesn't exist yet, not because the test itself is broken.

### GREEN — Make It Pass

Write the **minimum production code** to make the failing test pass. This is YAGNI (You Aren't Gonna Need It) in action — don't write code for cases that aren't tested yet.

The Transformation Priority Premise (Robert C. Martin) helps decide what "minimum" means. Prefer the simplest transformation that works, in this order:

| Priority | Transformation | Example |
|----------|---------------|---------|
| 1 | `{}` → nil | Return null/nil/None |
| 2 | nil → constant | `return 3` |
| 3 | constant → variable | `return a` |
| 4 | unconditional → conditional | Add an `if` |
| 5 | scalar → collection/data structure | Use a list, map, or domain object |
| 6 | statement → iteration | Add a loop |

If a constant makes it pass, don't add an `if`. If an `if` makes it pass, don't add a loop. The next test will force you to generalize if needed.

Rules:
- No refactoring in this phase.
- Don't handle cases that aren't tested yet.
- Hardcoded return values are fine if they're genuinely the minimum.
- Run the **full test suite**, not just the new test. If an existing test broke, investigate the regression — don't hack around it.

### REFACTOR

Improve the code without changing behavior. All tests must stay green.

Look for:
- **Duplication** — extract shared code
- **Unclear names** — rename to reveal intent
- **Unnecessary complexity** — simplify logic
- **Structural issues** — improve organization

Rules:
- One small change at a time.
- Run all tests after each change.
- If a test fails after refactoring, revert and try a smaller change.
- This phase is **optional** per cycle. If the code is already clean, move on to the next RED.

## Incrementality

Each cycle adds one behavior. Progress from simple to complex:

| Order | Category | Example |
|-------|----------|---------|
| 1 | Trivial happy path | `add(1, 2)` returns `3` |
| 2 | Happy path variations | `add(0, 0)` returns `0`, `add(-1, 1)` returns `0` |
| 3 | Edge cases | Empty input, null, boundary values |
| 4 | Error handling | Invalid arguments, missing data, malformed input |
| 5 | Complex scenarios | Multiple collaborating components, state transitions |

After each complete cycle, state what test comes next and why — so the progression is clear and deliberate.

## Triangulation

When a test passes with a hardcoded value (e.g. `return 3`), write a second test with a different input to force generalization. This is the mechanism that turns `return 3` into `return a + b`.

Triangulation answers the question: "Do I need to generalize this code, or is the simple version enough?" If one test passes with a constant, maybe a constant is all you need. When a second test demands a different result, the constant breaks and you're forced to write real logic.

Use triangulation deliberately — it's how you avoid both over-engineering (generalizing too early) and under-engineering (leaving hardcoded values that should be logic).

## Test Naming

Test names should describe **behavior**, not implementation details.

**First priority:** match the project's existing naming convention. Consistency matters more than any specific style.

**If no convention exists**, use descriptive names that read as sentences:

| Bad | Good |
|-----|------|
| `testAdd` | `adds_two_positive_numbers` |
| `test1` | `returns_zero_when_input_is_empty` |
| `testErrorHandling` | `throws_validation_error_for_negative_age` |
| `testProcess` | `skips_already_processed_items` |

A good test name tells you what broke without reading the test body.

## When to Use TDD

TDD is not always the right approach. Use judgment:

| Scenario | TDD Value | Notes |
|----------|-----------|-------|
| New feature | High | TDD drives the design |
| Bug fix | High | Write a test that reproduces the bug first, then fix |
| Complex business logic | High | Tests document the rules |
| Exploratory / prototyping | Low | Spike first to learn, then TDD the real implementation |
| UI layout / styling | Low | Visual output is hard to assert meaningfully |
| Glue code / simple wiring | Low | If there's no logic to test, TDD adds ceremony without value |

If the user asks for TDD in a low-value scenario, mention the tradeoff but follow their lead.

## Framework Quick Reference

| Language | Runner | Run command |
|----------|--------|-------------|
| TypeScript/JS | Jest / Vitest | `npx jest` / `npx vitest run` |
| Python | pytest | `python -m pytest` |
| Go | go test | `go test ./...` |
| Ruby | RSpec | `bundle exec rspec` |
| Rust | cargo | `cargo test` |
| C# / .NET | xUnit / NUnit / MSTest | `dotnet test` |
| C++ | Google Test / Catch2 | `ctest` |
| Swift | XCTest | `swift test` |
| Dart / Flutter | flutter_test | `flutter test` |
| Elixir | ExUnit | `mix test` |
| Haskell | Hspec / Tasty | `cabal test` / `stack test` |

For full detection logic and conventions per framework, see [references/framework-detection.md](references/framework-detection.md).

## Common Pitfalls

| Mistake | What to do instead |
|---------|-------------------|
| Writing multiple tests before running any | Write one, run it, see it fail |
| Test fails on import error and you move on | Fix the import first — the test must fail for the right reason |
| Writing "clever" code in GREEN | Write dumb, obvious code. Cleverness comes in REFACTOR |
| Creating a new test file when one exists | Find the existing test file for that module and add to it |
| Skipping test execution ("it should pass") | Always run. Surprises are where bugs hide |
| Over-engineering in GREEN | Apply Transformation Priority Premise — use the simplest change |
| Generalizing after one test | Use triangulation — wait for a second test to force generalization |
