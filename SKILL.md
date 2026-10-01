---
name: coding-workflow
description: "Master coding workflow — the ONE coding skill. Asks repo ownership, review level and delivery mode, runs the full cycle (context, analysis, TDD, execution, security scan, independent review, git, honest report), and explains each unit back so the user can catch wrong intent. Merges debugging discipline, TDD, testing strategy, pre-commit security review, anti-bloat review, planning, spikes, PR lifecycle and journaling into one self-contained file. Portable."
version: 3.0.0
author: Fandi Iswara Saputra (@fandimetall)
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [coding, workflow, testing, tdd, qa, debugging, code-review, quality-gate, security, anti-bloat, planning, portable]
    related_skills: []
---

# Coding Workflow — Master Protocol (v3.0, unified)

> **One file, no dependencies.** This is the **only** coding-process skill you need.
> Debugging, testing, QA, security review, anti-bloat review, planning, spikes, PR
> lifecycle and journaling all live **inside this file** — nothing is deferred to a
> sibling skill you may not have installed.
>
> v3.0 **absorbs** what used to be separate skills (`plan`, `spike`, `systematic-
> debugging`, `test-driven-development`, `testing-strategy`, `requesting-code-review`,
> `simplify-code`, `ponytail*`, `coding-journal`, `brief-ku`, `github-pr-workflow`,
> `github-issue-to-pr`, `git-patch-porting`, `merge-reconciler`, `agent-skills-
> addyosmani`, `ecc-agent-harness`). Those are gone on purpose: **one source of
> truth, so no two skills can contradict each other.**

**MUST run on EVERY coding / problem-solving task, without being asked.**

---

## ⛔ THE LOCK — read this before anything else

Three rules outrank every other rule in this file. They exist because the
absorbed skills contradicted each other, and one of them contradicted the
user's own priority.

**1. EFFECTIVE & EFFICIENT > MINIMUM. Short code is NOT the goal.**

This file contains a full anti-bloat toolkit (§ 4.10). It is a **tool, not a
target.**

- The goal is code that **hits the target, saves resources, and is safe** —
  not code that is fewest lines.
- When shorter means **more fragile, harder to read, or harder to maintain**,
  **the longer version is the correct one.** Take it, and say why.
- Be proud that it works and is safe. Never be proud that it is short.
- A `ponytail:`-style "one line!" instinct is a **hint to check**, never an
  instruction to obey. If the one-liner is unreadable, **reject it.**

**2. ONE SKILL, ONE PROCESS.** The cycle below is the whole process. Do not
import a second coding process on top of it. If another skill seems to command
a different order (e.g. "plan only, never implement", "throw the code away"),
that skill is **a phase of this one**, not a competing authority — follow the
phase boundaries here.

**3. THE USER'S EXPLICIT INSTRUCTION OUTRANKS EVERYTHING**, including this
file. If the user says "just do it, skip the ceremony", you skip it — **out
loud**, naming what you skipped (`§ RULES`: silent skipping is the failure this
protocol exists to prevent).

---

## Tool Mapping — read this if you are NOT running Hermes

This document is written in a tool-agnostic way, but a few names are Hermes-specific.
Map them to whatever you have:

| Written as | Hermes | Claude Code / Cursor / Copilot | Manual fallback |
|---|---|---|---|
| `skills_list()` / `skill_view()` | skill loader | read `.claude/skills/`, `.cursor/rules/` | skip — use this file alone |
| `codegraph explore/callers/impact` | codegraph MCP | LSP "find references", IDE call hierarchy | `grep -rn "fn_name" .` |
| `terminal` | shell tool | built-in shell | your own terminal |
| `read_file` / `search_files` | file tools | built-in read/search | `cat`, `rg` |
| `sequentialthinking` (MCP) | MCP server | extended-thinking toggle | write the reasoning out by hand |
| Context7 (MCP) | docs MCP | docs lookup / web fetch | read official docs |
| `delegate_task` | subagent spawner | sub-agent / Task tool | do it yourself, sequentially |
| `coding_journal` section | file on disk | same | a plain `NOTES.md` |

**Everything else in this file is pure discipline — no tooling required.**

---

## PRA-PHASE — Before Writing Code

The pre-phase answers **five** questions, in order. Each one changes how
strictly every later phase runs. Skipping them means the whole workflow runs
at a guessed strictness — which is worse than no workflow.

| # | Question | Where it came from |
|---|---|---|
| 0.0 | Is this a **vague app idea** that needs a spec first? | `brief-ku` |
| 0.1 | Who does this repo **belong to**? | original |
| 0.1b | What **review level** does the user want? | original |
| 0.1c | How should the code be **delivered**? | original |
| 0.2 | Is the task **clear and low-risk** — or does it need a **spike** or a **plan**? | `spike`, `plan` |

### 0.0 — Is this a vague app idea? Build the spec first (from `brief-ku`)

If the user describes an **application idea** — even one sentence, e.g. *"buatkan
saya aplikasi absensi"* — do **not** start coding. Turn the idea into a spec first.

**Five stages:**

1. **Understand** the idea — restate it back in one sentence and confirm.
2. **Clarify** — 4–7 questions **max** per round, using the `clarify` tool, never
   plain text. Ask only what is business-critical; use sensible defaults for the rest.
3. **Recommend** architecture and stack (§ stack table below).
4. **Generate** the spec: PRD + technical spec.
5. **Prepare** an implementation-ready plan (then continue to § 0.2).

**Two mandatory first questions:**

| # | Question | Default if unanswered |
|---|---|---|
| 1 | **Scope** — full functional, or UI prototype only? | full functional |
| 2 | **Platform** — web, or mobile? | web |

**Rules:**

- **Never silently invent business-critical requirements.** If you assumed
  something, write the assumption down where the user can see it.
- **Never bury a non-technical user in jargon.**
- **Prefer the simplest architecture that satisfies the requirement.** No
  microservices for a CRUD app. No queue for a synchronous job. (This is the
  *effective & efficient* rule of § THE LOCK, applied at design time.)

**Default stack — recommendations only, never imposed:**

| Use case | Stack |
|---|---|
| Modern web / SaaS | Next.js + TypeScript + PostgreSQL + Prisma + Tailwind |
| Shared hosting / cPanel | PHP + MySQL, or plain static + a small API |
| Static SPA (portfolio, tool) | Vite + vanilla/TS → static build, **no server** |
| Desktop | pywebview / Tauri, or Electron if the team already lives in Node |

**Done when:** you can write the requirement in one sentence the user agrees
with, and you know the platform.

---

### 0.1 — Ask this FIRST, every time: who does this repo belong to?

**Before any coding starts, establish the ownership context.** You cannot pick the
right level of ceremony without it, and guessing wrong costs either wasted hours
(too much process on a scratch script) or an incident (too little on a shared repo).

Ask, in one short question:

> *"Is this a repo just for you, or is it shared with a team / other people?"*

| Answer | Strictness |
|---|---|
| **Solo / throwaway** | Lighter ceremony: no branch required, self-review allowed if you say so — but **root cause, tests and honest reporting still apply.** |
| **Team / shared / public** | Full ceremony: branch, atomic commits, **independent** review, no shared-history rewrite. |

**When you cannot tell, treat it as a TEAM repo.** The cost of extra care on a
solo repo is minutes; the cost of too little care on a shared one is an incident.

**Do not guess from the size of the task.** A one-line change to a shared repo
still deserves the team track; a 500-line change to a scratch file does not.

---

### 0.1b — Ask second: what review level does the user want?

Ask **two** things — they are independent (PHASE 3.8.1 explains how to use the answer):

1. **Can they judge code?** (expert / can read some / cannot read code)
2. **Do they *want* to understand it?** (yes, explain it / no, just make it work)

| Can read code | Wants to learn | Mode | Presentation |
|---|---|---|---|
| yes | — | **Expert** | Full diff, technical decisions, trade-offs, where you are least confident |
| partly / no | yes | **Learning** | Annotated real code, one concept at a time, open invitation to ask |
| no | no | **Plain** | Meaning-first: what it does, why, what changed, one thing to try |

**Never hide the code from someone able to judge it.** **Never dump code on
someone who cannot read it.** **Never lock a willing learner out of the
code** — "cannot read code" and "does not want to learn" are not the same thing,
and treating a willing learner as a plain reader is the one presentation choice
that leaves the user *worse off* for having worked with you.

---

### 0.1c — Ask third: how should the code be *delivered*?

| Delivery mode | What the user gets | Use when |
|---|---|---|
| **pointer** *(default)* | **Changed lines pasted** + a pointer like `lib/services/filter.py:34-40` for the unchanged context | Almost always — compact yet reviewable |
| **inline** | The full changed code in the reply | Small change, or the user asked for it |
| **hybrid** | Summary + pasted key hunks + pointers for the rest | Large change where a few hunks matter most |

**The lines that CHANGED are always pasted.** Only **unchanged context** becomes a
pointer. A pointer with nothing pasted is **not** a compact review — it is a
withheld one. **Meaning first, then the changed code, then a pointer for the rest.**

---

### 0.2 — Then: is the task clear and low-risk? (absorbs `spike` + `plan`)

If **it is not clear what to build**, or **the approach is risky**, do NOT start coding.

#### 0.2a — SPIKE (from `spike`) — when the answer is only findable by building

A spike is a **throwaway experiment** whose output is **knowledge, not
production code**. Use it when the user says *"let me try this"*, *"I want to
see if X works"*, *"is this even possible?"*, *"compare A vs B"*, or when a real
unknown (performance, cost, a third-party API's actual behaviour) blocks design.

**Do NOT spike when:** the answer is knowable from docs or by reading code
(*just research*), the work is on the production path (*plan it instead*), or
the idea is already validated (*just implement*).

**The loop:**

```
decompose  →  research  →  build  →  verdict
   ↑______________________________________↓
              iterate on findings
```

1. **Decompose** the idea into **2–5 independent feasibility questions** — one
   spike per question, each framed *Given / When / Then*, each with a risk label.
2. **Research** what is already known before building anything.
3. **Build** the smallest thing that answers the question. Keep it disposable.
4. **Verdict** — write the answer in one line per spike: *proven / disproven /
   still unknown*, and the evidence.

**Spike rules (this is where the old rule contradicted the file, so it is now explicit):**

- A spike's code is **not deliverable**. After the verdict, **do not leave it
  silently in the repo** — either delete it and start clean with TDD, or, if the
  user wants to keep it, **say so explicitly and mark it** so it cannot be
  mistaken for production code.
- **Never declare "it works" after one happy-path run.** Test the edges, follow
  the surprising result, and report the verdict honestly.
- **Answer honesty:** `unknown` is a valid verdict. Do not upgrade it to
  `proven` because you ran out of patience.

#### 0.2b — PLAN (from `plan`) — when the task needs agreement before code

Write a markdown plan and **execute nothing** in that turn. Required for tasks
**> 1 hour or > 3 files**, and **always for team repos**.

Save it under:

```
.hermes/plans/YYYY-MM-DD_HHMMSS-<slug>.md
```

(relative to the active working directory — Hermes file tools are backend-aware,
so this keeps the plan with the workspace on local, docker, ssh and cloud
backends). If the runtime names a specific path, use that exact path.

**A plan is a PHASE of this workflow, not an alternative to it.** The moment the
user says *"now implement it"*, this workflow resumes at PHASE 0 — the plan does
**not** lock the session into "never write code".

**Write the plan assuming the implementer has zero context** and questionable
taste. Document everything they need: which files to touch, the code, the test
commands, docs to check, how to verify.

**Bite-sized tasks:** each task = **2–5 minutes of focused work**, one action each.
*"Write the failing test"* = one step. *"Run it and watch it fail"* = the next.

**Too big:** `### Task 1: Build authentication system` (50 lines, 5 files)
**Right size:** `### Task 1: Create User model with email field` (10 lines, 1 file)

**Plan structure:** Goal · current context / assumptions · proposed approach ·
step-by-step tasks · files likely to change · tests / validation · risks,
trade-offs and open questions.

**Behaviour while planning:** if the request is clear enough, write the plan
directly; if it is genuinely underspecified, ask **one** brief question instead
of guessing; after saving, reply briefly with what you planned and the path.

#### 0.2c — ASK THE USER — when the *goal itself* is vague

Do not guess and then build the wrong thing.

**Signs you need the pre-phase:** you cannot write one sentence of the form
*"done means ..."*; or 2+ approaches look equally reasonable; or there is
external uncertainty (performance, cost, a third-party API).

> **Anti-pattern:** starting work, then asking about repo ownership only when you
> reach the commit step. By then the analysis depth, review requirement, and branch
> decision were all made on an assumption you never checked.

---

## PHASE 0 — Load Context (mandatory, in order)

Load these **before** you touch code, in this order. Say out loud which ones you
skipped and why.

1. **The user's instruction** — re-read it. Restate the goal in one sentence.
2. **`SOUL.md` / project rules** — identity, standing rules, house style.
3. **Relevant skill(s)** — `skills_list()` → `skill_view('<name>')` for anything
   touching this task's tools or domain.
4. **The code you are about to change** — read it *whole*, not just the diff
   context. Read the callers and the tests too.
5. **The project's own conventions** — lint config, test command, commit style.
   Follow them; do not invent parallel ones.

**Optional but strongly recommended when available:**

- **`codegraph` / IDE call hierarchy** — before editing a shared function, look at
  its **blast radius**: who calls it, what it returns to, what breaks if the
  signature or behaviour changes. When no codegraph is available, `grep -rn "<fn>" .`
  is a perfectly good substitute — **do it**.
- **Context7 / official docs** — never edit third-party API usage from memory.

> **Anti-pattern:** loading the file, reading three lines around the cursor, and
> "fixing" it. You will break the caller you never looked at.

---

## PHASE 1 — Analysis & Root Cause (absorbs `systematic-debugging`)

### 1.1 — The Iron Law — true for features, mandatory for bugs

```
NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST
```

If the task is a **bug, a failure, or unexpected behaviour**, you may **not**
propose a fix before completing this phase. Symptom fixes are failure — they
return as the same bug, later, wearing a different hat.

Use this phase for **any** technical issue: test failures, production bugs,
unexpected behaviour, performance problems, build failures, integration issues.

**Use it ESPECIALLY when:** the fix "seems obvious"; you are under time
pressure; you have already tried one or more fixes; the previous fix did not
work; or you do not fully understand the issue.

**Do not skip it because:** the issue seems simple (simple bugs have root
causes too), you are in a hurry (rushing guarantees rework), or someone wants it
fixed **now** (systematic is *faster* than thrashing).

### 1.2 — The Feedback Loop Rule (the strongest idea absorbed into v3.0)

> **The feedback loop is the debugging work.**

Before you read code to build a theory, **create or identify a tight command
that goes RED on the user's exact symptom and GREEN when the bug is fixed.**

A **tight** loop is:

- **fast** — seconds, so you can run it dozens of times;
- **deterministic** — same result every run (for flaky bugs: raise the repetition
  count until it is reliable, or force the race with a sleep/hook);
- **agent-runnable** — one command, no hand-holding;
- **specific** — it fails on **this** bug, not merely "doesn't crash".

**If a clean reproduction is hard, spend disproportionate effort building the
loop.** Guessing without a red-capable loop is exactly the failure mode this
whole phase exists to prevent.

**Report the loop to the user before using it:** the command, and the observed
RED output. A loop that has not been seen to fail has proven nothing.

### 1.3 — Root Cause Investigation (do these, in order)

1. **Read the error messages carefully.** Do not skip warnings — they often
   contain the answer. Read the stack trace **completely**: line numbers, file
   paths, error codes. `search_files` for the error string in the codebase.
2. **Build the tight feedback loop** (§ 1.2) and watch it fail.
3. **Trace the data.** Where does the wrong value *enter*, and where does it
   first *become* wrong? Fix the **origin**, not the last place it looked wrong.
4. **Check recent changes.** `git log`, `git diff`, a dependency bump, a config
   change. Most regressions have a commit attached.
5. **Compare a working case to the broken one.** What is different? Diff the two
   paths explicitly rather than reasoning in the abstract.
6. **State the root cause in one sentence, and name the evidence.** If you cannot,
   you have not finished — go back to step 1.

**Only then** propose a fix. The fix must address the **stated root cause**, and
you must be able to explain **why the fix works**, not merely that the symptom
went away.

### 1.4 — The Rule of Three (when a fix does not work)

| Attempt | What to do |
|---|---|
| 1st fix fails | Re-read the error. Re-run the loop. Adjust the **theory**, not the code. |
| 2nd fix fails | **Stop.** You are guessing. Go back to § 1.3 step 1 with fresh eyes. |
| 3rd fix fails | **Stop and escalate.** Report honestly: what you tried, what you observed, what you still do not know. Ask for help or more context. |

Thrashing past three attempts is the clearest signal that the root cause was
never found. **Say so** — an honest "I have not found the root cause" is worth
more than a fourth guess.

### 1.5 — Analysis of a non-bug task (feature / refactor / change)

When the task is **not** a bug, Phase 1 still applies — it just answers
different questions:

1. **What exactly is being asked?** Restate it; confirm the *"done means ..."*
   sentence exists. If it does not, go back to § 0.2c and ask.
2. **What is the current behaviour, and what is the target behaviour?** Both, in
   concrete terms (input → expected output).
3. **What is the blast radius?** Who calls this, what depends on it, what else
   could break? (Use codegraph / grep.)
4. **What are the constraints?** Performance, compatibility, dependencies,
   existing conventions, the user's stack.
5. **What are the options, and which one is *effective & efficient*?** Name at
   least the alternative you rejected and why. (§ THE LOCK rule 1.)
6. **What could go wrong?** Failure modes, edge cases, the input nobody tested.

**Do not proceed to Phase 3 with a question you could have answered by reading
the code.** And **do not** turn analysis into an excuse for delay — the goal is
understanding, not a longer document.

---

## PHASE 2 — Reference (only if an external library/API is involved)

Skip this phase **only when no third-party library or API is involved** — and
**say that out loud** when you skip it.

- **Never write third-party API usage from memory.** Fetch the current docs
  (Context7, official docs, the installed package's own source).
- **Check the installed version.** The docs for `latest` and the version in
  `requirements.txt` / `package.json` disagree more often than you think.
- **Prefer the documented call over a clever workaround.** If you must work
  around the documented path, write down why.
- **Cite what you relied on** — the doc URL or the file you read — so the user
  can check it.

---

## PHASE 3 — Execution (TDD-first, per-function cycle)

### 3.0 — The TDD Iron Law (mandatory — absorbed from `test-driven-development`)

```
NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST
```

**Write the test first. Watch it fail. Then write the minimal code to pass.**

**Core principle:** *if you did not watch the test fail, you do not know that it
tests the right thing.* A test that has never failed has proven nothing.

**Always:** new features · bug fixes · refactoring · behaviour changes.
**Exceptions (ask the user first, and say so out loud):** throwaway prototypes ·
generated code · configuration files.

**One honest refinement of the absorbed rule** (see § THE LOCK rule 1): the old
skill said *"wrote code before the test? delete it, start over."* That stays true
for **production code that drifted ahead of its test.** It does **not** apply to
**a spike that already produced a verdict** (§ 0.2a) — there the knowledge is the
deliverable, and the *spike code* is discarded, not the *finding*. Be precise
about which one you are doing, and say which.

### 3.1 — RED → GREEN → REFACTOR

| Step | Do | Do not |
|---|---|---|
| **RED** | Write **one** minimal test showing what should happen. Run it. **Watch it fail** — for the right reason. | Write several tests at once; write a test that passes immediately. |
| **GREEN** | Write the **minimum** code that makes it pass. Run it. | Add unrequested features; optimize early. |
| **REFACTOR** | Clean up with the test green. Run it again. | Change behaviour while refactoring. |

**A good test:** clear name, tests **real** behaviour, one thing, fast,
deterministic. **A bad test:** asserts on implementation details, tests a mock
instead of the code, name says nothing, passes when the feature is broken.

**If the test passes the first time you run it,** the test is wrong — it is not
exercising what you think. Fix the test before touching the code.

### 3.2 — Choose the test type before you write it

Pick the **smallest** test that proves the behaviour. Do not reach for a heavy
kind when a lighter one answers the question. (Full catalogue: **Appendix A**.)

| Level | Proves | Cost |
|---|---|---|
| **Unit** | one function/unit behaves correctly | lowest — default choice |
| **Integration** | units work together (db, fs, network boundary) | medium |
| **End-to-end** | the user-facing flow works | highest — use sparingly |

**Rule of thumb:** one unit test per new branch/behaviour; an integration test
when a boundary is actually crossed; an E2E test only when a real user path is
at stake. **Do not** build a test pyramid out of habit — build the tests that
would **actually catch a regression here**.

### 3.3 — The per-function cycle — 8 steps (never skip steps 6 or 7)

For **each** function / unit of work, in this exact order:

| # | Step | Meaning |
|---|---|---|
| 1 | **WRITE** | Write the failing test first (§ 3.0), then the code. |
| 2 | **WORKS** | Run it on the **happy path**. Confirm the intended result. |
| 3 | **TEST** | Run the **edges**: empty, null, zero, negative, huge, unicode, duplicate, concurrent. |
| 4 | **AUDIT** | Re-read your own change as a stranger: naming, dead code, error handling, secrets, logging of PII. |
| 5 | **LIVE** | Run the **real** entry point (the app, the CLI, the endpoint) — not just the test. |
| 6 | **EXPLAIN** | Explain it back to the user (§ 3.8.1) — the *meaning*, not a file list. |
| 7 | **LINK** | State what changed, where, and how to undo it (§ PHASE 5). |
| 8 | **NEXT** | Move to the next unit — only after 6 and 7 are done. |

> **Steps 6 and 7 may NEVER be skipped** — not for a deadline, not for a
> "one-liner", not because the change "obviously can't break anything". These
> are the two steps where the user catches a wrong intent **before** it ships.

### 3.4 — Scope discipline

- **One logical change at a time.** If you notice an unrelated bug, **write it
  down** and finish the current change first. Do not "while I am here" your way
  into a 400-line diff.
- **Do not reformat untouched lines.** It hides the real change from the reviewer.
- **Match the surrounding code.** Its naming style, its error style, its test
  style. A "better" style that contradicts the file is a new bug in review.
- **Delete dead code you created.** Do not leave commented-out blocks; git is
  the archive.
- **Never commit secrets.** No keys, tokens, `.env` values, or connection strings
  in code, tests, fixtures or logs. If you touch one by accident, **say so
  immediately** — do not quietly remove it and hope.

### 3.8.1 — EXPLAIN: explain every finished unit back

**Not a bare file list.** The *meaning*:

1. **What it does** — in the user's language, not the compiler's.
2. **How** — the approach, briefly.
3. **Why this way** — including the alternative you rejected, and why.
4. **How it was verified** — the loop/test you ran, and its result (red → green).
5. **One thing the user can try** — a concrete command or click.
6. **Where you are least confident** — say it plainly.

**The user is the only reviewer who knows what the code was supposed to
accomplish.** If you never explain it, you remove their only chance to catch a
misunderstanding early. **A task is not "done" until the user can review it.**

Present it in the mode chosen at § 0.1b, and deliver it in the mode chosen at
§ 0.1c — **meaning first, then the changed lines, then a pointer for the rest.**

---

## PHASE 4 — Verification (the Impact-Audit Gate)

### 4.1 — Run the tests. Report the truth.

- Run the tests **you wrote** and the **existing** suite. Both.
- **Baseline-aware:** capture the failure count **before** your change
  (`git stash` → run → `git stash pop`). **Only NEW failures block the commit.**
  Do not claim you broke something that was already broken — and do not
  hide a new failure behind an old one.
- **Red is red. Skipped is skipped. Unknown is unknown.** Never write "all tests
  pass" when any of them failed, were skipped, or were never run.
- If you could not run the tests, say **why**, and say what that means for
  confidence. "I did not run it" is an acceptable answer. "It should work" is not.

### 4.2 — Run the real entry point

Tests prove the unit. They do **not** prove the feature. Run the actual thing:

- the CLI command, the endpoint, the page, the app;
- with **realistic** input, not only the fixture;
- and look at the result with your own eyes, not just the exit code.

**A green test suite and a broken app is a normal outcome** — it is exactly what
step 5 of the per-function cycle (§ 3.3) exists to catch.

### 4.3 — Impact-Audit Gate (the reviewer's checklist)

Answer these six, in order. **Any "no" blocks Phase 5.**

| # | Question | If the answer is "no" |
|---|---|---|
| 1 | Do the tests pass — **including the ones I did not write**? | fix it, or report it as a **new** failure |
| 2 | Did I run the **real** entry point, not just tests? | go do it now |
| 3 | Did I check the **blast radius** — every caller of what I changed? | grep the callers before you commit |
| 4 | Is the change **exactly** what was asked — no scope creep? | split out the extra work |
| 5 | Can I **undo** this cleanly, and do I know how? | create the branch/rollback path first |
| 6 | Would I be comfortable if the user read **every line** of this diff? | rewrite the part that makes you hesitate |

### 4.8 — Static security scan (MANDATORY before every commit)

**Scan added lines only.** Any match is a finding: fix it, or explain why it is
safe — **in writing**, to the user.

```bash
# Hardcoded secrets
git diff --cached | grep "^+" | grep -iE "(api_key|secret|password|token|passwd)\s*=\s*['\"][^'\"]{6,}['\"]"

# Shell injection
git diff --cached | grep "^+" | grep -E "os\.system\(|subprocess.*shell=True"

# Dangerous eval/exec
git diff --cached | grep "^+" | grep -E "\beval\(|\bexec\("

# Unsafe deserialization
git diff --cached | grep "^+" | grep -E "pickle\.loads?\("

# SQL injection (string formatting in queries)
git diff --cached | grep "^+" | grep -E "execute\(f\"|\.format\(.*SELECT|\.format\(.*INSERT"
```

**Also check, by eye:** private keys, `.env` values, connection strings with
credentials, absolute paths containing a username, PII in logs or fixtures,
secrets in test files (they get published too).

> **If you ever find a real secret in a commit that was already pushed:**
> say so **immediately and loudly**. Removing it in the next commit does **not**
> un-leak it — the credential must be **rotated**, and the user must be told.
> Quietly deleting it is the worst possible response.

### 4.9 — Optional: the four-lens review (from `simplify-code`)

For a change big enough to deserve it, re-read your own diff through four lenses
**in this order** — output is a list of findings, not a rewrite:

| Lens | Ask |
|---|---|
| **1. Reuse** | Does this duplicate something that already exists here? (the most common waste) |
| **2. Quality** | Naming, dead code, error handling, unclear control flow |
| **3. Efficiency** | Redundant work, needless allocations/queries/loops, N+1 |
| **4. Altitude** | Is the change at the right level — too much in one place, or spread too thin? |

**Optional by design** — it is a second pair of eyes, not a gate. Findings go
into § 4.10 or the report; they do not silently expand the diff.

### 4.10 — Anti-bloat review — the ladder (from `ponytail`), with the guard

**Climb the ladder after you understand the problem — never instead of it.**

1. **Does this need to exist at all?** A speculative need: skip it, and say so in one line. (YAGNI)
2. **Already in this codebase?** A helper/type/pattern that already lives here → reuse it.
3. **Does the standard library do it?** Use it.
4. **Does a native platform feature cover it?** (`<input type="date">` over a picker lib, CSS over JS, a DB constraint over app code.)
5. **Does an already-installed dependency solve it?** Use it. Do not add a new one for what a few lines can do.
6. **Can it be one clear line?** Then one line.
7. **Only then:** the minimum code that works.

**And the guard — read it twice, it overrides rung 6:**

> **EFFECTIVE & EFFICIENT > MINIMUM. Short code is NOT the goal.**
>
> Rung 6 is a **hint to check**, never an instruction to obey. Rungs 1–5 are
> about **not inventing** things that already exist — that part is always right.
> But when the shortest form is **unreadable, fragile, or hard to maintain**,
> **the longer version is the correct one. Take it, and say why.**
>
> "The best code is the code never written" is true for **speculative** code.
> It is **false** for code the user actually needs.

**Also absorbed from the same source, because it is simply correct:**

> **Bug fix = root cause, not symptom.** Before editing, **grep every caller** of
> the function you are about to touch. One guard in the shared function beats a
> guard in every caller — and patching only the path the ticket names leaves
> every sibling caller still broken. **Fix it once, where all callers route through.**

### 4.12b — Independent review (never verify your own work)

> **Core principle: no agent should verify its own work. Fresh context finds what you miss.**

Do **not** call Phase 4 complete on the strength of your own re-read. Get a
**genuinely independent** pass, in this order of preference:

1. **A subagent with no memory of writing the code** — give it the diff and the
   requirement, ask it to find problems, and do **not** tell it what you think is
   fine.
2. **A linter / static analyser / type checker** — objective, cheap, no ego.
3. **The test suite** — if it can fail on this bug, it is a reviewer.
4. **At minimum: a fresh read of the diff** — after a break, as if a colleague
   wrote it.

**This is the rule that the absorbed `requesting-code-review` skill broke:** it
offered an **auto-fix loop**, which makes the reviewer the author — exactly what
this rule forbids. **A reviewer may report. The author fixes.** If you
automatically fix your own findings, you have not had a review.

**Report the review result honestly**, including "the independent pass found
nothing" — an empty review is a result, not a failure.

### 4.13 — The rule of three, for review loops

| Round | What to do |
|---|---|
| 1st review finds issues | Fix them. Re-verify (§ 4.1). |
| 2nd review finds issues | Fix them — but **look for a pattern**. Two rounds of the same class of problem means the task was misunderstood. |
| 3rd round | **Stop.** Do not enter a fourth. Report honestly: what keeps coming back, and that you suspect the scope or the requirement is the real problem. |

---

## PHASE 5 — Version Control & Deploy (LOCKED until the Impact-Audit Gate passes)

### 5.1 — Pick your mode (from the answer you already got in PRA-PHASE 0.1)

| Repo | Track |
|---|---|
| **Solo / throwaway** | Direct on the working branch is acceptable. Self-review allowed **only if you say so**, and **never on a bug fix**. |
| **Team / shared / public** | Branch, PR, CI, **independent** review. No exceptions. |
| **When unknown** | Treat as **team**. |

### 5.2 — Branch (unless solo/throwaway)

```bash
git checkout -b <type>/<short-slug>      # feat/, fix/, chore/, docs/, refactor/
```

Use a descriptive slug, not your name or a date. One branch = one logical change.

### 5.3 — Commits: one logical change per commit (atomic)

- **Atomic:** each commit does **one** thing and passes the tests **on its own**.
- **Message shape:** imperative subject ≤ 72 chars, then a body with **why**
  (the *what* is in the diff).
  `fix(filter): keep glute_bridge when the user has a knee injury`
- **No mixed content:** do not put a refactor and a behaviour change in one
  commit — they need to be revertible separately.
- **Never commit:** secrets, `.env`, build output, editor files, big binaries,
  or a file you have not read.

### 5.4 — Semantic Versioning (SemVer)

| Change | Bump |
|---|---|
| Backwards-incompatible | **MAJOR** |
| New capability, compatible | **MINOR** |
| Bug fix / internal only | **PATCH** |

### 5.5 — Tag + CHANGELOG (release only)

- Update `CHANGELOG.md` under a new heading `## [x.y.z] - YYYY-MM-DD`.
- Use the **real** date. **Get the date from the system clock, not from memory** —
  a wrong release date is a small lie that ships to everyone.
- Tag **after** the changelog commit: `git tag -a vX.Y.Z -m "..."` → push the tag.
- **The changelog entry and the tag must agree.** If they disagree, the changelog is wrong.

### 5.6 — Rollback: know the way home *before* you need it

- Know the **undo** for every step, and state it: `git revert <sha>` for a pushed
  commit, `git checkout -- <file>` for uncommitted, the tag/branch to return to.
- **Never rewrite shared history.** Once pushed to a shared branch, **fix forward
  with `git revert`.**
  **`reset` · `rebase` · `push --force` on a shared branch is forbidden** — that is
  how one mistake becomes everybody's problem.
- Before anything destructive: **state what will be lost, and get a yes.**

### 5.7 — Coding journal

Append an entry per solved problem (format in **Appendix A**). The journal is how
the *next* session — and future you — avoids re-solving the same problem. It is
written **after** the change is verified, never instead of verifying it.

### 5.8 — Deploy + verify

1. Deploy the **exact** artifact you tested (not a rebuilt one).
2. Verify the deployed thing: hit the endpoint, load the page, run the command.
3. Watch the logs/errors for a few minutes.
4. Report what you deployed, where, and the evidence that it works.

### 5.9 — Pull request lifecycle (absorbs `github-pr-workflow`)

**Auth and repo detection first:**

```bash
gh auth status                          # is gh authenticated?
git remote -v                           # owner/repo for the PR target
```

**The loop:**

1. **Push the branch:** `git push -u origin <branch>`
2. **Open the PR** with a real body: what changed, why, how it was verified, and
   a **test plan** the reviewer can follow.
3. **Watch CI.** Do not walk away and assume green:
   ```bash
   gh pr checks --watch                    # or: gh run list --branch <branch>
   ```
4. **Fix CI failures honestly.** Get the failure detail (`gh run view --log-failed`),
   fix the *cause*, push, and re-verify. **Do not** re-run a flaky job and call it
   fixed until you know why it failed once.
5. **Merge** only when checks are green and review is done. Prefer a **squash or
   merge commit** consistent with the repo's history — do not introduce a new style.
6. **After merge:** delete the branch, pull the base branch, and confirm the
   deployed/merged state.

**References for this section** (read, do not improvise):
`references/github/conventional-commits.md` (commit format) ·
`references/github/ci-troubleshooting.md` (a red CI run that will not go green) ·
`references/github/templates/pr-body-feature.md` ·
`references/github/templates/pr-body-bugfix.md` (PR body skeletons).

**Never:** force-push a shared PR branch · merge with red checks because the
deadline is close · describe the CI state as green when you did not look.

### 5.10 — From an issue to a PR (absorbs `github-issue-to-pr`)

When the task arrives as an **issue** (not a direct request), the risk is
different: the issue may be **stale, already fixed, or based on a wrong premise.**

1. **Read the live issue** — body **and the full thread**. Comments often contain
   the real requirement, or a maintainer's decision that overrides the title.
2. **Sweep for duplicates** — search existing PRs/branches for work already done.
   Do not build a second implementation of the same fix.
3. **Validate the premise** against the **current** code — and against design
   intent. If the reported behaviour is **correct by design**, say so and stop;
   do not "fix" a feature.
4. **Define acceptance and risk** before coding: what proves it fixed, what could
   break.
5. **Implement the smallest complete change** — and **fix the class, not the
   instance** (see § 4.10: one guard where all callers route through).
6. **Prove the regression test bites — the sabotage run.** Sabotage the fix
   (revert it, or break the line) and confirm the test **goes red**. A test that
   stays green under sabotage is not testing the bug. **Restore the fix, confirm
   green.** Report both colours.
7. **Run the repo's own quality gates**, then open the PR **immediately** — do
   not let a verified fix sit unshared.
8. **Shepherd CI honestly** (§ 5.9 step 3–4) and **close the loop**: link the PR to
   the issue, state what shipped, and what remains open.

**Pitfall:** treating "the issue says X" as "X is true". Verify first.

### 5.11 — Porting a change across branches (absorbs `git-patch-porting`)

When a fix must land on an **older branch** (release line, fork, stale checkout),
**do not copy-paste the diff.** Port it and re-verify.

- **Core rule: a ported patch is a new change, not a copy.** It must pass **its
  own** tests on the target branch.
- Use `git cherry-pick <sha>` when the branches are close; resolve conflicts
  deliberately. For distant branches, re-implement the *intent*, not the text.
- **Do not port tests blindly** — the target branch may have a different harness.
- Verify on the target: run its suite, run its entry point. Report the target's
  result, not the source's.
- **Pitfall:** assuming "same code = same behaviour" when a dependency, config or
  schema differs. Check the versions.

**Reference for this section:** `references/git/stale-checkout-port.md` — the full
procedure for landing a patch in a stale checkout or fork.

### 5.12 — Resolving a merge conflict (absorbs `merge-reconciler`)

When two sides disagree, the failure mode is **silently picking a side** — or
worse, "resolving" by keeping both and breaking the code.

**Impartiality contract:** the goal is not "my side wins", it is **the correct
combined result.**

1. **Gather both sides.** Read the **full** intent of each: `git log --merge`,
   `git diff --ours`, `git diff --theirs`. Understand **why** each side changed.
2. **Classify every conflicted hunk:** *(a)* the same change on both sides
   (trivial), *(b)* genuinely different intent (needs a decision),
   *(c)* one side supersedes the other (keep the newer **intent**, not the newer text).
3. **Resolve deliberately.** For *(b)* and *(c)*, you are making a design
   decision — **state it and justify it**, out loud.
4. **Verify the result.** A conflict resolution is **code**: run the tests, run the
   entry point, and read the merged region — the classic failure is a merge that
   compiles and behaves differently.
5. **Hand back:** say which hunks took which resolution, and anything you had to
   decide for the user.

**Pitfalls:** resolving by "keep mine" · dropping the other side's test · a merge
that passes tests but changes behaviour · **and the classic: a conflict marker
left in the file.** Always `grep -rn "<<<<<<<" .` before you commit.

---

## PHASE 6 — Honest Report, Explanation & Journal

### 6.1 — The honest report (non-negotiable)

Report exactly what happened, in this shape:

| Field | Rule |
|---|---|
| **What I did** | The change, in one or two sentences of *meaning* |
| **How I verified it** | The **exact command** and its **observed result**, both colours |
| **What is red** | Failures, skips, warnings — **as they are** |
| **What I did NOT do** | Untested paths, skipped phases, unrun commands |
| **What I am unsure about** | The part where you are least confident |
| **What could break** | Blast radius, and how to notice |

**The three forbidden phrasings:**

- *"All tests pass"* — when any test failed, was skipped, or was never run.
- *"It should work"* — when you did not run it. Say **"I did not run it."**
- *"Done"* — before the user can review it (§ 3.8.1).

**Red is red. Skipped is skipped. Unknown is unknown.**
A silent skip is the failure this entire protocol exists to prevent.

### 6.2 — Explain every unit back

The six-part explanation from § 3.8.1 — meaning first, then the changed lines,
then a pointer for the rest, in the delivery mode the user chose (§ 0.1c).

### 6.3 — Journal entry

Append to the coding journal (**format: Appendix A**). Task · approach · what
went wrong · the lesson. Written **after** verification, never instead of it.

---

# ATURAN / RULES

These are the standing rules. Each one exists because its absence caused a real
problem — most of them in the session that produced v3.0.

- **Effective & efficient > minimum.** Do not be proud that the code is short; be
  proud that it hits the target, saves resources, and is safe. (§ THE LOCK 1)
- **Skipping a phase is allowed only when it genuinely does not apply** (e.g. no
  external library → skip PHASE 2). **Say out loud why you skipped it.** Silent
  skipping is the failure mode this whole protocol exists to prevent.
- **The per-function cycle steps 6 and 7 (EXPLAIN, LINK) and the Impact-Audit
  Gate (§ 4.3) may NEVER be skipped** — not for a deadline, not for a "one-liner",
  not because the change "obviously can't break anything".
- **Root cause before fix.** "Quick fix now, investigate later" is how you get the
  same bug twice. (§ 1.1–1.3)
- **Build the red-capable feedback loop before you read code to theorise.**
  (§ 1.2) A loop you have not seen fail has proven nothing.
- **Never verify your own work.** A reviewer may **report**; the **author fixes**.
  (§ 4.12b)
- **Honest reports only.** Red is red. Skipped is skipped. Unknown is unknown.
  (§ 6.1)
- **Every change must be reversible.** Branch, atomic commits, and a known
  rollback path are part of "done" — not optional polish. **If you cannot undo
  it, you do not control it.** (§ 5.6)
- **Never rewrite shared history.** Once pushed to a shared branch, fix forward
  with `git revert`. `reset` / `rebase` / `push --force` on shared branches is how
  one mistake becomes everyone's problem. (§ 5.6)
- **Establish repo ownership before writing code** (§ 0.1). **When unknown, treat
  it as a team repo.**
- **Match the review to the reviewer** (§ 0.1b). **Never hide the code from
  someone able to judge it; never dump code on someone who cannot. Never lock a
  willing learner out of code.**
- **Deliver the code the way the user asked** (§ 0.1c). The lines that **changed**
  are always pasted; only unchanged context becomes a pointer. **A pointer with
  nothing pasted is a withheld review, not a compact one.**
- **Fix the class, not the instance.** Grep every caller; fix once where all
  callers route through. (§ 4.10)
- **Scan for secrets before every commit** (§ 4.8). **If a real secret was
  already pushed, say so immediately — the credential must be rotated.**
- **Ask, do not guess** — when the goal itself is vague (§ 0.2c). Guessing wrong
  costs more than one question.

### If you are tempted to skip — read this table

| The temptation | The reality |
|---|---|
| "It's a one-liner" | One-liners cause outages. The cycle is minutes. |
| "Tests are obvious, I'll add them after" | A test written after the code tests the code, not the requirement. |
| "No time to find root cause" | You have time to fix it twice? Thrashing is slower than thinking. |
| "I'll explain later" | Later never comes, and the user loses their only chance to catch it. |
| "It's my own repo, no review needed" | You are the worst reviewer of your own bug. Get a fresh read. |
| "The user said hurry" | Say what you skipped. Then hurry *honestly*. |
| "I don't need to write this down" | Next session starts from zero. The journal is the only memory. |
| "It works on my machine" | Run the real entry point (§ 4.2), not the fixture. |

---

## Quick Reference

```
PRA-PHASE   0.0 vague idea?   → spec (brief)    0.1 who owns the repo?
            0.1b review level? 0.1c delivery?   0.2 clear & low-risk?
            └ 0.2a SPIKE · 0.2b PLAN · 0.2c ASK
PHASE 0     Load context (rules → skills → the code → conventions)
PHASE 1     Analysis & root cause   ← Iron Law + FEEDBACK LOOP + Rule of Three
PHASE 2     Reference — only with an external library/API
PHASE 3     Execution — TDD red→green→refactor + the 8-step per-function cycle
PHASE 4     Verification — tests · real entry point · IMPACT-AUDIT GATE ·
            secret scan · 4-lens · ladder (with guard) · INDEPENDENT review
PHASE 5     Git — branch · atomic commits · SemVer · rollback · PR · issue→PR
            (sabotage run) · porting · merge conflicts
PHASE 6     Honest report · explain back · journal
```

**The six minimum, always:**

```
1. Root cause before fix.
2. One function, one full cycle (write → works → test → audit → live → explain → link).
3. Never verify your own work.
4. Report honestly.
5. Explain every unit back to the user.
6. Every change undoable.
```

---

## Appendix A — Testing catalogue & journal format

### A.1 — Classify the task first (from `testing-strategy`)

**Pick the *minimum sufficient* set of test types. Do not blanket-apply all of
them — most tasks need 1–3.**

| Question | If yes → |
|---|---|
| Pure function / business rule, no I/O? | `unit` |
| Touches DB, filesystem, network, another service? | `integration` |
| User-visible multi-step flow? | `e2e` |
| Depends on an external API / clock / random / payment? | `test-doubles` |
| Money, auth, or data-loss-adjacent? | `property` + `unit` |
| UI component with stable render output? | `snapshot` |
| Consumes someone else's API? | `contract` |
| Hot path / will take real traffic? | `load` |
| Fixing a reported bug? | `regression` |
| Refactor with existing tests? | keep existing + `coverage` delta |

**Default matrix — start here, escalate only when evidence demands it:**

| Task shape | Minimum set |
|---|---|
| Pure logic (parsing, math, state machine) | `unit` |
| Service + DB | `unit` + `integration` (+ `fixtures`) |
| Service + third-party API | `integration` + `test-doubles` (+ `contract` if you own both sides) |
| REST/GraphQL endpoint | `integration` + `contract` |
| Auth / money / critical invariant | `unit` + `property` |
| User flow (signup → action → confirm) | `e2e` (1 happy path) + `integration` for branches |
| Bug fix | `regression` (**reproduce first — it must fail before the fix**) |
| Refactor | existing suite + `coverage` delta |
| UI component library | `snapshot` + `unit` for prop logic |
| High-traffic endpoint / batch job | `load` (**after** correctness is proven) |

> A CRUD endpoint does not need `load` + `e2e` + `contract` + `snapshot`.
> **Escalate only when evidence demands it.**

### A.2 — Order of writing (always)

1. **Reproduce:** for a bugfix, write the failing test **first** and run it. It
   **must** fail, for the **right** reason.
2. **Fastest layer first:** `unit` → `integration` → `e2e`.
3. **One behaviour per test.** Name states the behaviour, not the function.
4. **Arrange–Act–Assert**, blank line between blocks.
5. **No logic in tests** — no `if`, no loops over assertions. Parametrize instead.
6. **Run the suite.** Report the exact command and the result.

### A.3 — Full references (12 files, moved into `references/testing/`)

| Reference | Use for |
|---|---|
| `references/testing/unit.md` | isolated logic, no I/O |
| `references/testing/integration.md` | real DB/HTTP/FS boundaries |
| `references/testing/e2e.md` | full user flow through the real stack |
| `references/testing/test-doubles.md` | stubs, fakes, spies, mocks |
| `references/testing/flaky.md` | diagnosing nondeterminism |
| `references/testing/coverage.md` | measuring and reading coverage |
| `references/testing/fixtures.md` | setup/teardown, factories, temp state |
| `references/testing/property.md` | hypothesis / fast-check invariants |
| `references/testing/snapshot.md` | golden files, UI rendering |
| `references/testing/contract.md` | consumer-driven API contracts |
| `references/testing/load.md` | throughput, latency, soak |
| `references/testing/regression.md` | reproducing fixed bugs permanently |

### A.4 — High-yield invariants for money / critical code

When the code moves money or guards a critical invariant, assert these with
`hypothesis` over **generated** inputs, not a handful of hand-picked cases:

- **Sign errors.** A "max loss" check written `abs(pnl) > limit` also trips on
  **profit** — the bot stops trading exactly when it is winning. Assert: profit
  never trips a loss guard.
- **Division by an unguarded input.** Guards often check one denominator
  (`avg_loss`) and forget another (`avg_win`). Assert the function never raises
  `ZeroDivisionError` across the input domain.
- **Guard the value you actually divide by**, not the raw input.

These properties find real bugs fast — they are worth the setup cost.

### A.6 — GitHub & git references (5 files, moved into `references/`)

| Reference | Use for |
|---|---|
| `references/github/conventional-commits.md` | commit message format — types, scope, breaking changes |
| `references/github/ci-troubleshooting.md` | diagnosing a red CI run (auth, cache, flake, runner) |
| `references/github/templates/pr-body-feature.md` | PR body skeleton for a feature |
| `references/github/templates/pr-body-bugfix.md` | PR body skeleton for a bugfix |
| `references/git/stale-checkout-port.md` | porting a change into a stale checkout / fork |

**How to use them:** when the task reaches PHASE 5 and the detail matters, read
the matching file — do **not** reconstruct commit formats or PR bodies from
memory. The templates in particular exist so the PR body is consistent and
complete on the first try.

### A.5 — Coding journal format

Append to the journal (see PHASE 5.7) one entry per solved problem:

```markdown
## [YYYY-MM-DD] <short-title>

**Task:** what was asked, in one line.
**Approach:** what you did, in 2–4 lines.
**Problems:** what went wrong, and the root cause once found.
**Lesson:** the reusable insight — the part future-you needs.
**Files:** the paths touched.
```

**Append only — never rewrite earlier entries.** The journal is a record, not a draft.

---

## Appendix B — Common failures (each one real)

| Failure | What it looks like | The rule that prevents it |
|---|---|---|
| **Symptom fix** | Bug "fixed", returns in a week | § 1.1 Iron Law |
| **Guessing without a loop** | Multiple fixes, none stick | § 1.2 Feedback Loop |
| **Self-review blindness** | "I checked it myself" | § 4.12b independent review |
| **Green tests, broken app** | Suite passes, feature fails | § 4.2 run the real entry point |
| **Silent phase skip** | Nothing mentioned, a phase absent | § RULES — say why you skipped |
| **Optimistic report** | "All tests pass" (some skipped) | § 6.1 honest report |
| **Scope creep** | Diff much bigger than the request | § 3.4 one logical change |
| **Fake test** | Test passes even with the bug present | § 5.10 sabotage run |
| **Leaked secret** | Key in a commit, quietly removed | § 4.8 scan + **rotate + tell** |
| **Force-push a shared branch** | Teammates' work lost | § 5.6 never rewrite shared history |
| **Conflict marker shipped** | `<<<<<<<` in the file | § 5.12 grep before commit |
| **Wrong release date** | CHANGELOG date from memory | § 5.5 take the date from the clock |
| **Un-revertible change** | No branch, no rollback path | § 5.6 undoable by design |
| **Stale-issue fix** | "Fixed" something already correct | § 5.10 validate the premise |

---

## Appendix C — Tool mapping (non-Hermes runtimes)

Already given in full near the top of this file (**Tool Mapping**). Summary:

- `skills_list()` / `skill_view()` → read your runtime's skills/rules directory,
  or skip and use this file alone.
- `codegraph` / call hierarchy → LSP references, IDE call hierarchy, or `grep -rn`.
- `terminal`, `read_file`, `search_files` → your runtime's shell and file tools.
- `delegate_task` → your runtime's subagent/Task mechanism; otherwise do it yourself.

**Nothing in this file requires a specific tool.** The discipline is portable.

---

## Appendix D — Complexity scale: sizing the ceremony

**This appendix is where "effective & efficient > minimum" becomes a number.**

Match the *ceremony* to the *change*. Over-ceremonising a one-line fix wastes the
user's time as surely as under-ceremonising a payment path.

| Scale | Looks like | Required | Skippable (say why) |
|---|---|---|---|
| **S1 — Trivial** | 1 file, ≤ 10 lines, no logic change (typo, text, config value) | Read the file · make the change · run the real entry point · honest report | PRA 0.0/0.2 · PHASE 2 · security scan (unless a secret-adjacent file) |
| **S2 — Small** | 1–2 files, one function, clear behaviour | Full per-function cycle · unit test · honest report | PRA 0.0 · PHASE 2 · 4-lens |
| **S3 — Moderate** | 2–5 files, several functions, some new behaviour | Full workflow · unit + integration · Impact-Audit Gate · independent review | PRA 0.0 · `load`/`e2e` unless the path is user-visible |
| **S4 — Large** | > 5 files, new module, cross-cutting change | Full workflow · plan first (§ 0.2b) · independent review · PR · branch | Nothing except PRA 0.0 if the spec exists |
| **S5 — Risky** | Money, auth, data migration, irreversible, shared/production | **Everything**, no exceptions · spike where uncertainty exists · extra independent review · explicit rollback plan the user agrees to | **Nothing.** State the rollback plan before starting. |

**How to use this table:**

1. **Size the change honestly, before starting.** Say the size out loud to the user.
2. **Size up, not down, when unsure.** S5 is not "more careful than needed" — it
   is the correct size for the risk.
3. **Sizing down requires the user's agreement** when the change touches money,
   auth, or data.
4. **Never use "it's small" to skip the security scan or the honest report.** Those
   are S1-and-up.

> **This is the guard on the whole anti-bloat apparatus.** The ladder (§ 4.10)
> and the minimum-set thinking in Appendix A exist to stop you **inventing**
> work — not to make you the author of fragile code. **Effective & efficient
> means the *right* size, not the *smallest* size.**

---

## Appendix E — Why this file has no dependencies

**v3.0 exists because separate skills contradicted each other.** They were written
at different times, by different authors, for different purposes, and nothing
reconciled them. The result was an agent that could follow two opposite
"mandatory" instructions in the same session.

**Consolidating them is the fix.** One file cannot contradict itself. Every rule
above has been compared against every other rule in this file, and the
contradictions were resolved **explicitly** — with the resolution written down:

| Old contradiction | Resolution in v3.0 |
|---|---|
| `plan` said "never implement" · workflow said "always implement" | Plan is a **phase** (§ 0.2b), not a competing authority |
| `spike` said "delete the code" · workflow said "every change undoable" | Spike code is **discarded, never left silently**; the **finding** is the deliverable (§ 0.2a) |
| `ponytail` said "one line!" · user said "short code is not the goal" | The ladder is a **hint**; **rung 6 has a guard** (§ 4.10) |
| `requesting-code-review` had an auto-fix loop · workflow said "never verify your own work" | **A reviewer reports; the author fixes** (§ 4.12b) |
| `testing-strategy` depended on 12 reference files | References **moved into this skill** (§ A.3) — no lost pages |

**If you add to this file, keep that property.** When a new rule disagrees with an
old one, **do not ship both** — resolve it, write the resolution down, and keep
this table current.

---

## License

MIT — © Fandi Iswara Saputra (@fandimetall)
