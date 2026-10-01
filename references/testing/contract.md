# contract

Verify that two sides agree on an interface: request/response shape, status
codes, event schema. Catches the break *before* deploy instead of in prod.

## When to use

- You call someone's API (consumer side)
- Someone calls your API (provider side)
- Two internal services exchange messages/events
- A mock/fake must stay faithful to the real service
- Third-party webhooks you receive

## When NOT to use

- Both sides live in one process and deploy together → `unit`/`integration`
- You only need to verify your own handler's logic → `integration`

## Two directions

| Direction | Who writes the test | Protects |
|---|---|---|
| **Consumer-driven** (Pact) | The consumer | Consumer breaks first, loudly |
| **Provider verification** | The provider | Old consumers keep working |

## Consumer-driven (Pact)

```ts
// consumer test — defines the expectation
const interaction = {
  state: 'user 42 exists',
  uponReceiving: 'a request for user 42',
  withRequest: { method: 'GET', path: '/users/42' },
  willRespondWith: {
    status: 200,
    body: { id: 42, name: like('alice'), email: like('a@b.c') },
  },
};

it('gets user 42', async () => {
  await pact.addInteraction(interaction);
  const res = await client.getUser(42);
  expect(res.name).toBeDefined();
  await pact.verify();
});
```

```python
# provider verification (pytest + pact-python)
def test_provider_against_pact(verifier):
    verifier.verify_pacts("./pacts/user_service-user_client.json")
```

## Schema-only alternative (lighter)

When full Pact is overkill, assert the shape against a spec:

```python
USER_SCHEMA = {
    "type": "object",
    "required": ["id", "name", "email"],
    "properties": {
        "id": {"type": "integer"},
        "name": {"type": "string"},
        "email": {"type": "string", "format": "email"},
    },
    "additionalProperties": False,
}


def test_get_user_matches_schema(client):
    body = client.get("/users/42").json()
    jsonschema.validate(body, USER_SCHEMA)
```

```bash
# OpenAPI-driven checks
npx @stoplight/spectral-cli lint openapi.yaml     # spec itself
npx dredd openapi.yaml http://localhost:3000      # impl vs spec
```

## Event / message contracts

```python
def test_order_created_event_schema():
    event = build_published_message(sample_order())
    jsonschema.validate(event, ORDER_CREATED_V1)   # versioned schema


def test_event_is_backward_compatible():
    # old consumer fields must still be present
    event = build_published_message(sample_order())
    assert {"order_id", "total", "currency"} <= event["data"].keys()
```

## Rules

- Version contracts (`v1`, `v2`). Never break a published field in place.
- Additive changes only: new optional fields. Removing/renaming is a major bump.
- Provider verification runs in the *provider* CI, against consumer pacts.
- A green contract test means the interface matches, not that behavior is correct.

## Commands

| Stack | Tool | Run |
|---|---|---|
| Python | `pact-python`, `jsonschema` | `pytest tests/contract` |
| JS/TS | `@pact-foundation/pact` | `npx jest tests/contract` |
| OpenAPI | `dredd`, `spectral` | `npx dredd openapi.yaml $URL` |
| Go | `pact-go`, `go-openapi` | `go test -run Contract` |

## Pitfalls

- Contract tests that assert values instead of shapes → brittle on data changes.
- Running provider verification only on release, not on every provider commit.
- Letting a failing provider verification be "fixed" by regenerating the pact.
- No versioning → silent breakage for existing consumers.
- Real credentials in pact files. Use example/redacted values.
