# unit

Test one unit of behavior in isolation. No DB, no network, no filesystem, no clock.

## When to use

- Pure functions: parsers, formatters, validators, calculators
- State machines and branching business rules
- Edge cases: empty input, boundary values, invalid types
- Anything you can call directly with plain arguments

## When NOT to use

- The logic only exists inside a DB query or HTTP handler → `integration`
- You need to prove two services talk correctly → `integration`
- You'd have to mock 3+ collaborators → the unit is too big; split it

## Rules

- One behavior per test. Name it after the behavior, not the function.
- No `if`, no `for` around assertions. Parametrize instead.
- No mocking what you own. If you need a fake, pass it in as a dependency.
- A unit test should run in milliseconds and never touch the disk.

## Skeleton

```python
def test_parse_rejects_negative_quantity():
    result = parse_order({"qty": -1})
    assert result.error == "qty must be positive"


@pytest.mark.parametrize("raw,expected", [
    ("1.5", Decimal("1.5")),
    ("1,5", Decimal("1.5")),
])
def test_parse_accepts_comma_and_dot(raw, expected):
    assert parse_amount(raw) == expected
```

```ts
it('rejects negative quantity', () => {
  expect(parseOrder({ qty: -1 }).error).toBe('qty must be positive');
});
```

```go
func TestParseRejectsNegative(t *testing.T) {
    got := ParseOrder(Order{Qty: -1})
    if got.Err == nil {
        t.Fatal("want error for negative qty")
    }
}
```

```dart
test('rejects negative quantity', () {
  expect(parseOrder({'qty': -1}).error, 'qty must be positive');
});
```

## Commands

| Stack | Run |
|---|---|
| Python | `pytest tests/unit -q` |
| JS/TS | `npx vitest run tests/unit` |
| Go | `go test ./... -run TestUnit -short` |
| Flutter | `flutter test test/unit` |

## Pitfalls

- Testing private helpers directly → brittle. Test through the public function.
- Asserting on log strings → breaks on every refactor.
- Sharing mutable fixtures across tests → order-dependent failures. See `fixtures.md`.
