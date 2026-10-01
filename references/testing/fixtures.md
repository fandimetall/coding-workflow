# fixtures

Reusable setup/teardown for tests. The right amount of reuse makes suites fast
and readable; too much makes them magic and order-dependent.

## When to use

- The same setup appears in 3+ tests (a DB, a client, a logged-in user)
- You need guaranteed teardown (temp files, containers, monkeypatches)
- You want parametrized variants of a resource (small/large dataset)
- Building domain objects with sensible defaults (factories)

## When NOT to use

- Setup used by exactly one test → inline it; a fixture adds indirection
- When the fixture hides the *interesting part* of the test
- Deep fixture chains (fixture → fixture → fixture) → the test becomes unreadable

## Scopes

| Scope | Lifetime | Use for | Cost |
|---|---|---|---|
| function | per test (default) | anything mutable | safest |
| class | per class | shared read-only config | ok |
| module | per file | expensive setup (browser, container) | leak risk |
| session | whole run | server process, DB container | isolation risk |

Prefer the narrowest scope that is still fast enough.

## Skeleton — pytest

```python
@pytest.fixture
def db(tmp_path):
    conn = sqlite3.connect(tmp_path / "t.db")
    conn.executescript(SCHEMA)
    yield conn            # teardown runs after the test
    conn.close()


@pytest.fixture
def make_user(db):
    created = []

    def _make(**kw):
        u = User(name=kw.pop("name", "alice"), **kw)
        db.save(u)
        created.append(u)
        return u

    yield _make
    for u in created:
        db.delete(u)


def test_duplicate_email_rejected(db, make_user):
    make_user(email="a@b.c")
    with pytest.raises(UniqueViolation):
        make_user(email="a@b.c")
```

## Skeleton — Vitest / Jest

```ts
let db: Pool;

beforeEach(async () => {
  db = await createTempDB();
});

afterEach(async () => {
  await db.end();
});

const makeUser = (over: Partial<User> = {}) => ({ name: 'alice', ...over });
```

## Skeleton — Go

```go
func newTestDB(t *testing.T) *sql.DB {
    t.Helper()
    db, err := sql.Open("sqlite3", "file::memory:?cache=shared")
    require.NoError(t, err)
    t.Cleanup(func() { db.Close() })
    return db
}
```

## Factories over fixture blobs

Prefer a factory with defaults and per-test overrides over five near-identical
fixtures. One factory beats five fixtures.

```python
def order(**over):
    base = {"qty": 1, "sku": "A", "price": Decimal("10")}
    return Order(**{**base, **over})


def test_zero_price_rejected():
    with pytest.raises(ValueError):
        validate(order(price=Decimal("0")))
```

## Commands

| Stack | Fixture mechanism |
|---|---|
| Python | `@pytest.fixture`, `conftest.py` for shared |
| JS/TS | `beforeEach`/`afterEach`, `test.extend()` |
| Go | helpers + `t.Cleanup`, no framework |
| Flutter | `setUp`/`tearDown`, `testWidgets` pump helpers |

## Pitfalls

- Module-scoped mutable fixture shared across tests → order-dependent flakes.
- Teardown that swallows exceptions → leaked resources mask real failures.
- `conftest.py` fixtures used implicitly everywhere → hard to trace.
- Fixtures that assert → failures point at the fixture, not the test.
- Seeding production-shaped data including real PII. Use synthetic data.
