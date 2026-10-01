# test-doubles

Replace a real dependency with a controlled stand-in. Five kinds, different
purposes. Choosing the wrong one is the most common testing mistake.

## The five kinds

| Kind | What it does | Use for |
|---|---|---|
| **Dummy** | Passed but never used | Filling required params |
| **Stub** | Returns canned answers | Forcing a code path (error, empty) |
| **Spy** | Records calls, real or stubbed | Asserting an interaction happened |
| **Mock** | Pre-programmed expectations | Verifying behavior *contract* |
| **Fake** | Working lightweight impl | In-memory DB, fake mail server |

Prefer **fake > stub > mock**. Mocks couple tests to implementation.

## When to use

- Third-party API you don't control (payment, email, SMS, LLM provider)
- Nondeterministic dependencies: `now()`, `uuid()`, `random()`, network
- Expensive resources: external LLM calls, paid APIs
- Simulating failures: timeouts, 500s, malformed responses

## When NOT to use

- **Never mock what you own.** Mock the boundary, test your code for real.
- Don't mock the thing under test.
- Don't stub a DB — use a real temp DB (`integration.md`). Stubbed DBs hide SQL bugs.
- If you need 4+ mocks for one test, the unit has too many collaborators. Refactor.

## Rules

- Mock at the *edge*: HTTP client, clock, random source.
- Assert on the message you send, not on internal call counts.
- A fake should behave like the real thing for the contract you rely on.
- Keep fakes in one shared place (`tests/fakes/`), not duplicated per test.

## Skeleton

```python
def test_charge_retries_on_timeout(monkeypatch):
    calls = []

    def fake_post(url, **kw):
        calls.append(url)
        if len(calls) == 1:
            raise TimeoutError
        return FakeResponse(200, {"id": "ch_1"})

    monkeypatch.setattr(gateway.requests, "post", fake_post)

    result = charge_with_retry(amount=100)

    assert result.ok
    assert len(calls) == 2  # retried exactly once
```

```ts
it('retries once on timeout', async () => {
  const post = vi.fn()
    .mockRejectedValueOnce(new Error('timeout'))
    .mockResolvedValueOnce({ status: 200, data: { id: 'ch_1' } });

  const result = await chargeWithRetry({ amount: 100 }, { post });

  expect(result.ok).toBe(true);
  expect(post).toHaveBeenCalledTimes(2);
});
```

```go
type fakeGateway struct{ calls int }

func (f *fakeGateway) Charge(ctx context.Context, amt int) (string, error) {
    f.calls++
    if f.calls == 1 {
        return "", context.DeadlineExceeded
    }
    return "ch_1", nil
}

func TestChargeRetriesOnce(t *testing.T) {
    fg := &fakeGateway{}
    res, err := ChargeWithRetry(context.Background(), fg, 100)
    require.NoError(t, err)
    assert.True(t, res.OK)
    assert.Equal(t, 2, fg.calls)
}
```

## Commands

| Stack | Tool |
|---|---|
| Python | `unittest.mock`, `monkeypatch`, `pytest-mock` |
| JS/TS | `vi.fn()` (Vitest), `jest.fn()` |
| Go | hand-written fakes, `testify/mock` |
| Flutter | `mockito`, `mocktail` |

## Pitfalls

- `patch("module.func")` where the name is wrong → silently patches nothing.
- Over-mocking: the test passes, production breaks, because the real thing behaves differently.
- Asserting call order across unrelated collaborators → brittle.
- Fakes that drift from the real API and never get updated.
