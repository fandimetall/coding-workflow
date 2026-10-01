# regression

Pin a fixed bug so it can never come back. The test IS the bug report,
executable. Write it before the fix.

## When to use

- Every bug fix, without exception
- A production incident that had a code cause
- A behavior someone reported twice
- Edge cases found by users that existing tests missed

## When NOT to use

- As a substitute for understanding the root cause (a pinned symptom can hide it)
- To pin behavior you *think* is intended but never verified

## The procedure (RED before GREEN)

1. **Reproduce** the bug as a failing test. Run it. It MUST fail, and it must
   fail for the reported reason — not on a typo or missing import.
2. **Record the failure output.** This is your proof the test is real.
3. **Fix** the code.
4. **Run again.** It must pass. Run the whole suite — nothing else may break.
5. **Commit the test with the fix**, same commit.

If step 1 does not fail, you have not reproduced the bug. Stop and re-read the report.

## Skeleton

```python
def test_order_total_excludes_cancelled_items_issue_481():
    """Regression #481: cancelled items were still counted in the total."""
    order = order_fixture(
        items=[item(sku="A", price=10), item(sku="B", price=5, status="cancelled")]
    )

    total = order.total()

    assert total == Decimal("10")   # was Decimal("15") before the fix
```

```ts
it('excludes cancelled items from total (#481)', () => {
  const order = orderFixture({
    items: [item({ sku: 'A', price: 10 }), item({ sku: 'B', price: 5, status: 'cancelled' })],
  });

  expect(order.total()).toBe(10); // was 15 before the fix
});
```

```go
func TestOrderTotalExcludesCancelled(t *testing.T) {
    // Regression #481
    o := orderFixture(item{SKU: "A", Price: 10},
                      item{SKU: "B", Price: 5, Status: "cancelled"})
    if got := o.Total(); got != 10 {
        t.Fatalf("total = %d, want 10 (issue #481)", got)
    }
}
```

```dart
test('excludes cancelled items from total (#481)', () {
  final o = orderFixture(items: [
    item(sku: 'A', price: 10),
    item(sku: 'B', price: 5, status: 'cancelled'),
  ]);
  expect(o.total(), 10); // was 15 before the fix
});
```

## Naming and traceability

- Put the issue id in the test name: `_issue_481` / `(#481)`.
- One line of comment stating the original symptom.
- Assert the *specific* corrected behavior, not just "doesn't crash".

## Where it belongs

Next to the code it protects, in the layer where the bug lived:

| Bug location | Regression test layer |
|---|---|
| Pure logic | unit |
| SQL / query | integration |
| API shape | contract |
| User flow | e2e (only if it only manifests there) |

## Pitfalls

- Writing the test *after* the fix and never confirming it fails → you may be
  pinning nothing. Always verify RED first.
- Asserting the wrong layer: an `e2e` test for a pure-logic bug is slow and hides the cause.
- Testing "no exception" instead of the corrected value → passes for the wrong reason.
- Pinning the symptom rather than the invariant → the bug returns in a new costume.
- No issue reference → nobody knows why the test exists. It gets deleted later.
