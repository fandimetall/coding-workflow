# integration

Test real components wired together: your code plus a real DB, HTTP server,
queue, or filesystem. Boundaries are real; only *external third parties* get faked.

## When to use

- Repository/DAO layers against a real (containerized or temp) database
- HTTP handlers through the real framework router
- Migrations, transactions, rollback behavior
- Serialization round-trips against a real schema
- Anything where the bug lives in the *wiring*, not the logic

## When NOT to use

- Pure logic → `unit` (faster, more precise)
- The full user journey through a browser → `e2e`
- You're calling a third-party API you don't control → `test-doubles` + `contract`

## Rules

- Use the real thing, isolated: `tmp_path`, test containers, or a scratch DB.
- Each test gets a clean slate. No shared rows between tests.
- Assert on observable outcomes (rows written, status code, JSON body), not internals.
- Truncate/reset between tests, do not rely on test ordering.
- Keep the count low — integration tests are the slow middle layer.

## Skeleton

```python
@pytest.fixture
def db(tmp_path):
    conn = sqlite3.connect(tmp_path / "t.db")
    conn.executescript(SCHEMA)
    yield conn
    conn.close()


def test_create_order_persists_line_items(db):
    repo = OrderRepo(db)
    repo.create(order_with_two_items())

    rows = db.execute("SELECT sku FROM line_items").fetchall()
    assert [r[0] for r in rows] == ["A", "B"]
```

```ts
it('persists line items', async () => {
  const repo = new OrderRepo(pgPool);
  await repo.create(orderWithTwoItems());
  const { rows } = await pgPool.query('SELECT sku FROM line_items');
  expect(rows.map(r => r.sku)).toEqual(['A', 'B']);
});
```

```go
func TestCreateOrderPersistsLineItems(t *testing.T) {
    db := newTestDB(t) // t.Cleanup closes + truncates
    repo := NewOrderRepo(db)
    require.NoError(t, repo.Create(ctx, orderWithTwoItems()))
    var skus []string
    require.NoError(t, db.Select(&skus, "SELECT sku FROM line_items"))
    assert.Equal(t, []string{"A", "B"}, skus)
}
```

## Commands

| Stack | Run |
|---|---|
| Python | `pytest tests/integration -q` |
| JS/TS | `npx vitest run tests/integration` |
| Go | `go test ./... -run Integration` (guard with `-short` skip) |
| Flutter | `flutter test integration_test` (device/driver) |

## Pitfalls

- Pointing at a shared dev database → tests corrupt each other. Always isolate.
- Relying on auto-increment IDs you hardcoded → use returned IDs.
- Testing the framework instead of your wiring (e.g. asserting Express returns 200 for any handler).
- Forgetting `t.Cleanup` / fixture teardown → leaked state across runs.
