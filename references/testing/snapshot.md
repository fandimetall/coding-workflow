# snapshot

Serialize output to a stored "golden" file and compare on every run. Catches
*unintended change*. It does not tell you the output is correct.

## When to use

- UI component rendering where the exact tree/markup matters
- Serialized payload shapes (JSON, HTML, generated code)
- CLI output and error messages that users read
- Detecting accidental changes after a refactor

## When NOT to use

- As the only test for logic → `snapshot` catches change, not correctness
- Highly dynamic output (timestamps, IDs, random) unless normalized first
- Large blobs nobody reviews → snapshots nobody reads get blindly updated
- Anything you'd approve with `-u` without reading the diff

## Rules

- **Always review the diff before updating.** `--update` without reading is how
  bugs get committed as "expected".
- Normalize volatile data first: dates, ids, paths, memory addresses, ordering.
- Keep snapshots small and human-readable. A 3000-line snapshot is dead weight.
- One snapshot per meaningful state (empty, filled, error), not per prop combo.
- Inline snapshots for small values; files for large render trees.

## Normalization

```python
def normalize(s: str) -> str:
    s = re.sub(r"[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}", "<uuid>", s)
    s = re.sub(r"\d{4}-\d{2}-\d{2}T[\d:.]+Z", "<ts>", s)
    s = re.sub(r"/tmp/[^\s\"']+", "<path>", s)
    return s


def test_render_invoice(snapshot):
    snapshot.assert_match(normalize(render(invoice_fixture())), "invoice.html")
```

```ts
expect(renderInvoice(invoice)).toMatchSnapshot();   // Vitest/Jest
await expect(page).toHaveScreenshot('invoice.png');  // Playwright visual
```

## Commands

```bash
# Python (pytest-snapshot / syrupy)
pytest --snapshot-update            # review the diff in git before committing

# JS/TS
npx vitest run -u
npx jest -u

# Playwright visual
npx playwright test --update-snapshots

# Flutter golden
flutter test --update-goldens
```

## Visual / golden snapshots

Pixel snapshots (Playwright `toHaveScreenshot`, Flutter goldens) are the most
brittle kind. Pin the viewport, font, device pixel ratio, and theme. Run them in
the same container image as CI. Expect cross-platform pixel diffs.

## Pitfalls

- Blindly updating on every failure → the snapshot becomes a changelog of bugs.
- Snapshotting a whole page instead of the component under test → unrelated churn.
- Un-normalized UUIDs/timestamps → every run fails.
- Storing snapshots outside version control → no history, no review.
- Treating a green snapshot as proof the UI is *right*.
- Secrets or real PII embedded in a committed snapshot. Redact before storing.
