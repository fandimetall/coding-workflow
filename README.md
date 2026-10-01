# Coding Workflow — Master Protocol

A single-file coding discipline for AI agents. **Six phases, one hard gate, zero dependencies.**

Drop one file into your agent's skills directory. From then on, every coding task —
bug fix, feature, refactor — follows the same disciplined path instead of an
improvised one.

> **Why one file?** Because skills that reference other skills break the moment
> someone installs only one of them. This one is self-contained: the debugging,
> testing, QA, and review discipline is embedded, not imported.

---

## What it actually enforces

Nine rules carry most of the weight:

1. **Root cause before fix.** No patch until you can state *why* it broke.
2. **One function, one full cycle.** Write → works → test → audit → live → **explain**. Then next.
3. **Never verify your own work.** Fresh eyes or a subagent finds what you miss.
4. **Explore it like a real user, not like its author.** Unit tests share the
   author's blind spots; exploratory QA does not.
5. **Report honestly.** Red is red. Skipped is skipped. `x/y passing`.
6. **Explain it back to the user.** After each unit, in plain language, so *they*
   can review it. You are the only author; they are the only one who knows what it
   was supposed to do.
7. **Show the code when the user can read it.** A plain summary cannot reveal an
   N+1 query or a needless abstraction. If the owner is a developer, they are the
   best reviewer you will ever get — do not hide the diff from them.
8. **Never lock a willing learner out of code.** A beginner who *wants* to
   understand gets annotated real code, one concept at a time. "Cannot read code"
   and "does not want to learn" are not the same thing.
9. **Deliver it the way they asked.** Inline, or a pointer like
   `lib/filter.py:34-40` — but **the lines that changed are always pasted**. Only
   the *unchanged* context becomes a pointer. A pointer with nothing pasted is not
   a compact review, it is a withheld one.

Plus one **hard gate** you cannot pass until three checks are done — blast radius,
test + audit, cleanup.

And one that makes the rest survivable: **every change reversible.** The gate
reduces bad changes; version control makes them undoable. Both, or neither is much
comfort on a bad day.

### Three review modes, because users differ

The workflow asks the user **up front** (PRA-PHASE 0.1b) — and it asks **two**
questions, not one: *can* you judge code, and do you *want* to understand it? Those
are independent, and each combination catches a different class of mistake:

| Mode | For | Catches | Cannot catch |
|---|---|---|---|
| **Expert** | user reads code fluently | inefficient or clumsy code | — |
| **Learning** | user can't read it *yet*, but wants to | **unexplained code** — the kind no future maintainer understands either | code quality (not yet) |
| **Plain** | user neither reads nor wants to | wrong intent — *"that is not what I meant"* | anything in the code itself |

The important one is **Learning**: without it, the skill's own design would keep a
beginner locked out of code for the entire project — making them *worse off* for
having worked with you. So a finished unit is not "done" when the tests pass; it is
done when the user has had a real chance to say *"yes, that is what I meant"*, and —
if they want it — *"I understand how it works."*

### Two ways to deliver the code (PRA-PHASE 0.1c)

Review *level* says how deep. Delivery *mode* says how the code reaches the reader —
useful when a long file would otherwise flood the chat.

| Mode | Shows | Best when |
|---|---|---|
| **Inline** | the changed code, in the reply | small changes, quick decisions |
| **Pointer** *(default)* | the changed lines **+** `lib/filter.py:34-40` + a one-line anchor | long files, scattered edits, keeping chat short |

The rule that keeps this honest: **the lines that changed are always pasted. Only
the lines that did *not* change become a pointer.** Pasted 200 lines where 6 changed
buries the 6 — but a pointer with *nothing* pasted is not a compact review, it is a
withheld one. So: **meaning first, then the changed code, then a pointer for the
rest.** In that order, every time.

---

## The flow

```
PRA-PHASE   Ask ownership (personal/team)    ← all three
            Ask review level (expert/         mandatory,
            learning/plain)                    every task
            Ask delivery mode (inline/pointer)
            spike / plan
     ↓
PHASE 0     Load context              skills · journal · blast radius
     ↓
PHASE 1     Analysis                  root cause + a tight red-capable loop
     ↓
PHASE 2     Reference                 current docs, never from memory
     ↓
PHASE 3     Execution                 effective code + per-function cycle
     ↓
PHASE 4     Verification         🔒   IMPACT-AUDIT GATE (a)(b)(c)
     ↓
PHASE 5     Version control           mode · branch · atomic commits ·
            & deploy                  SemVer · tag · rollback · verify
```

**Step 1 is a question, not an analysis.** Before any code is written, the workflow
makes the agent ask whether the repo is *personal* or *team/shared*. The answer is
not cosmetic — it decides how hard the blast radius is checked (PHASE 0), whether
the agent may review its own work (PHASE 4), and whether it may commit to `main`
(PHASE 5). Asking it at commit time is too late: the previous four phases already
ran at a guessed strictness.

---

## What it looks like in practice

The single highest-leverage rule is the **per-function cycle** (PHASE 3.8). This is
the real bug that motivated it.

**The bug.** An injury filter was added to a workout planner. For a user with a knee
injury, the filter *added* `glute_bridge` as a knee-safe substitute — and then the
very same filter *discarded* it, because it classified movements by **muscle trained**
instead of by **joint loaded**. Result: a knee-injured user received an **empty** leg
list.

**Why the tests missed it.** The unit tests were written by the same mind that wrote
the filter, so they tested the filter's own (wrong) mental model. Green tests, broken
product. It surfaced only during exploratory QA — running the planner with a *real*
profile and reading the output.

**The rules that catch it earlier next time:**
- PHASE 3.8 — test each function the moment it is written, not after the module is done.
- PHASE 4.12(b) — exploratory QA on a realistic profile, not a synthetic one.
- ATURAN — *"An empty list where a list was expected is a bug, not a page."*
- PHASE 1 — root cause before fix: the *classification* was wrong, not the filter line.

---

## Install

**Hermes**
```bash
cp -r coding-workflow ~/.hermes/profiles/<you>/skills/software-development/
```

**Claude Code**
```bash
mkdir -p .claude/skills && cp -r coding-workflow .claude/skills/
```

**Cursor / Windsurf**
Copy the body of `SKILL.md` into `.cursor/rules/coding-workflow.mdc`.

**Anything else (ChatGPT, Gemini, a raw API call)**
Paste `SKILL.md` into your system prompt. The tool-mapping table at the top tells
the model how to substitute tools it actually has.

**No install at all**
Read it and use it as a checklist. Nothing in the discipline requires tooling.

---

## Tool portability

The document is written tool-agnostically, with one mapping table for the few names
that are runtime-specific:

| Written as | Hermes | Claude Code / Cursor | Manual fallback |
|---|---|---|---|
| `skills_list()` | skill loader | read `.claude/skills/` | skip — use this file alone |
| `codegraph` | codegraph MCP | LSP find-references | `grep -rn "fn" .` |
| Context7 | docs MCP | docs lookup / fetch | read official docs |
| `delegate_task` | subagent spawner | Task tool | do it sequentially |

Everything else is pure discipline — works anywhere.

---

## What is embedded

| Discipline | Where |
|---|---|
| Root-cause investigation + tight feedback loop | PHASE 1 |
| Read-docs-not-memory | PHASE 2 |
| Effective-vs-short, input validation, anti-data-loss | PHASE 3.6–3.7 |
| Per-function cycle + TDD (RED-GREEN-REFACTOR) | PHASE 3.8 |
| Testing strategy — minimum sufficient kinds | PHASE 4.12(b) |
| Exploratory QA / dogfooding | PHASE 4.12(b) |
| Independent review + security scan + baseline-aware gate | PHASE 4.12(b) |
| Impact audit + cleanup | PHASE 4.12(a)(c) |
| Repo-ownership question (personal vs team) | PRA-PHASE 0.1 |
| **Review level question (expert / learning / plain)** | **PRA-PHASE 0.1b** |
| **Delivery mode question (inline / pointer + file:line format)** | **PRA-PHASE 0.1c** |
| **Per-function cycle — write/works/test/audit/live/explain** | **PHASE 3.8** |
| **Unit explanation back to the user for manual review** | **PHASE 3.8.1** |
| **Expert layer — show code, decision, trade-off, weak point** | **PHASE 3.8.1b** |
| **Learning layer — concept, annotated code, one term, invite to ask** | **PHASE 3.8.1c** |
| Version-control discipline — context modes, branch, atomic commits, SemVer, tag, CHANGELOG, rollback | PHASE 5 |
| Rule of Three — when to question the architecture | ATURAN |
| Rationalisation table — "I don't need this because…" | ATURAN |
| Coding journal format | Appendix A |

---

## A real bug this prevents

A workout planner had a knee-injury filter. It correctly added `glute_bridge` to the
list of knee-safe substitutes — and then the filter *also* discarded `glute_bridge`,
because it classified the movement by the muscle it targets rather than the joint it
loads. The result: for a user with a knee injury, the leg list came back **empty**.

Nobody caught it while writing the filter. It surfaced only when someone ran the
system with a realistic profile and noticed *an empty list where a list was expected*.

Two rules in this file would each have caught it:
- **Per-function cycle** — the substitute list was written and never tested in isolation.
- **Exploratory QA** — "an empty result where data was expected is a bug, not a page."

---

## Files

```
coding-workflow/
├── SKILL.md      ← the protocol (self-contained)
├── README.md     ← this file
├── CHANGELOG.md  ← release history (Keep a Changelog + SemVer)
└── LICENSE       ← MIT
```

## Credits & License

Author: **Fandi Iswara Saputra** ([@fandimetall](https://github.com/fandimetall))

MIT. Use it, fork it, ship it. See [LICENSE](LICENSE).
