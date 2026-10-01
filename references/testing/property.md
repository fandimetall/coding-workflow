# property

State an invariant that must hold for *all* inputs, then let the framework
generate hundreds of cases trying to falsify it. Finds the edge cases you
would never think to write.

## When to use

- Round-trips: `decode(encode(x)) == x`, `parse(format(x)) == x`
- Money and arithmetic: `total == sum(items)`, no negative balances
- Sorting / filtering: output is ordered and a subset of input
- Auth and invariants: no token grants access it shouldn't
- Parsers and serializers with wide input space
- Anything where "the obvious examples" already pass but you suspect edges

## When NOT to use

- Behavior with no describable invariant (most UI)
- Slow operations — property runs 100+ cases by default
- As a replacement for a couple of explicit named examples (keep those too)

## Rules

- One invariant per test. Name it as a sentence: "decode inverts encode".
- Let the framework shrink: a failure must be reported as the *minimal* input.
- Seed the generator and record the seed so a failure is reproducible.
- Combine with a handful of hand-written examples for documentation value.

## Skeleton — Python (hypothesis)

```python
from hypothesis import given, strategies as st, settings

@given(st.text())
def test_encode_decode_roundtrip(s):
    assert decode(encode(s)) == s


@given(st.lists(st.integers(min_value=1, max_value=10_000)))
def test_total_is_sum_of_items(amounts):
    order = build_order(amounts)
    assert order.total == sum(amounts)
    assert order.total > 0


@settings(deadline=None, max_examples=200)
@given(st.decimals(min_value=0, max_value=1_000_000))
def test_no_negative_balance(amount):
    acct = Account(balance=Decimal("0"))
    acct.deposit(amount)
    acct.withdraw(amount)
    assert acct.balance == 0          # not -0.0001
```

## Skeleton — JS/TS (fast-check)

```ts
import fc from 'fast-check';

test('decode inverts encode', () => {
  fc.assert(
    fc.property(fc.string(), (s) => {
      expect(decode(encode(s))).toBe(s);
    }),
  );
});

test('total equals sum of items', () => {
  fc.assert(
    fc.property(fc.array(fc.integer({ min: 1, max: 10_000 })), (amounts) => {
      expect(buildOrder(amounts).total).toBe(amounts.reduce((a, b) => a + b, 0));
    }),
  );
});
```

## Commands

| Stack | Tool | Run |
|---|---|---|
| Python | `hypothesis` | `pytest tests/test_props.py -q` |
| JS/TS | `fast-check` | `npx vitest run tests/props` |
| Go | `testing/quick`, `gopter` | `go test -run Property` |
| Flutter | `glados` | `flutter test test/props` |

Reproduce a known failure:

```bash
pytest test_props.py --hypothesis-seed=12345
```

## Pitfalls

- Writing a property that only re-implements the code → tautology, always passes.
- Too-broad generators (`st.text()` for a field that must be an email) → noise failures.
- Ignoring the shrunk counterexample and patching the generator to hide it.
- Flaky properties from unseeded randomness or time dependence.
- Using property tests to cover what 3 example tests would cover better.
