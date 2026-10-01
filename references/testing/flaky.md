# flaky

A test that passes and fails without any code change. It is a bug — in the
test, or in the code the test is exposing. Never "fix" it with retries.

## Root causes (check in this order)

1. **Time** — `now()`, timezone, DST, sleep-based waits, TTL boundaries
2. **Order** — tests share state; pass alone, fail in suite (or vice versa)
3. **Randomness** — unseeded `random()`, `uuid()`, hash iteration order
4. **Concurrency** — parallel workers hitting shared rows/files/ports
5. **Async** — not awaiting, race between assertion and completion
6. **Network** — real third-party calls without retry/timeout control
7. **Resource** — port already in use, disk full, temp dir collision
8. **Float/precision** — exact equality on computed decimals

## Diagnosis procedure

```
# 1. Run it in isolation
pytest tests/test_x.py::test_y -q

# 2. Run the whole suite (order dependence)
pytest -q

# 3. Force a different random order
pytest -q -p no:randomly --randomly-seed=12345

# 4. Repeat N times to measure the flake rate
pytest tests/test_x.py::test_y -q --count=50
```

Then bisect: freeze time, fix the seed, serialize the test, stub the network —
one at a time.

## Fixes by cause

| Cause | Fix |
|---|---|
| Time | Inject a clock; `freezegun` / `vi.useFakeTimers()`; never `sleep` |
| Order | Reset state per test; unique DB rows; no module-level mutable state |
| Randomness | Seed the RNG in the fixture; sort before asserting |
| Concurrency | Unique per-worker data; separate DB/schema/port per worker |
| Async | `await` properly; use framework `pumpAndSettle` / `waitFor` helpers |
| Network | Stub it, or add explicit timeout + retry policy in the test |
| Float | Compare with tolerance: `pytest.approx`, `toBeCloseTo` |

## Repro detection

```python
@pytest.mark.flaky(reruns=5, reruns_delay=0)  # DIAGNOSTIC ONLY, never merge this
```

Use repeat-runs to *measure* the flake rate, then remove the marker and fix the
cause. A merged `reruns=` marker is technical debt with interest.

## Commands

| Stack | Repeat to measure flakiness |
|---|---|
| Python | `pytest -q --count=50` (pytest-repeat) |
| JS/TS | `npx vitest run --retry=0` in a loop; `--sequence.shuffle` |
| Go | `go test -count=50 -race ./...` |
| Flutter | `flutter test --repeat=50` |

## Pitfalls

- Retrying to green → the flake stays; you just stopped seeing it.
- `sleep(2)` to "stabilize" → slower and still flaky under load.
- Debugging by adding prints without reproducing first.
- Quarantining a flaky test and never returning to it. Set a deadline.
