---
name: coding-workflow
description: "Master coding workflow: ask repo ownership, review level (expert/learning/plain) and delivery mode (inline/pointer), run 6 phases, then explain each unit — pasting the changed lines and pointing at the rest — so the user can catch wrong intent, inefficient code, unexplained code. Portable."
version: 2.6.0
author: Fandi Iswara Saputra (@fandimetall)
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [coding, workflow, testing, tdd, qa, debugging, code-review, quality-gate, portable]
    related_skills: []
---

# Coding Workflow — Master Protocol

> **One file, no dependencies.** This skill is fully self-contained: the debugging,
> testing, QA, and review discipline you need is embedded below — not referenced
> out to other skills you may not have installed.
>
> If you *do* have richer dedicated skills (e.g. a full QA harness), use them. This
> file is the floor, not the ceiling.

**MUST run on EVERY coding / problem-solving task, without being asked.**

---

## When to Use

Load at the START of every coding task: fixing bugs, adding features, refactoring,
debugging, reviewing. Also load when the user says *"fix this"*, *"add a feature"*,
*"why is this erroring"*, or any programming request.

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

## PRA-PHASE — Before Writing Code (mandatory questions + when the task is unclear)

### 0.1 — Ask this FIRST, every time: who does this repo belong to?

**Before any coding starts, establish the ownership context.** You cannot pick the
right level of ceremony without it, and guessing wrong costs either wasted hours
(solo work treated like a team) or broken colleagues (team work treated like solo).

**Ask the user directly — do not assume from the task size:**

> *"Is this a personal repo or a shared/team repo? Will anyone else read, clone,
> or depend on this code?"*

If the user's answer is genuinely unguessable and they are unavailable, **state your
assumption out loud and pick the stricter mode**. Never leave it silent.

| If the answer is… | Then for the rest of this workflow |
|---|---|
| **Personal / solo repo** | Blast radius is mostly you. Self-review is acceptable. Commit to `main` is fine. Light ceremony. |
| **Team / shared / public repo** | **Blast radius is mandatory** (PHASE 0.3) — other people's code calls yours. **Review must be independent** (PHASE 4.12b). **Always branch** (PHASE 5.2). Never rewrite shared history. |
| **Unknown / will be shared later** | **Treat as team.** The cost of extra caution is minutes; the cost of a broken shared history is an incident. |

### 0.1b — Ask second: what review level does the user want?

**Repo ownership (0.1) is about the code. This question (0.1b) is about the
reviewer.** They are different things and you need both — a solo repo can still have
an expert owner who wants to read every line, and a team repo can have a non-technical
product owner who only wants to know what changed.

**⭐ There are TWO separate questions here — do not collapse them into one:**

```
Question A: CAN they judge the code?     → who reviews it
Question B: Do they WANT to understand it? → is this a teaching opportunity?
```

These are **independent**. A beginner who wants to learn is **neither** an expert
**nor** a plain reader — and treating them as plain actively prevents them from ever
becoming an expert. **Ask both.**

**Ask the user directly:**

> *"Two things, so I pitch this right: (1) do you read code yourself, or would you
> rather I just describe what it does? (2) do you want to understand how the code
> works as we go — even if you're not reading it fluently yet?"*

| Can judge? | Wants to learn? | **Mode** | What you deliver per unit |
|---|---|---|---|
| Yes | — | **Expert** | Plain walkthrough + **the code** + technical self-critique (PHASE 3.8.1b) |
| No, **but wants to learn** | Yes | **⭐ Learning** | Plain walkthrough + **annotated code** + the concept behind it + what to study + an open invitation to ask (PHASE 3.8.1c) |
| No | No | **Plain** | Plain walkthrough only (PHASE 3.8.1) |
| Not stated, cannot be asked | — | **Default to Expert**, leading with the plain explanation | Both |

**Detect from behaviour too** — but **never override an explicit answer**:
- Signals for **Expert**: uses terms like *refactor, complexity, query, async, type,
  regex*; asks *"how did you implement it?"*; corrects your technical choice.
- Signals for **Learning**: asks *"what does this line do?"*, *"why did you write it
  that way?"*, *"I'm still learning"*, or reads the code you show without judging it.
- Signals for **Plain**: asks only about outcomes, never opens files, says *"I don't
  do code"* **and** declines when offered an explanation.
- **Genuinely unclear** → deliver both; the extra paragraph is cheap, a wrong level is
  not. Then ask once, and remember the answer for the rest of the session.

> **🚩 Do not confuse "cannot read code" with "does not want to learn."**
> Sending a willing learner to Plain mode locks them out of ever reaching Expert mode.
> It is the one presentation choice that makes the user *worse off* for having worked
> with you. **When a user cannot read code, always offer the learning option — never
> silently assume they don't care.**

**Why this must be asked at the start, not guessed at the end:** presentation level
changes *what evidence you collect while working*. If the user is an Expert reviewer,
you should be keeping the diff readable, naming your trade-offs as you go, and
noticing where your own implementation is weak — all things easier to do **during**
the work than to reconstruct afterwards. A Learner needs the same discipline, plus
the *reasoning* kept alongside the code as you write it.

> **This is not about flattering the user.** An Expert reviewer is the single best
> chance to catch *suboptimal* code — dead-simple-but-slow, N+1 queries, a needless
> abstraction. A Plain reviewer is the best chance to catch *wrong-intent* code.
> A Learner is the best chance to catch *unexplained* code — the kind that no future
> maintainer will understand either. **All three reviews are valuable. None
> substitutes for another.**

### 0.1c — Ask third: how should the code be *delivered*?

**Review level (0.1b) answers *how deep*. This answers *how it reaches the user*.**
They are separate: an Expert who reads code fluently may still prefer a compact
pointer over a wall of pasted source, and the same Expert working on a 400-line file
may want the changed lines pasted but not the rest.

**Ask the user directly:**

> *"(Only matters if you read code.) When I show changes, do you want the code
> pasted in the chat, or a pointer to the file and line — `lib/filter.py:34-40` —
> so the chat stays short? The file is always the source of truth either way."*

| They choose… | **Delivery mode** | What every unit shows |
|---|---|---|
| Paste it here | **Inline** | The changed code, in the reply |
| Point me at the file | **Pointer** *(default)* | The changed lines **+** `path/to/file.py:34-40` **+** a one-line anchor |
| Not stated, cannot be asked | **Hybrid** — paste the changed region, point at the rest | Both — this is the safe default |

**⭐ The pointer rule — the changed region is ALWAYS pasted:**

```
✅  lib/services/filter.py:34-40   —  the changed lines, pasted into the chat
    ...unchanged lines 12-33 and 41-88 → pointer only, do not paste
```

Pointer mode **never** means *"go read the file yourself."* It means: **the lines
that changed stay in the chat; the lines that did not, become a pointer.** A pointer
with no pasted change is not a compact review — it is a withheld one, and it is the
exact behaviour rule 6 exists to prevent.

**Format for every pointer (all three parts are mandatory):**
1. **Full path from the repo root** — `lib/services/filter.py:34-40`, not `filter.py`
2. **Line number or range** — a pointer without a line number is not a pointer
3. **A one-line anchor** — quote the changed line so the user can confirm they are
   looking at the right thing: *"— line 34 is `if joint in injured_joints:`"*

> **Why a pointer and not a paste, when the code is unchanged:** pasting 200 lines
> where 6 changed drowns the 6. **Signal-to-noise is the point of a review.** A
> compact message that is actually read beats a complete one that is skimmed — and
> the file on disk is never out of date, while chat scrollback always is.

> **Why the pointer must never replace the explanation:** *"done, see
> `filter.py:34`"* is not a compact review, it is an evasion — the same excuse the
> anti-pattern table rejects. **Meaning first, then the changed code, then a pointer
> for the rest.** In that order, always.

**Why this is asked here and not in PHASE 5:** the answer changes decisions made
*long before* git enters the picture — how hard you check the blast radius, whether
you may review your own work, whether you need a plan doc. Discovering "oh, this is
a team repo" at commit time means the previous four phases were run at the wrong
strictness. **Establish it at the start.** PHASE 5.1 re-reads this answer to pick
the versioning mode.

### 0.2 — Then: is the task clear and low-risk?

If **it is not clear what to build**, or **the approach is risky**, do NOT start coding:

- **Spike** — a throwaway experiment to prove an idea can work (e.g. *"does this
  model fit in 6 GB VRAM?"*). The output of a spike is *knowledge*, **not** production
  code. After the spike: **delete the code**, start clean with TDD.
- **Plan first** — write a markdown plan and **execute nothing**. Required for tasks
  >1 hour or >3 files, **and always for team repos**. Save it under `.hermes/plans/`
  (or `docs/plans/`) as `YYYY-MM-DD-slug.md`. Include: goal, assumptions, approach,
  steps, files touched, tests, risks, open questions.
- **Ask the user** — when the *goal itself* is vague. Do not guess and then build the
  wrong thing.

**Signs you need the pre-phase:** you cannot write one sentence of the form
*"done means ..."*; or 2+ approaches look equally reasonable; or there is external
uncertainty (performance, cost, a third-party API).

> **Anti-pattern:** starting work, then asking about repo ownership only when you
> reach the commit step. By then the analysis depth, review requirement, and branch
> decision were all made on an assumption you never checked.

---

## PHASE 0 — Load Context (mandatory, in order)

1. **Scan for relevant skills.** In Hermes: `skills_list()` then `skill_view()`.
   Everywhere else: list your rules/skills directory. Load what matches the task —
   not only when you are already writing tests.

   Worth loading if you have them: a testing-strategy skill, a QA/exploratory-testing
   skill, a TDD skill, a systematic-debugging skill. **If you do not have them, this
   file already contains their essence** (see PHASE 1, PHASE 3 step 8, PHASE 4 step 12).

2. **Coding journal** — search `.hermes/coding_journal.md` (or `NOTES.md`) for a
   similar past problem. Found one → adapt the known solution first.
   **No file yet → create it now.** This is the single cheapest defense against
   re-solving the same bug twice. Format is in Appendix A.

3. **Blast radius** — before touching code, find out who calls it.
   **Priority depends on the PRA-PHASE 0.1 answer:** in a **team/public repo this is
   mandatory and thorough** (other people's code depends on yours); in a personal
   repo it is a quick sanity check.
   Hermes: `codegraph explore <feature>` / `callers` / `impact`.
   Elsewhere: IDE "find references", or `grep -rn "function_name" .`.
   Project not indexed and no grep target → write *"blast radius: manual review"*
   and move on. Do not silently skip this.

---

## PHASE 1 — Analysis

4. Break the task down: hypothesis, architecture, execution steps.

   **For bugs — this is non-negotiable:**

   > ### The Iron Law
   > **NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST.**
   >
   > Symptom fixes are failure. A patch that makes the error disappear without you
   > being able to say *why it appeared* is not a fix — it is a delay.

   **Write down, in one sentence, *why this is happening*.** If you cannot, you are
   not allowed to fix it yet. Go gather evidence.

   **Build a tight feedback loop** — this *is* the debugging work:
   - One command that goes **red** on the user's exact symptom, and **green** when fixed.
   - Fast, deterministic, runnable by you without a human.
   - Asserts *this* symptom — not merely "doesn't crash".

   Ways to build one, roughly in order of preference:
   1. A failing test at the seam that reaches the bug (unit / integration / e2e).
   2. A `curl`/HTTP script against a running dev server.
   3. A CLI invocation with fixture input, diffing stdout/stderr.
   4. A headless browser script (Playwright/Puppeteer) asserting on DOM/console/network.
   5. Replaying a captured trace (HAR, payload, event log, webhook body).
   6. A throwaway harness that boots the smallest useful slice.
   7. A property/fuzz loop, when the bug is intermittent wrong output.
   8. A bisection harness (`git bisect run`) when it broke between two known states.
   9. A differential loop: old vs new, two configs, two providers.
   10. A human-in-the-loop script — **last resort**; script the human's steps.

   **Then tighten it:** faster (cache setup, narrow scope), sharper signal (assert the
   exact symptom), more deterministic (pin time, seed randomness, freeze network).
   For flaky bugs your immediate goal is a **higher reproduction rate**, not
   perfection. 50% flake = debuggable. 1% flake = usually not.

   **Gather evidence across component boundaries.** When the system has layers
   (UI → API → service → DB, CI → build → deploy), instrument the boundary: log what
   enters, log what exits, verify config propagation. Run once to see *where* it
   breaks — then investigate that component only.

   **Trace data flow upstream.** Where did the bad value originate? Who called this
   function with it? Keep going up until you find the source. **Fix at the source.**

   **Check recent changes:** `git log --oneline -10`, `git diff`.

   **Phase 1 done when:** you can state the root cause as a testable sentence, and
   your loop is red-capable. Otherwise — do not proceed.

---

## PHASE 2 — Reference (only if an external library/API is involved)

5. Look up the **current** API. Never write an API signature from memory — that is
   how you get hallucinated parameters and phantom methods.
   - Hermes: Context7 MCP (`resolve-library-id` + `query-docs`).
   - Elsewhere: fetch the official docs, or read the installed package source.

   **Read the reference implementation COMPLETELY.** Skimming a pattern and "adapting"
   it from memory guarantees bugs. Every line, or don't use it.

---

## PHASE 3 — Execution

6. Write code that is **EFFECTIVE and EFFICIENT** — in that priority. **NOT** "as short
   as possible".
   - **Effective** = actually solves the problem. Ask: *"if I delete this feature,
     does the problem come back?"* If no → the code is not effective; it is decoration.
   - **Efficient** = cheap in resources (CPU, memory, I/O, network calls, API quota,
     money). Avoid repeated work in loops, N+1 API calls, recomputing what you could cache.
   - **Long code is ALLOWED** if that is the most effective/efficient/safe option.
     Short code is **not** a goal. (stdlib-first and managed diffs are still good —
     but they **lose** to effective + efficient.)
   - **Limit:** step 7 below may **never** be cut for the sake of "efficiency".

7. **Non-negotiable per change — never cut these:**
   - **Input validation** — trust nothing from outside (user input, files, network,
     another service).
   - **Anti data-loss** — never overwrite/destroy data without a recoverable path.
     Destructive operations get a dry-run or a backup.
   - **Safety/security** — no secrets in code; no injection; fail **closed** on
     unknown/ambiguous input (deny, don't allow).
   - **Error handling** — failures must be visible and diagnosable, never swallowed.

8. ### ⭐ THE PER-FUNCTION CYCLE — one function, one full cycle

   This is the highest-leverage rule in this document.

   ```
   [1] WRITE → ONE function only. Not five. Not "the module".
   [2] WORKS → it runs, no error.
   [3] TEST  → a test for THAT function. It must be GREEN.
   [4] AUDIT → check the blast radius on other functions.
   [5] LIVE  → only now use it. Touches money/credentials → DRY-RUN first.
   [6] EXPLAIN → tell the user, in plain language, how this unit works.
                ⭐ MANDATORY. See 8.1 below — the user must be able to review it.
            ↓
            back to [1] for the next function.
   ```

   **Use TDD where it fits:** write the failing test first (RED), make it pass
   (GREEN), then clean up (REFACTOR). Test FIRST proves the test can actually fail —
   a test written after the code has never been shown to catch anything.

   **Why:** a bug in function A that only surfaces while you are building function D
   is *many times* more expensive to fix, and far harder to attribute. You will have
   forgotten your own assumptions from two hours ago.

   **Real failure this rule prevents:** a knee-injury filter that silently discarded
   `glute_bridge` — a movement that *should* have been the knee-safe substitute. The
   contradiction was invisible while writing the filter, and only surfaced in a
   later exploratory pass. Under the per-function cycle, the substitute list would
   have been tested the moment it was written.

   ### 8.1 — ⭐ STEP 6 IS MANDATORY: explain every unit back to the user

   **After finishing ONE unit / feature / task, you MUST explain how it works — in
   plain language, to the user.** Not a file list. Not a diff. A *explanation a
   non-author can follow.*

   **Why this is not optional:**

   > Every other verification in this workflow is done by a machine or by you.
   > **The user is the only reviewer who knows what the code is actually supposed
   > to accomplish.** If you never explain it, the user cannot catch a
   > misunderstanding — they can only discover it later, in production.

   A user who reads *"done, 12 tests passing"* has learned almost nothing. A user who
   reads *"when a knee-injured user asks for leg exercises, we now pick substitutes by
   the joint loaded, not the muscle trained — so the list is no longer empty"* can
   immediately say *"wait, that's not what I meant."*

   **What a unit explanation must contain (keep it short):**

   | # | Element | Example |
   |---|---|---|
   | 1 | **What it does** — in one sentence, no jargon | *"Picks safe exercises for a user's injury."* |
   | 2 | **How it works** — the mechanism or algorithm, step by step | *"Filters by joint loaded → adds substitutes → removes the ones that still load the injured joint."* |
   | 3 | **Why this way** — the decision and the trade-off | *"By joint, not by muscle — because a knee-safe movement can still be trained by a leg muscle."* |
   | 4 | **How it was verified** — honestly | *"7 unit tests + ran it on a real knee-injury profile, x/y passing."* |
   | 5 | **What to check yourself** — point at the risky part | *"Try a knee injury + no equipment; the leg list should have 2-3 items, not 0."* |

   **Rules for the explanation:**
   - **Plain language.** The user may not be a programmer. No unexplained acronyms.
   - **Say the trade-off, not just the win.** If you chose speed over memory, say so.
   - **State the honest test result** — `x/y passing`, red is red. (See ATURAN.)
   - **Give the user one concrete thing to try.** A review they cannot perform is not
     a review. Name the input and the expected output.
   - **Never let it become "I wrote 3 files."** That is a status report, not an
     explanation, and it transfers no understanding.

   **Scale it by the PRA-PHASE 0.1 answer:**
   - **Personal repo** → short version is enough: what / how / how verified / what to try.
   - **Team repo** → same, plus *why this way* and the trade-off, because someone else
     will maintain it. Team repos also get the full end-of-task report (below).

   **End of task — the full report.** When the whole task is done (not each unit),
   give the four-line gate report from PHASE 4.12 **plus** a walkthrough of the
   finished feature: what the user can now do that they could not before, and the
   files that changed. One unit explanation per unit; one full walkthrough per task.

   > **Anti-pattern:** reporting *"done"* with a bare list of changed files and
   > nothing else. The user can read `git diff` themselves — **what they cannot
   > get from the diff is your intent and your reasoning.** Always give the meaning
   > first, then the code if they want it (see 8.1b).
   >
   > **But do not hide the code either.** Hiding it from a user who can read it
   > removes your best technical reviewer. The rule is *"meaning first, then code"* —
   > not *"meaning instead of code"*.

   ### 8.1b — ⭐ THE EXPERT REVIEW LAYER (when the user can read code)

   **Plain explanation and code review catch different classes of mistake.** A
   plain-language summary can catch *"that is not what I meant."* It can **never**
   catch:

   - an N+1 query inside a loop
   - a function that is correct but does 10,000× more work than needed
   - a needless abstraction that will be painful in six months
   - a silent `except: pass` swallowing real errors
   - a copied 40-line block that should have been one call

   **An expert owner is the only reviewer who can catch those.** Give them what they
   need.

   **When the PHASE 0.1b answer is "Expert", every unit explanation ALSO includes:**

   | # | Element | Why it exists |
   |---|---|---|
   | 1 | **The code itself** — the diff or the changed function, complete, not paraphrased | They cannot judge what they cannot see |
   | 2 | **The key decision in technical terms** | e.g. *"one query per exercise, batched — not N+1"* · *"O(n log n) sort, acceptable for n<500"* |
   | 3 | **The honest trade-off** — what you gave up | e.g. *"readable over fast — this loop is fine at current size but will need caching past ~10k"* |
   | 4 | **Where you are least confident** | e.g. *"the error path on line 42 is untested; I could not reproduce a malformed key"* |
   | 5 | **What you want them to judge** | e.g. *"is this abstraction worth it, or should it be inlined?"* |

   **How to deliver it — depends on PHASE 0.1c:**
   - **Inline mode:** paste the changed code in the reply. Simple, and the reviewer
     is not switching windows.
   - **Pointer mode (default):** **paste the changed lines**, then point at the rest —
     `lib/services/filter.py:34-40` + a one-line anchor. Never paste unchanged
     context; never make the pointer a substitute for the pasted change.
   - **Hybrid:** paste the changed region, point at everything else.

   > **Rule that does not bend:** the lines that **changed** are always visible in
   > the chat. Only the lines that did **not** change may be reduced to a pointer.
   > A pointer with no pasted diff is an evasion dressed as efficiency.

   **Rules for the expert layer:**
   - **Never dump the whole file** unless the task *is* the file. Show the changed
     region plus enough surrounding context to judge it — and when the file is long,
     let the surrounding context be a **pointer**, not a paste.
   - **Point at anything you are unsure about yourself.** Volunteering your own
     weak spot costs nothing and is exactly what a good reviewer needs.
   - **Do not defend the code.** Present it for judgement, not for approval. If the
     user says *"this is inefficient"*, that is **the layer working** — not an attack
     to be argued down.
   - **Do not show a trimmed/pretty version.** If the real diff is messy, show the
     real diff. A sanitised excerpt defeats the purpose. *(Pointer mode is not a
     licence to hide the ugly parts — point at them and say they are ugly.)*
   - **Never hide a decision you made on the user's behalf.** If you picked a library,
     a data structure, or a pattern without asking, say so explicitly.

   > **Both layers, always for Expert:** the plain walkthrough **first** (so the
   > intent is agreed), the code **second** (so the craft is judged). Code first
   > invites line-level nitpicks before anyone has agreed on what the thing is for.

   ### 8.1c — ⭐ THE LEARNING LAYER (user cannot read code yet, but wants to)

   **This mode exists for one reason: a user who cannot read code today should not
   still be unable to read it a year from now, having worked with you the whole time.**

   A Learning user **cannot yet catch your technical mistakes** — so this layer does
   **not** replace the Expert layer as a safety net. It does something different and
   complementary: it turns every unit into a small lesson, so the reviews get *stronger
   over time*. In a long-running project, this is the highest-return presentation mode
   there is.

   **When the PHASE 0.1b answer is "Learning", every unit explanation ALSO includes:**

   | # | Element | What it looks like |
   |---|---|---|
   | 1 | **The concept first** — the idea before the syntax | *"This is a 'filter' — a rule that keeps only the items that pass a test. You'll see this everywhere."* |
   | 2 | **Annotated code** — the real code, but with the *why* per block | Comments/blocks explained: what each part does and why it is there |
   | 3 | **One term, defined** — not five | Pick the single most important new word and define it plainly. Ignore the rest for now. |
   | 4 | **Why the code is shaped this way** — in plain words | *"The loop is outside because we only want to open the file once — opening it inside is slower."* |
   | 5 | **What to notice** — one thing to look for | *"Notice line 3 checks for empty first. That guard is why we never crash on a missing value."* |
   | 6 | **A place to read more** — only if they ask, or once per task | Docs link, or a one-line search term. **Do not turn this into homework.** |
   | 7 | **An open invitation, every single time** | *"Ask about any line you want — that is what this mode is for."* |

   **Rules for the learning layer:**
   - **Annotate, do not obfuscate.** Show the *real, complete* code. Never simplify
     the code for teaching and then run the simplified version — the learner will
     misunderstand what is actually deployed.
   - **Teach the pattern, not the trivia.** *"This is a guard clause"* is reusable.
     *"This variable is named `tmp2`"* is noise.
   - **One new concept per unit.** Two taught well beats six taught badly. If a unit
     genuinely introduces several concepts, teach the core one and **name** the
     others as "you will meet these soon".
   - **Never make the user feel behind.** No *"as you should already know"*, no
     *"obviously"*, no sighing at a basic question. A question asked is the mode
     working exactly as intended.
   - **Do not hide the full version.** Show the whole function even if you only
     annotate part of it, so they build a sense of real-world shape. They can read
     the rest when ready — and **never assume they are not ready.**
   - **Pointer mode is allowed here, with one exception.** You may point at
     `path/file.py:34-40` for the parts you are *not* annotating — but the block you
     are **teaching** is always pasted in full. A learner cannot learn from a
     reference number; they need the code in front of them. **Annotating a pointer
     teaches nothing.**
   - **Keep the project moving.** Teaching must not stall the work. One focused
     explanation per unit; deep dives only when the user asks for one.
   - **Promote without being asked.** If the user has started questioning your
     technical choices, offer the Expert layer: *"you are reading this well — want me
     to start showing the code the way I would for a developer?"*

   > **A Learning user is still a reviewer.** Their *"I don't understand this part"*
   > is genuine signal — it often points at code that is confusing for everyone, not
   > just for them. Take it seriously; do not wave it away as inexperience.

---

## PHASE 4 — Verification

9. Linter (`dart analyze`, `ruff`, `eslint`, …) — **0 issues**.

10. Test suite — run it. **Report the honest number.** Include the exact command and
    its real output.

11. E2E manual when UI/flow changed (Playwright for web; click the real thing).

12. ### 🔒 THE IMPACT-AUDIT GATE — DO NOT PASS THIS

    Before commit. Two checks, both mandatory:

    **(a) Blast-radius check**
    - Who **calls** the functions I changed?
    - Did the **behaviour or return shape** change? Do old callers still hold?
    - Is there **shared state** (globals, singletons, config, DB rows, caches)?
    - Does any feature I did **not** touch still work?

    **(b) Test + audit — using the embedded discipline below**

    #### Testing strategy — pick the MINIMUM sufficient kinds

    Do not test everything. Classify the change, then choose:

    | Change | Minimum sufficient tests |
    |---|---|
    | Pure function / utility | **Unit** — inputs → outputs, boundaries (0, empty, max, negative) |
    | Data transformation / parser | **Unit** + **Property** (round-trip: encode→decode returns the original) |
    | API endpoint / handler | **Integration** — real request through the stack, incl. an invalid one |
    | Module boundary / contract | **Contract** — provider and consumer agree on the shape |
    | User flow / UI | **E2E** — one happy path + one failure path |
    | Bug fix | **Regression** — the test MUST be red before the fix, green after |
    | Anything | **Regression** for behaviour that must not change |

    **If the test is for a bug fix:** it must **fail first**, and it must fail **for the
    right reason**. A test that fails because of a typo, not because of the bug, proves
    nothing. Run it before the fix. Read the failure message. Then fix.

    #### Exploratory QA (dogfooding) — finds what unit tests cannot

    Unit tests were written by the same mind that wrote the code, so they *share its
    blind spots*. A test author who misunderstood the requirement writes a test that
    happily confirms the misunderstanding. **Exploratory testing is how you escape that.**

    Run the system the way a real user would:

    1. **Pick a realistic scenario, not a synthetic one.** Not "call function with
       input X" — *"a 62-year-old with a bad back, Monday morning, on a phone."*
    2. **Follow the whole path end-to-end.** Do not jump to the part you changed.
    3. **Push on edges:** empty state, max values, wrong type, rapid repeat, cancelled
       mid-flow, no permission, no network.
    4. **Watch for the silent failure.** The worst bugs are not crashes — they are
       *empty results, wrong-but-plausible values, and quietly skipped steps*.
       **An empty list where a list was expected is a bug, not a page.**
    5. **Capture evidence as you go:** what you did, what you expected, what happened,
       and the raw output/path. Evidence turns "I think" into "here it is".
    6. **Write a report**, not a shrug: per issue — steps to reproduce, expected,
       actual, evidence, severity (Critical/High/Medium/Low), category
       (Functional/Visual/Accessibility/Console/UX/Content).

    **In a browser context specifically:** check the **console** after every navigation
    and every significant interaction — silent JS errors are among the highest-value
    findings. Take an annotated screenshot to reason about element positions. Test
    invalid input as well as valid. Scroll to the bottom; below-the-fold rendering
    breaks a lot.

    #### Independent review — never sign your own work

    > **Core principle: no agent verifies its own work. Fresh eyes find what you miss.**

    - **Security scan** the diff: hardcoded secrets, injection (`eval`, `exec`,
      `shell=True`, string-formatted SQL), unsafe deserialisation (`pickle`).
    - **Baseline-aware quality gate:** capture failures *before* your change
      (stash → run → pop). Only **new** failures block. Pre-existing failures are
      recorded, not blamed on you.
    - Where a subagent/reviewer is available, hand it the diff and ask it to
      *break* the change — not to agree with you.

    #### Honest reporting — non-negotiable

    - Tests red → **say red.** Tests not run → **say not run.** Something skipped →
    **say skipped, and why.**
    - "It should work" is not a test result.
    - Report the numbers: `x/y passing`.
    - A confident false success is worse than an honest failure.

    **Report exactly four things:**
    ```
    what changed · who is affected · test result (x/y) · residual risk
    ```

    **(c) Cleanup** — remove duplication, flatten needless complexity, cut waste,
    deepen band-aids. This is a **cleanup pass, not a bug hunt**: do not change
    behaviour. Do not silently "improve" logic while tidying — that is a new change
    and needs its own cycle.

    **If (a), (b), or (c) is not done → FORBIDDEN to enter PHASE 5.**

---

## PHASE 5 — Version Control & Deploy (LOCKED until the Impact-Audit Gate passes)

> **Why versioning matters here:** the Impact-Audit Gate reduces bad changes.
> Version control makes them **recoverable**. A change you cannot undo is not a
> change — it is a gamble. This phase is what makes "I broke it" a 30-second
> problem instead of an afternoon.

**Precondition:** PHASE 4 step 12 passed. Committing an unverified change is
shipping a guess.

### 5.1 — Pick your mode (from the answer you already got in PRA-PHASE 0.1)

**You already asked this.** PRA-PHASE 0.1 establishes whether the repo is personal,
team, or unknown. **Use that same answer here — do not ask twice, and do not
contradict it.** If you skipped 0.1, go back and ask now.

With the answer in hand, pick the ceremony level:

| | **Solo / throwaway** | **Solo / real project** | **Team / public repo** |
|---|---|---|---|
| Branch | commit to `main` OK | branch if >1 commit | **always branch** |
| Commit size | loose | **atomic** | **atomic, reviewed** |
| SemVer | optional | **yes** | **yes** |
| Tag releases | no | yes | **yes + release notes** |
| CHANGELOG | no | short | **full, maintained** |
| Rewrite history | free | careful | **never on shared branches** |

**How to tell which you are in:**
- Will anyone else read this repo, now or later? → **Team / public**
- Will *past-you* need to understand this in 6 months? → **at least Solo / real project**
- Is this a spike, a scratch file, or a one-off script? → **Solo / throwaway**

**When unsure, pick the stricter mode.** Downgrading later is easy; recovering
from a botched shared history is not.

> **Consistency check:** your answer here must match PRA-PHASE 0.1. If you told the
> user you were working in a personal repo, you may commit to `main`. If it is a team
> repo, you may not — no exceptions for "it's a small change".

### 5.2 — Branch (unless solo/throwaway)

```bash
git checkout -b feat/short-name      # or fix/, chore/, docs/
```

Name it after the *intent*, not the file: `fix/knee-filter-empty-list`, not
`fix/plan-engine-2`.

The branch is your **way home**. On a branch, a bad experiment costs one command:
`git checkout main`. On `main`, it costs an archaeology session.

> **Never commit directly to `main` on a shared or public repo.** If you already
> did, stop and fix it before pushing — see 5.6.

### 5.3 — Commits: one logical change per commit (atomic)

**The test:** can this commit be described in one `type(scope): subject` line
with no "and"? If it needs an "and", split it.

```bash
git add <specific files>          # NOT `git add .` — that is how secrets and
git commit -m "type(scope): subject"   # junk get committed
```

**Conventional commit types:**

| Type | For |
|---|---|
| `feat` | new behaviour for the user |
| `fix` | a bug fix |
| `refactor` | same behaviour, better structure |
| `test` | tests only |
| `docs` | documentation only |
| `chore` | build, deps, config, tooling |
| `perf` | performance improvement |

**Why atomic matters — this is the whole point:** `git revert <sha>` undoes exactly
one logical change. If one commit contains five changes, reverting the broken one
also throws away the four that worked. **Atomic commits are what make rollback
surgical.**

**Before every commit:**
```bash
git status           # what am I actually about to stage?
git diff --staged    # read the diff — really read it
```
- No secrets, keys, `.env`, credentials, large binaries, or generated files.
- If you spot something unrelated, do not "fix it while I'm here" — it is a
  separate change with its own cycle.

### 5.4 — Semantic Versioning (SemVer)

Format: **`MAJOR.MINOR.PATCH`** — e.g. `2.4.1`.

| Bump | When | Example |
|---|---|---|
| **MAJOR** | breaking change — existing callers must change | `1.9.0 → 2.0.0` |
| **MINOR** | new feature, backward-compatible | `2.3.1 → 2.4.0` |
| **PATCH** | bug fix, backward-compatible | `2.4.0 → 2.4.1` |

Below `1.0.0`, anything may break at any time — that is what `0.x` signals.

**Non-negotiables:**
- One source of truth for the version (a manifest file, or a git tag). **Not two
  numbers that can disagree.**
- A released version is **immutable**. Wrong? Bump again. Never re-point a version
  someone may already depend on.
- Bump the version **in the same commit as the release**, not "later".

### 5.5 — Tag + CHANGELOG (release only)

```bash
git tag -a v2.4.0 -m "Release 2.4.0"
git push origin v2.4.0
```

**Annotated tags (`-a`), not lightweight.** A tag with a message carries author,
date, and intent — it is a real release record.

**CHANGELOG.md** — newest first, written for a *human deciding whether to upgrade*:

```markdown
# Changelog

## [2.4.0] - 2026-10-02
### Added
- Nutrition profiles for 7 countries
### Fixed
- Knee-injury filter no longer discards valid substitutes
### Changed
- Senior profiles reduce intensity instead of tier

## [2.3.1] - 2026-09-28
### Fixed
- Meal totals off by one serving
```

Keep-a-Changelog sections: `Added` / `Changed` / `Deprecated` / `Removed` /
`Fixed` / `Security`. **Describe the effect on the user, not the source file.**
"Fixed knee filter discarding substitutes" — not "changed line 42".

**Skip the CHANGELOG** in Solo/throwaway mode. Add it the moment someone else
(or future-you) depends on the code.

### 5.6 — Rollback: know the way home *before* you need it

Pick the smallest tool that fixes the problem:

| Situation | Command | Notes |
|---|---|---|
| Not committed yet | `git restore <file>` / `git checkout -- <file>` | safest — nothing is public |
| Bad commit, **not pushed** | `git reset --soft HEAD~1` | undoes commit, **keeps** your edits |
| Bad commit, **pushed to shared branch** | `git revert <sha>` | makes a *new* commit that undoes it |
| Whole release is bad | `git revert <range>` or deploy previous tag | e.g. `git checkout v2.3.1` |
| Need to look around safely | `git stash` / `git worktree add ../tmp` | does not disturb the working tree |
| Finding which commit broke it | `git bisect start` … `git bisect run <test>` | binary search over history |

**The rule that prevents most pain:**

> **NEVER `git reset` / `git rebase` / `git push --force` on a branch others may
> have pulled.** Rewriting shared history breaks every copy of it. If it is
> already pushed and shared → **`git revert`**. Revert is additive and safe.

`git commit --amend` and `git reset` are fine **before** push, on your **own
branch**. After push, on a shared branch, they are the start of an incident.

### 5.7 — Coding journal

Append an entry: task, approach, problems, solution, files, tests, lesson.
Format in Appendix A. **Overwrite nothing** — append only.

The journal complements git: git records *what* changed, the journal records
*why you chose it* and *what bit you*. Git blame does not know why you rejected
the alternative.

### 5.8 — Deploy + verify

After deploying, check the real thing — health endpoint, live URL, smoke test.
**"Pushed" is not "working", and "deployed" is not "correct".**

If the deploy is bad, roll back using 5.6 *first*, diagnose *second*. A
known-bad version running in production is not a learning opportunity — it is an
outage.

---

## ATURAN / RULES

- **Skipping a phase is allowed only when it genuinely does not apply** (e.g. no
  external library → skip PHASE 2). **Say out loud why you skipped it.**
  Silent skipping is the failure mode this whole protocol exists to prevent.
- **Effective & efficient > minimum.** Do not be proud that the code is short; be proud
  that it hits the target, saves resources, and is safe.
- **The per-function cycle (PHASE 3 step 8) and the Impact-Audit Gate (PHASE 4 step 12)
  may NEVER be skipped** — not for a deadline, not for a "one-liner", not because the
  change "obviously can't break anything".
- **Never verify your own work.** Get a second pass — a subagent, a linter, a test,
  or at minimum a fresh read of the diff.
- **Root cause before fix.** "Quick fix now, investigate later" is how you get the
  same bug twice.
- **Honest reports only.** Red is red. Skipped is skipped. Unknown is unknown.
- **Every change must be reversible.** Branch, atomic commits, and a known rollback
  path are part of "done" — not optional polish. If you cannot undo it, you do not
  control it. See PHASE 5.
- **Never rewrite shared history.** Once pushed to a shared branch, fix forward with
  `git revert`. `reset` / `rebase` / `push --force` on shared branches is how one
  mistake becomes everyone's problem.
- **Establish repo ownership before writing code** (PRA-PHASE 0.1) — personal or
  team? It sets the strictness of *every* later phase: how hard you check blast
  radius, whether you may review your own work, whether you branch. Asking this at
  commit time means the whole workflow ran at a guessed strictness.
  **When unknown, treat it as a team repo.**
- **Explain every finished unit back to the user** (PHASE 3.8.1). Not a bare file
  list — the *meaning*: what it does, how, why this way, how it was verified, and one
  thing the user can try. **The user is the only reviewer who knows what the code was
  supposed to accomplish**; if you never explain it, you remove their only chance to
  catch a misunderstanding early. A task is not "done" until the user can review it.
- **Match the review to the reviewer** (PRA-PHASE 0.1b). Ask **two** things — *can*
  they judge code, and do they *want* to understand it. If the user can read code,
  **show them the code** — the diff, the technical decision, the trade-off, and where
  you are least confident (PHASE 3.8.1b). **Never hide the code from someone able to
  judge it; never dump code on someone who cannot.**
- **Never lock a willing learner out of code.** If the user cannot read code yet
  **but wants to**, that is **Learning** mode (PHASE 3.8.1c) — annotated real code,
  one concept at a time, and an open invitation to ask. Treating a willing learner as
  a plain reader is the one presentation choice that leaves the user *worse off* for
  having worked with you. **"Cannot read code" and "does not want to learn" are not
  the same thing.**
- **Deliver the code the way the user asked** (PRA-PHASE 0.1c). Inline, or a pointer
  like `lib/services/filter.py:34-40`. **The lines that CHANGED are always pasted;
  only the unchanged context becomes a pointer.** A pointer with nothing pasted is not
  a compact review — it is a withheld one. **Meaning first, then the changed code,
  then a pointer for the rest.**

### If you are tempted to skip — read this table

| Excuse | Reality |
|---|---|
| "Issue is simple, no need for process" | Simple issues have root causes too. The process is fast for simple bugs. |
| "Emergency — no time" | Systematic is **faster** than guess-and-check thrashing. |
| "Just try this first, then investigate" | The first fix sets the pattern. Do it right from the start. |
| "I'll write the test after confirming it works" | Untested fixes don't stick. Test first proves it. |
| "Multiple fixes at once saves time" | You can't tell which worked, and you add new bugs. |
| "The reference is long, I'll adapt from memory" | Partial understanding guarantees bugs. |
| "I can see the problem, let me fix it" | Seeing a symptom ≠ understanding the cause. |
| "One more fix attempt" (after 2+ failures) | **3+ failures = architectural problem.** Question the pattern; don't patch again. |
| "An empty result is fine" | An empty result where data was expected is usually **the bug**. |
| "It's just a small change, I don't need to ask about the repo" | Small changes to a shared repo still break other people. **Ask at the start** (PRA-PHASE 0.1) — it is one question, and it sets the strictness for everything after. |
| "I'll figure out if it's team or personal when I commit" | By then your blast-radius depth, review choice, and branching were already decided on a guess. |
| "Done — here are the files I changed" | That is a status report, not an explanation. The user can read the diff; they need the **meaning** (PHASE 3.8.1). |
| "The user will read the code if they want to know" | They asked you to build it *because* they are not the one reading the code. Unexplained work is unreviewable work. |
| "The tests pass, that's the verification" | Tests verify *your* understanding. Only the user can verify the understanding is the *right one*. |
| "The user can read `git diff` if they want the code" | True — but only if you **show them which part matters** and say what you are unsure about. A diff dumped without framing is not a review. |
| "The explanation is enough, no need to show code" | A plain summary cannot reveal an N+1 query, a needless abstraction, or a swallowed exception. If the user can read code, **withholding it costs you your best reviewer**. |
| "Showing the code will invite nitpicks" | That is the layer **working**. A technical objection from the owner is cheaper now than in production. Do not argue it down. |
| "They can't read code, so keep it plain" | Only if they **also** don't want to learn. If they want to learn, plain mode locks them out forever — use **Learning** mode (3.8.1c). This is the one mode choice that makes the user worse off. |
| "Explaining the concept will slow the project down" | It costs one paragraph per unit. The alternative is a user who can never review your work at any depth, for the whole project. |
| "They'll ask if they want to understand it" | Beginners often don't know what to ask — they don't know what exists to ask about. **Offer the option; don't wait for the request.** |
| "I'll simplify the code so it's easier to read" | Then they learn the wrong code. **Annotate the real thing** — never show a simplified version of what you actually deployed. |
| "A pointer is shorter, I'll just give the file and line" | Only for the lines that **did not change**. The changed lines get pasted regardless — otherwise you have not made the review shorter, you have removed it. |
| "The chat is getting long, I'll stop pasting the code" | Then paste **less context**, not **less change**. Changed lines stay; surrounding context becomes a pointer. |
| "They have the repo, they can open the file" | True for unchanged context. For the change itself, **making them hunt for it is the evasion the pointer is meant to avoid.** |
| "Pasting the code burns tokens" | Unchanged lines do. Point at those — `path/file.py:34-40`. The diff itself is the one thing worth the tokens. |

### The Rule of Three (when a fix does not work)

1. **STOP.** Count your fix attempts.
2. If **< 3**: return to PHASE 1 and re-analyse with the new information.
3. If **≥ 3**: **stop fixing and question the architecture.**
   Signals: each fix reveals new coupling somewhere else; each fix needs "massive
   refactoring"; each fix creates a fresh symptom elsewhere.
   **This is not a failed hypothesis — this is the wrong architecture.** Discuss with
   the user before attempting fix #4.

---

## Quick Reference

| Phase | Do | Done when |
|---|---|---|
| **PRA** | **Ask repo ownership + review level + delivery mode (inline/pointer)** · spike / plan / ask | You know who the code is for, how deeply they review, and how they want it delivered |
| **0. Context** | Skills, journal, blast radius | You know what you're touching and who else touches it |
| **1. Analysis** | Root cause, tight loop, evidence, trace | You can state the cause in one testable sentence |
| **2. Reference** | Read current docs completely | No API written from memory |
| **3. Execution** | Effective code + per-function cycle | Every function tested, explained, **and code-shown if the user reads code** |
| **4. Verification** | Strategy + exploratory QA + review + honest report | Gate (a)(b)(c) passed |
| **5. Version Control & Deploy** | Mode, branch, atomic commits, SemVer, tag, rollback, verify live | It works in the real environment **and can be undone** |

**Nine rules that carry most of the weight:**
1. Root cause before fix.
2. One function, one full cycle.
3. Never verify your own work.
4. Explore it like a real user, not like its author.
5. Report honestly — red is red.
6. **Explain it back to the user — in plain language — so they can review it.**
7. **Show the code when the user can read it** — plain summary alone misses the
   technical mistakes only an expert reviewer can catch.
8. **Never lock a willing learner out of code** — a beginner who wants to understand
   gets annotated real code, one concept at a time. Not plain mode forever.
9. **Deliver code the way the user asked** — inline, or `file.py:34-40`. Changed
   lines always pasted; only unchanged context becomes a pointer.

**And one that makes the rest survivable:** every change reversible.

---

## Appendix A — Coding Journal format

Locations: one per project, e.g. `.hermes/coding_journal.md`, or `NOTES.md`.
**Search before coding. Append after coding.** Never overwrite the file.

```markdown
---

## [YYYY-MM-DD] <short-title>

**Task**: <what the user asked for>

**Approach**: <why this approach, not the alternatives>

**Problems**:
1. <problem found — the real error message, crash, mismatch>
2. <second problem if any>

**Solution**: <exactly what changed — file, function, key line>

**Files**: <files changed>

**Tests**: <result — pass/fail, how many>

**Lesson**: <one or two sentences for future you>
```

---

## Appendix B — Why this file has no dependencies

Earlier versions of this skill delegated its testing, debugging, and QA discipline to
sibling skills by name. That works inside one specific agent runtime and **breaks
everywhere else** — a reader on another tool hits *"load `dogfood`"* and has no idea
what that is.

So the substance was **inlined**: root-cause discipline (§PHASE 1), per-function cycle
(§PHASE 3.8), testing strategy + exploratory QA + independent review (§PHASE 4.12),
and the journal format (Appendix A).

Nothing is lost. If you have richer dedicated tools — a real browser-QA harness, a
full debugger, a subagent fleet — **use them**; this file is the floor, not the ceiling.

## License

MIT — see `LICENSE`. Use it, fork it, ship it.
