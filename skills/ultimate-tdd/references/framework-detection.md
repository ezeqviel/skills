# Framework Detection

How to identify the testing framework, test location, and naming conventions for a project.

## Detection Strategy

Follow this order. Stop as soon as you have a clear answer:

1. **Config files** — `package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, `Gemfile`, `pom.xml`
2. **Runner configs** — `jest.config.*`, `vitest.config.*`, `pytest.ini`, `.rspec`, `phpunit.xml`
3. **Existing test files** — look at their location, naming, and imports to infer the framework
4. **Ask the user** — if signals conflict or nothing is found

## Detection by Language

### Node.js / TypeScript

| Signal | Framework | Test dir | File pattern | Run command |
|--------|-----------|----------|--------------|-------------|
| `jest` in devDependencies | Jest | `__tests__/` or colocated | `*.test.ts`, `*.spec.ts` | `npx jest` or `npm test` |
| `vitest` in devDependencies | Vitest | `__tests__/` or colocated | `*.test.ts`, `*.spec.ts` | `npx vitest run` |
| `mocha` in devDependencies | Mocha | `test/` | `*.test.js`, `*.spec.js` | `npx mocha` |
| `@playwright/test` in devDependencies | Playwright | `tests/` or `e2e/` | `*.spec.ts` | `npx playwright test` |

Check `package.json` scripts for the test command:
```bash
cat package.json | grep -A2 '"test"'
```

### Python

| Signal | Framework | Test dir | File pattern | Run command |
|--------|-----------|----------|--------------|-------------|
| `pytest` in requirements or pyproject.toml | pytest | `tests/` | `test_*.py` | `python -m pytest` |
| No pytest dependency | unittest | `tests/` | `test_*.py` | `python -m unittest discover` |

Check pyproject.toml for test configuration:
```bash
cat pyproject.toml | grep -A5 '\[tool.pytest'
```

### Go

| Signal | Framework | Test dir | File pattern | Run command |
|--------|-----------|----------|--------------|-------------|
| `go.mod` exists | go test (stdlib) | Colocated with source | `*_test.go` | `go test ./...` |
| `testify` in go.mod | go test + testify | Colocated with source | `*_test.go` | `go test ./...` |

Go tests always live next to the source file they test. No separate test directory.

### Ruby

| Signal | Framework | Test dir | File pattern | Run command |
|--------|-----------|----------|--------------|-------------|
| `rspec` in Gemfile | RSpec | `spec/` | `*_spec.rb` | `bundle exec rspec` |
| `minitest` in Gemfile | Minitest | `test/` | `*_test.rb` | `bundle exec rake test` |

### Rust

| Signal | Framework | Test dir | File pattern | Run command |
|--------|-----------|----------|--------------|-------------|
| `Cargo.toml` exists | cargo test | Inline `#[cfg(test)]` modules or `tests/` for integration | `*_test.rs` or inline | `cargo test` |

Rust unit tests typically live in the same file as the code, inside a `#[cfg(test)]` module. Integration tests go in a top-level `tests/` directory.

### Java / Kotlin

| Signal | Framework | Test dir | File pattern | Run command |
|--------|-----------|----------|--------------|-------------|
| `junit` in pom.xml or build.gradle | JUnit | `src/test/java/` | `*Test.java` | `mvn test` or `gradle test` |

### PHP

| Signal | Framework | Test dir | File pattern | Run command |
|--------|-----------|----------|--------------|-------------|
| `phpunit/phpunit` in composer.json | PHPUnit | `tests/` | `*Test.php` | `./vendor/bin/phpunit` |

### C# / .NET

| Signal | Framework | Test dir | File pattern | Run command |
|--------|-----------|----------|--------------|-------------|
| `xunit` in .csproj | xUnit | Separate test project (`*.Tests/`) | `*Tests.cs` | `dotnet test` |
| `NUnit` in .csproj | NUnit | Separate test project (`*.Tests/`) | `*Tests.cs` | `dotnet test` |
| `MSTest` in .csproj | MSTest | Separate test project (`*.Tests/`) | `*Tests.cs` | `dotnet test` |

.NET convention is a separate test project mirroring the source project structure (e.g. `MyApp/` and `MyApp.Tests/`).

### C++

| Signal | Framework | Test dir | File pattern | Run command |
|--------|-----------|----------|--------------|-------------|
| `gtest` or `GTest` in CMakeLists.txt | Google Test | `tests/` or `test/` | `*_test.cpp`, `test_*.cpp` | `ctest` or `./build/tests` |
| `Catch2` in CMakeLists.txt or conan | Catch2 | `tests/` or `test/` | `*_test.cpp`, `test_*.cpp` | `ctest` or `./build/tests` |

Check CMakeLists.txt for test configuration:
```bash
grep -i 'gtest\|catch2\|enable_testing' CMakeLists.txt
```

### Swift

| Signal | Framework | Test dir | File pattern | Run command |
|--------|-----------|----------|--------------|-------------|
| Xcode project or Package.swift | XCTest | `Tests/` | `*Tests.swift` | `swift test` or `xcodebuild test` |

Swift Package Manager projects use `Tests/` at the root. Xcode projects use a `*Tests` target.

### Dart / Flutter

| Signal | Framework | Test dir | File pattern | Run command |
|--------|-----------|----------|--------------|-------------|
| `pubspec.yaml` with `flutter` | flutter_test | `test/` | `*_test.dart` | `flutter test` |
| `pubspec.yaml` without `flutter` | dart test | `test/` | `*_test.dart` | `dart test` |

### Elixir

| Signal | Framework | Test dir | File pattern | Run command |
|--------|-----------|----------|--------------|-------------|
| `mix.exs` exists | ExUnit | `test/` | `*_test.exs` | `mix test` |

ExUnit is built into Elixir — no extra dependency needed. Test files mirror the `lib/` structure inside `test/`.

### Haskell

| Signal | Framework | Test dir | File pattern | Run command |
|--------|-----------|----------|--------------|-------------|
| `hspec` in .cabal or package.yaml | Hspec | `test/` | `*Spec.hs` | `cabal test` or `stack test` |
| `tasty` in .cabal or package.yaml | Tasty | `test/` | `*Test.hs`, `*Tests.hs` | `cabal test` or `stack test` |
| `HUnit` in .cabal or package.yaml | HUnit | `test/` | `*Test.hs` | `cabal test` or `stack test` |

Check the test suite definition in the .cabal file or package.yaml for the test entry point.

## Finding Existing Tests for a Module

Given a source file, search for its corresponding test file:

```bash
# For a file like src/auth/login.ts, try these patterns:
find . -type f \( -name "login.test.*" -o -name "login.spec.*" -o -name "test_login.*" -o -name "login_test.*" \) 2>/dev/null

# Also check __tests__ directories:
find . -path "*/__tests__/login*" 2>/dev/null

# For Go, just look next to the source file:
ls src/auth/login_test.go 2>/dev/null
```

If a test file exists, add your new tests there. If not, create one following the project's existing naming convention.
