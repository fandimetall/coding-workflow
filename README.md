# Coding Workflow — Master Protocol

**One file. Every coding task. No dependencies.**

Drop a single `SKILL.md` into your agent's skills directory. From then on, every
coding task — bug fix, feature, refactor, release — follows the same disciplined
path instead of an improvised one.

> **Why one file?** Because skills that reference other skills break the moment
> someone installs only one of them. Worse, **separate skills contradict each
> other** — two of them can both say *MUST* and mean opposite things. This file
> absorbed sixteen competing skills so that the contradiction is impossible.
> The debugging, testing, security, review and git discipline is **embedded, not
> imported.**

---

## What it enforces

**THE LOCK** — three rules at the very top that override everything else:

1. **Effective & efficient > minimum.** Short code is **not** the goal. Be proud
   that the code hits the target safely, never that it is small.
2. **One skill, one process.** There is no second workflow to appeal to.
3. **The user's explicit instruction outranks everything.**

Then the six minimum, always:

1. **Root cause before fix.** No patch until you can state *why* it broke.
2. **One function, one full cycle.** Write → works → test → audit → live → **explain**.
3. **Never verify your own work.** A reviewer may **report**; the author **fixes**.
4. **Report honestly.** Red is red. Skipped is skipped. Unknown is unknown.
5. **Explain every unit back to the user** — so *they* can catch wrong intent.
6. **Every change undoable.** Branch, atomic commits, known rollback path.

---

## The shape

```
PRA-PHASE   who owns the repo · review level · delivery mode
            0.0 vague idea → spec    0.2 spike | plan | ask
PHASE 0     load context  (rules → skills → the code → conventions)
PHASE 1     analysis & root cause        ← Iron Law + FEEDBACK LOOP + rule of three
PHASE 2     reference (only with an external library/API)
PHASE 3     execution                    ← TDD red→green→refactor + 8-step cycle
PHASE 4     verification                 ← tests · real entry point · IMPACT-AUDIT GATE
                                           · secret scan · 4-lens · ladder · INDEPENDENT review
PHASE 5     git & deploy                 ← branch · atomic commits · SemVer · rollback
                                           · PR lifecycle · issue→PR (sabotage run)
                                           · patch porting · merge conflicts
PHASE 6     honest report · explain back · journal
```

Five appendices carry the detail: **A** testing catalogue (12 types + decision
matrix + money invariants), **B** common failures, **C** tool mapping (portable
to any runtime), **D** complexity scale **S1–S5**, **E** the contradiction
resolution table.

---

## The two ideas that do the most work

**1. Build a red-capable feedback loop before you theorise.**

> The feedback loop *is* the debugging work. Reproduce the user's exact symptom
> with a command you have **seen fail**. A loop that has never gone red has
> proven nothing.

**2. Make the test bite — the sabotage run.**

> Break the fix on purpose and confirm the regression test **goes red**. A test
> that stays green under sabotage is not testing the bug.

Both exist because agents — and humans — routinely claim *"done"* without
evidence. This file refuses to accept that.

---

## What it refuses to do

- **Refuses to be proud of short code.** § 4.10 has an explicit guard: the
  anti-bloat ladder tells you not to *invent* work, never to write fragile code.
- **Refuses self-review.** § 4.12b requires an independent pass.
- **Refuses optimistic reporting.** *"All tests pass"* is banned when any test
  failed, was skipped, or was never run.
- **Refuses to rewrite shared history.** No `reset` / `rebase` / `push --force`
  on a shared branch, without exception.
- **Refuses to hide code from whoever must judge it** — and refuses to dump code
  on whoever cannot.

---

## Install

```
your-agent/skills/coding-workflow/SKILL.md
your-agent/skills/coding-workflow/references/...
```

Only the file and its `references/` folder are needed. `references/` is used by
PHASE 3 (test types) and PHASE 5 (commits, CI, PR bodies, patch porting).

## Size

| Part | Size |
|---|---|
| `SKILL.md` | ~59 KB |
| `references/` (17 files) | ~47 KB |
| **Total** | **~105 KB** |

---

## Version

**3.0.0** — see `CHANGELOG.md` for the full merge record, including which skill
each rule came from and which contradiction each rule resolved.

## License

MIT — © Fandi Iswara Saputra (@fandimetall)
