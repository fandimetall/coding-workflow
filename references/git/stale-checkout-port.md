# Worked Pattern: Stale Checkout to Upstream Worktree

## Situation

A live application checkout contained many unrelated modifications. The intended bug fix touched five TypeScript files. Local `main` and `origin/main` had one unique commit each, while two target files had been heavily refactored upstream. Focused tests passed in the old checkout, but opening a PR there would have produced a misleading, oversized diff.

## Reliable sequence

1. Enumerate all dirty files and isolate the five intended paths.
2. Save a path-scoped diff as a portable behavioral record.
3. Inspect `origin/main` versions of the key symbols to confirm the bug still exists and identify refactored equivalents.
4. Create a separate worktree and branch directly from the current `origin/main` SHA.
5. Verify worktree cleanliness and base equality.
6. Attempt `git apply --3way` once.
7. When every hunk failed because context had changed, stop mechanical retries and port behavior manually against current interfaces.
8. Re-read current repository contribution rules before choosing configuration surfaces; stale implementation choices may no longer be acceptable upstream.
9. Run focused tests in the upstream worktree, inspect changed paths, and compare broader-suite failures to a clean baseline.
10. Commit only the scoped files; verify commit range and diff before fork/push/PR operations.

## Key diagnostic distinction

- **Bug absent upstream:** do not port; reassess premise.
- **Bug present and files unchanged:** apply or cherry-pick mechanically.
- **Bug present but surrounding files refactored:** semantic port using the old patch as intent, not source code authority.

## Interruption recovery

Network fetches and large worktree creation may outlive a foreground call. Use a tracked background process, then on resume check:

- active process list;
- `git worktree list`;
- whether the destination directory exists;
- whether the branch was created;
- current `origin/main` SHA.

Only retry after determining which steps actually completed. This prevents duplicate fetches, conflicting branch creation, and false assumptions after an interrupted session.
