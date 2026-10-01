# e2e

Drive the whole system the way a user does: real browser or real app, real
backend, real DB. Highest confidence, slowest, most brittle. Use sparingly.

## When to use

- One happy path per critical user journey (signup, login, checkout, publish)
- Cross-service flows where a break is a real outage
- Smoke tests after deploy
- Regression guard for a flow that broke in production before

## When NOT to use

- To test branching logic → `integration` or `unit` covers it faster
- For every variation of a form → parametrize at the `integration` layer
- When the UI is still churning daily → you'll rewrite the test weekly

## Rules

- Cap it: a handful of journeys, not dozens of tests. e2e is a smoke net.
- Prefer role/label selectors over CSS classes or XPath.
- Wait for a *state*, never `sleep()`. Use auto-waiting locators.
- Seed data through the API or fixtures, not by clicking through setup.
- Never run against production. Use a dedicated ephemeral environment.
- If a test flakes twice, quarantine it and fix the root cause (`flaky.md`).

## Skeleton

```python
from playwright.sync_api import expect

def test_user_can_signup_and_see_dashboard(page, base_url):
    page.goto(f"{base_url}/signup")
    page.get_by_label("Email").fill("new@example.com")
    page.get_by_label("Password").fill("s3cret-passphrase")
    page.get_by_role("button", name="Create account").click()

    expect(page.get_by_role("heading", name="Dashboard")).to_be_visible()
```

```ts
test('user can signup and see dashboard', async ({ page, baseURL }) => {
  await page.goto(`${baseURL}/signup`);
  await page.getByLabel('Email').fill('new@example.com');
  await page.getByLabel('Password').fill('s3cret-passphrase');
  await page.getByRole('button', { name: 'Create account' }).click();
  await expect(page.getByRole('heading', { name: 'Dashboard' })).toBeVisible();
});
```

```dart
testWidgets('user can signup and see dashboard', (tester) async {
  await tester.pumpWidget(const App());
  await tester.enterText(find.byKey(const Key('email')), 'new@example.com');
  await tester.enterText(find.byKey(const Key('password')), 's3cret-passphrase');
  await tester.tap(find.text('Create account'));
  await tester.pumpAndSettle();
  expect(find.text('Dashboard'), findsOneWidget);
});
```

## Commands

| Stack | Run |
|---|---|
| Python | `pytest tests/e2e --headed=false` |
| JS/TS | `npx playwright test` |
| Flutter | `flutter test integration_test` |
| Smoke tag | `npx playwright test --grep @smoke` |

## Pitfalls

- `time.sleep` / fixed waits → flaky. Wait for elements or network idle instead.
- CSS/class selectors → break on restyle. Use roles and labels.
- Shared test accounts → parallel runs collide. Generate unique data per run.
- Treating a green e2e suite as proof of correctness — it proves the path, not the edges.
- Secrets in fixtures. Redact; use env vars injected by CI.
