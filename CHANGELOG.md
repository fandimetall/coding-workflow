# Changelog

All notable changes to this skill are documented here.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
Versioning follows [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [2.6.0] - 2026-10-01

### Added

- **PRA-PHASE 0.1c — delivery mode: `inline` or `pointer`.** Review level (0.1b)
  says *how deep*; delivery mode says *how the code reaches the user*. Chosen once,
  applies to the whole task. Default **pointer** — short chat, the file on disk is
  the source of truth.
- **Pointer format (all three parts mandatory)** — full path from the repo root,
  line number or range, and a one-line anchor: `lib/services/filter.py:34-40` —
  *"line 34 is `if joint in injured_joints:`"*.
- **How-to-deliver block in PHASE 3.8.1b**, per mode.

### Fixed

- **"Show the changed region plus enough surrounding context" was ambiguous.**
  *"Region"* was never defined, so a 200-line function could be pasted in full and
  still be compliant — chat flooded with lines that had not changed. **Now defined:
  the changed lines are pasted, the unchanged context becomes a pointer.**
- **A missing capability, not just a missing rule.** The workflow forbade *"dumping
  the whole file"* three separate times but never offered a **compact** alternative.
  Prohibition without an alternative is the shape of a rule that gets ignored.
  0.1c supplies the alternative.

### Changed

- **"Eight rules that carry most of the weight" → nine** — added *deliver code the
  way the user asked; changed lines always pasted, unchanged context becomes a
  pointer*.
- **Learning mode gets a pointer exception.** Pointers are allowed for parts you are
  not annotating, but the block being **taught** is always pasted — **annotating a
  pointer teaches nothing.**
- **The "never dump" rules now read as "dump less context, not less change."**
- Four new rows in "tempted to skip": *"a pointer is shorter"*, *"the chat is getting
  long"*, *"they have the repo"*, *"pasting burns tokens"* — each answered with the
  same distinction: **unchanged context is what gets abbreviated, never the change.**

### Guarded against

- **Pointer mode must not resurrect the evasion rule 6 forbids.** *"Done, see
  `filter.py:34`"* is not compact — it is withheld. The rule is now explicit:
  **meaning first, then the changed code, then a pointer for the rest.**

## [2.5.0] - 2026-10-01

### Added

- **Learning mode — PHASE 3.8.1c.** For a user who **cannot read code yet but wants
  to learn.** Each unit also carries: *the concept before the syntax · annotated real
  code · one term defined · why the code is shaped this way · what to notice · a
  place to read more · an open invitation to ask.*
- **PRA-PHASE 0.1b now asks TWO questions, not one** — (A) *can* they judge code,
  (B) do they *want* to understand it. These are independent axes; collapsing them
  loses the beginner who wants to learn.

### Fixed

- **A third mode was missing.** 2.4.0 offered only *Expert* and *Plain*. A user who
  cannot read code **but wants to** had no home — and fell into Plain, which by
  design shows no code at all. That made the skill the *reason* they never learned.
- **Plain vs Learning were conflated.** The 2.4.0 trigger for Plain was *"non-
  technical / avoids code"* — which lumps **"cannot"** together with **"does not want
  to."** The distinction now matters: *cannot + willing* → **Learning**;
  *cannot + unwilling* → **Plain**.

### Changed

- **0.1b table rebuilt** around the two axes (can judge × wants to learn), with the
  four resulting modes and explicit behaviour signals for each.
- **Three-way value statement** — Expert catches *suboptimal* code, Plain catches
  *wrong-intent* code, **Learner catches *unexplained* code** — the kind no future
  maintainer would understand either.
- **"Seven rules that carry most of the weight" → eight** — added *never lock a
  willing learner out of code*.
- Four new rows in "tempted to skip": *"they can't read code, so keep it plain"*,
  *"explaining will slow the project down"*, *"they'll ask if they want to know"*,
  *"I'll simplify the code so it's easier to read"*.
- Guiding rule added to the learning layer: **annotate, do not obfuscate** — never
  simplify the code for teaching and then ship the simplified version.

## [2.4.0] - 2026-10-01

### Added

- **PRA-PHASE 0.1b — mandatory second question: what review level does the user
  want?** Repo ownership (0.1) is about the *code*; this is about the *reviewer*.
  The two are independent — a solo repo can have an expert owner, a team repo can
  have a non-technical owner. The answer decides how every unit is presented, so it
  must be asked at the start, not guessed at the end.
- **PHASE 3.8.1b — the Expert Review Layer.** When the user can read code, each unit
  explanation also carries: *the code itself (diff/excerpt) · the technical decision ·
  the honest trade-off · where the agent is least confident · what it wants judged.*
- **Why this is a separate layer, not a repeat of 3.8.1:** a plain-language summary
  can catch *"that is not what I meant."* It can never catch an N+1 query, a correct-
  but-10000×-slower function, a needless abstraction, or a silent `except: pass`.
  **Only a code-reading owner can.** Neither layer substitutes for the other.

### Fixed

- **Contradiction with 2.3.0.** The 2.3.0 anti-pattern note said *"the user can read
  `git diff` themselves — that is not what they need."* That is correct for a Plain
  reviewer and **wrong for an Expert one** — it actively withheld the code from the
  person best able to judge it. Rewritten as ***"meaning first, then code"***, not
  *"meaning instead of code."*

### Changed

- **ATURAN** — new rule: *match the review to the reviewer.*
- **ATURAN (3.8.1)** — softened from *"not a file list, not a diff"* to *"not a
  **bare** file list"*, so it no longer forbids showing code.
- **"Six rules that carry most of the weight" → seven** — added *show the code when
  the user can read it*.
- Three new rows in "tempted to skip": *"the user can read git diff"*, *"the
  explanation is enough"*, *"showing the code invites nitpicks"*.
- Quick Reference: the PRA row now asks for **ownership + review level**.

## [2.3.0] - 2026-10-01

### Added

- **PHASE 3 step 8 — sixth step in the per-function cycle: `EXPLAIN`.** After a unit
  works, is tested, and its blast radius is checked, the agent must **explain how it
  works to the user, in plain language, so the user can review it manually.**
- **PHASE 3.8.1 — full specification of that explanation.** Five required elements:
  *what it does · how it works · why this way (trade-off) · how it was verified
  (honest `x/y`) · what the user should try themselves.* Plus an explicit end-of-task
  report (the PHASE 4.12 four-liner + a walkthrough of the finished feature).
- **Why this closes a real gap:** every other verification in the workflow is done by
  a machine or by the agent itself. **The user is the only reviewer who knows what the
  code was supposed to accomplish.** Without an explanation, the user's only feedback
  loop is discovering the misunderstanding in production.
- Three new rows in the "tempted to skip" table: *"here are the files I changed"*,
  *"the user can read the code"*, *"the tests pass, that's the verification"*.

### Changed

- **ATURAN** — new rule: *explain every finished unit back to the user; a task is not
  "done" until the user can review it.*
- **Quick Reference** — PHASE 3's "done when" now requires the unit to be explained,
  not just tested.
- **"Five rules that carry most of the weight" → six** — added *explain it back to
  the user*.
- Explanation depth scales with the PRA-PHASE 0.1 answer: short form for a personal
  repo, short form **plus why-this-way and the trade-off** for a team repo.

## [2.2.0] - 2026-10-01

### Added

- **PRA-PHASE 0.1 — mandatory repo-ownership question.** Before any coding starts,
  ask whether the repo is personal or team/shared. The answer sets the strictness
  of every later phase: blast-radius depth, whether self-review is allowed, whether
  you must branch. Previously this was discoverable only at commit time — after
  four phases had already run at a guessed strictness.
- **Anti-pattern note** in PRA-PHASE: asking about repo ownership at the commit
  step is too late.
- Two new rows in the "tempted to skip" table — *"it's just a small change, I don't
  need to ask about the repo"* and *"I'll figure out the repo type when I commit"*.

### Changed

- **PHASE 5.1** no longer picks the mode from scratch — it **re-reads the PRA-PHASE
  0.1 answer**, and adds a consistency check (team repo ⇒ never commit to `main`,
  no exceptions for small changes).
- **PHASE 0.3 blast radius** is now *mandatory and thorough* for team/public repos,
  a quick sanity check for personal ones.
- **PRA-PHASE "plan first"** is now always required for **team repos**, not only
  for tasks >1 hour or >3 files.
- Quick Reference updated: the PRA row now starts with *"ask repo ownership"*.

## [2.1.0] - 2026-10-01

### Added

- **PHASE 5 rewritten as a full version-control discipline** (was 3 lines).
  A change you cannot undo is not a change — it is a gamble.

  - **5.1 Context modes** — a table matching versioning ceremony to context:
    *solo/throwaway* → *solo/real project* → *team/public*. When unsure, pick
    the stricter mode.
  - **5.2 Branching** — the branch is your way home. Never commit directly to
    `main` on a shared repo.
  - **5.3 Atomic commits** — one logical change per commit, tested by "can this
    be described in one line with no *and*?". This is what makes `git revert`
    surgical instead of destructive.
  - **5.4 Semantic Versioning** — MAJOR/MINOR/PATCH table, plus the
    non-negotiables: one source of truth, released versions are immutable,
    bump in the release commit.
  - **5.5 Tag + CHANGELOG** — annotated tags only; changelog written for a human
    deciding whether to upgrade, describing *effect on the user* rather than
    "changed line 42".
  - **5.6 Rollback** — a decision table from the safest tool to the heaviest,
    plus the rule that prevents most pain: **never rewrite shared history**.
  - **5.7/5.8** — coding journal (git records *what*, the journal records *why*)
    and deploy verification. Roll back first, diagnose second.

- **Two new rules in ATURAN**: *every change must be reversible*, and *never
  rewrite shared history*.

### Changed

- **PHASE 5 renamed** from "Git & Deploy" to "Version Control & Deploy" — it now
  covers the full lifecycle, not just the commit.
- **Quick Reference table** updated: PHASE 5's "done when" is now *"it works in
  the real environment **and can be undone**"*.
- `description` frontmatter extended with the version-control discipline keywords.

## [2.0.0] - 2026-10-01

### Changed

- **BREAKING: skill is now fully self-contained.** All eight external skill
  dependencies removed and their content embedded inline:

  | Was delegated to | Now embedded as |
  |---|---|
  | `systematic-debugging` | PHASE 1 — Iron Law, tight-feedback-loop, trace-upstream |
  | `testing-strategy` | PHASE 4.12(b) — change-type → minimum-sufficient-test table |
  | `dogfood` | PHASE 4.12(b) — exploratory QA procedure |
  | `test-driven-development` | PHASE 3.8 — RED-GREEN-REFACTOR inside the per-function cycle |
  | `requesting-code-review` | PHASE 4.12(b) — independent review + security scan |
  | `simplify-code` | PHASE 4.12(c) — cleanup pass |
  | `spike`, `plan` | PRA-PHASE |
  | `coding-journal` | Appendix A |

- **Added Tool Mapping table** — the document now states how each Hermes-specific
  tool name maps to Claude Code, Cursor, Copilot, or a manual fallback, so it is
  usable outside Hermes.
- **Added README.md and LICENSE (MIT).**

### Added

- Appendix B documenting *why* the external references were removed.

## [1.0.0] - 2026-10-01

### Added

- Initial release. Six-phase workflow (0–5) with a mandatory Impact-Audit Gate
  between PHASE 4 and PHASE 5, the per-function cycle, the Rule of Three, the
  anti-skip excuse table, and the coding-journal format.

[2.6.0]: https://github.com/fandimetall/coding-workflow/compare/v2.5.0...v2.6.0
[2.5.0]: https://github.com/fandimetall/coding-workflow/compare/v2.4.0...v2.5.0
[2.4.0]: https://github.com/fandimetall/coding-workflow/compare/v2.3.0...v2.4.0
[2.3.0]: https://github.com/fandimetall/coding-workflow/compare/v2.2.0...v2.3.0
[2.2.0]: https://github.com/fandimetall/coding-workflow/compare/v2.1.0...v2.2.0
[2.1.0]: https://github.com/fandimetall/coding-workflow/compare/v2.0.0...v2.1.0
[2.0.0]: https://github.com/fandimetall/coding-workflow/compare/v1.0.0...v2.0.0
[1.0.0]: https://github.com/fandimetall/coding-workflow/releases/tag/v1.0.0
