# coverage

A measurement of which code ran during tests. Useful as a *gap finder*,
dangerous as a *goal*. 100% coverage means every line executed, not that
anything was verified.

## When to use

- Finding untested branches in critical modules (auth, money, parsing)
- Tracking a coverage floor on changed lines (diff/patch coverage)
- Auditing after a refactor: did the old behavior stay covered?
- Onboarding: revealing which modules have zero tests

## When NOT to use

- As a merge gate at a fixed global percentage → incentivizes assertion-free tests
- To compare teams or projects
- As a substitute for thinking about edge cases

## Reading the report

- **Line** — executed or not. Cheapest, weakest.
- **Branch** — both sides of each `if` taken. This is the useful one.
- **Diff/patch** — coverage of only the lines you changed. Best practical gate.

Aim for high branch coverage in the *risky* modules, not uniformly.

## Commands

```bash
# Python
pytest --cov=src --cov-branch --cov-report=term-missing
pytest --cov=src --cov-report=html          # open htmlcov/index.html

# JS/TS (Vitest)
npx vitest run --coverage
npx vitest run --coverage --coverage.reporter=html

# Go
go test -coverprofile=cover.out ./...
go tool cover -func=cover.out
go tool cover -html=cover.out

# Flutter
flutter test --coverage
# then convert lcov.info to html with genhtml
```

## Diff coverage (recommended gate)

```bash
# only fail if NEW/CHANGED lines are under 80%
diff-cover coverage.xml --compare-branch=main --fail-under=80
```

## Config example

```toml
# pyproject.toml
[tool.coverage.run]
branch = true
source = ["src"]

[tool.coverage.report]
exclude_lines = [
  "pragma: no cover",
  "if TYPE_CHECKING:",
  "raise NotImplementedError",
  "if __name__ == .__main__.:",
]
```

## Pitfalls

- `# pragma: no cover` sprinkled to hit a number → the gap moves, it doesn't close.
- Tests that call code but assert nothing → coverage goes up, confidence doesn't.
- Counting generated code, migrations, or `__init__` re-exports.
- Chasing the last 5% (rare error paths) while risky branches sit at 40%.
- Coverage of `e2e` runs mixed with unit coverage → inflates the number.
