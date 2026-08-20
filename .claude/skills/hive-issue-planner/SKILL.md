---
name: hive-issue-planner
description: "Thao's hive workflow — Issue Planner: break a plan or spec into tracer-bullet tickets with blocking edges, plus Verification DOD per ticket, review-phase labels (review batching for the gate on the unit they merge into — NOT an ordering), HITL Gate field, and numeric-only numbering — the breakdown /hive-master reads verbatim."
disable-model-invocation: true
---

# Hive Issue Planner

Break a plan, spec, or conversation into **tickets** — tracer-bullet vertical slices, each declaring its **blocking** tickets, each tagged with a **review phase**.

Tracker + label conventions come from `PROJECT-PROFILE.md` (Tracker entry); no profile → run `/hive-profile-builder` first.

Quizzing the user (step 4): voice + format = **`/hive-report`**. (Ticket bodies stay technical — agents read those.)

## Process

### 1. Gather context

Source of truth: the **spec** (`spec.md`, or the tracker issue from `/hive-spec-writer`) + the **ADRs / decision log** (from `/hive-architect`). Read both in full, plus conversation context.

- Standalone, or user passes a reference (spec path, issue number, URL) → fetch it, read full body + comments.
- **Bug ticket** → also read `docs/agents/escaped-bugs.md` if present.
    - *Its class + root-cause + "signal" rows source the same-class neighbor checks below. That list — not a mid-run `decisions.md` — is the durable cross-run record of known defect classes.*

### 2. Explore the codebase

Required whenever a later step depends on repository state, not just the spec text — a purely conceptual plan with no repository-dependent claims can skip it, but say so at the quiz (step 4). Required when:
- the spec claims a failure axis "not applicable" (step 5 verifies this by exploration — it can't if this step skipped)
- a bug ticket's neighbor-check needs checking against actual neighboring surfaces
- ticket locality, blast radius, or prefactoring opportunities depend on how the code is actually laid out

- Explore the current code state.
- Ticket titles + descriptions use the project's domain glossary vocabulary; respect ADRs in the area touched.
- Look for prefactoring that makes implementation easier. *"Make the change easy, then make the easy change."*

### 3. Draft vertical slices

Break the work into **tracer bullet** tickets. Each ticket contains:

- **ID** (numbering rule, below)
- **Phase**
- **Title**
- **Blocked by**
- **Gate:** `autonomous` or `HITL`
- **Tier:** `default` (may be omitted) or `strict` (§3b)
- **What it delivers**
- **Verification DOD**
- **Covered invariants:** required when the spec declares any — the ids the quiz's coverage map (step 4) shows against this ticket
- **Tracker keys:** added after publication (step 6)

The subsections below define each field's content rule; this is the field list itself.

#### 3a. Slices

<vertical-slice-rules>

- Each slice cuts a narrow but complete path through every layer (schema, API, UI, tests) — vertical, never a horizontal slice of one layer
- A completed slice is demoable or verifiable on its own
- **Cluster by context locality:** one ticket = one coherent context — work whose files/modules must be read together to do it. A fresh agent loads a tight, self-contained surface, not a scatter. Prefer slices touching files that already sit near each other.
- **Size each slice best-effort:**
  - **Target:** that ticket's own Builder execution — not the multi-ticket hive run as a whole — stays under **~80%** of the applicable context window, the ceiling past which a model's quality visibly drops.
  - **Window:** the smallest Builder context window this run allows (resolve from the approved model table when the Builder model is already known; otherwise the workflow's conservative default, **~200k**). Sizing against the largest available window (~1M, Opus/interactive) risks a ticket that overflows whatever smaller-window model actually builds it.
  - **Planning split:** ~40% initial load, ~40% implementation/tests/patching, ~20% reserve.
  - **Initial load** = files read/edited + their direct dependencies + ticket body + repo rules (CLAUDE.md, ADRs, profile) **plus the platform's auto-injected overhead** (rules, skills, tool definitions the platform silently pulls in) — never assume that overhead is zero.
  - **Not a runtime cap — a breakdown heuristic.** No dispatched leaf can reliably inspect its own live context usage mid-run (every usage view is shown to the human, not handed to the agent — Cursor's is also off in remote/SSH, so cloud runs have no live check at all), so the ceiling is held here, at breakdown. Surface that can't fit the initial-load target → **split the ticket**, never ship it oversized.
  - *Rationale evidence: a confirmed Cursor case ate ~50% of the window on injected skills alone; a real `/context` showed ~17% on injected tool defs before any work started. If a platform ever exposes an agent-readable usage value, this can become a real runtime self-check — see `/hive-lead`'s `ctx` note.*
- **Split trigger:** a candidate slice forcing a large cross-cutting surface (many files, broad refactor blast radius) → split into narrower slices, or pull the shared piece into a prefactor ticket (shared-helper rule below); never ship one context-heavy ticket.
- Any prefactoring should be done first
- Two tickets need the same new helper → extracting it is a prefactor ticket blocking both. Parallel Builders must never each write their own copy.

</vertical-slice-rules>

#### 3b. Blocking edges, phases, numbering, gates

Give each ticket its **blocking edges** — tickets that must complete before it starts. No blockers → starts immediately.

Tag each ticket with a **review phase** — related tickets sharing **one combined review pass** at the gate of the unit they all merge into, after they have all merged into it. (`/hive-gate-lead` runs that gate; `/hive-master` only routes on its verdict.)

**Human-facing phase gloss** — what the person approving the breakdown needs to know a phase buys them. A **unit's gate** fires once all its children have merged into it, and its tier comes only from that unit's **parent branch**:
- parent `main` → **full** gate (`/hive-panel` + `/hive-gatekeeper` + a `unit-flow` `/hive-scout` over the unit diff, then bugbot, then the full E2E), ending **held** — the fold is the human's alone (Law *H*).
- parent anything else → **light** gate (bugbot + E2E), and the gate-Lead merges the unit into its parent itself.

`/hive-gate-lead`'s **Canonical order** remains the executable source (`/hive-master` owns only the gate decision) — if this gloss and that order ever disagree, the gate-Lead wins and the gloss is the bug.

Executable rules:
- A phase is **not** an ordering — order comes only from ticket-level `Blocked by` edges.
- Independent tickets inside one phase have no ordering between them; the orchestrator still runs them one at a time, in any order deps allow.
- **A phase IS a unit** — `/hive-master`'s collapsed branch topology treats "tickets, phases, and runs" as the same kind of thing, each with its own branch and gate. No separate merge-unit field: put every ticket that must merge and gate together in the same phase, including wide-refactor batches sharing an integration branch (§3d).
- Phases are part of the breakdown the user approves, and of the published structure (sections in the ticket file, or the epic structure on a real tracker).
- Orchestrators (`/hive-master`, `/hive-lead`) read them **verbatim** — never re-derive them.

Every ticket therefore carries three orchestration fields: **`Blocked by`**, **`Gate:`**, and **`Tier:`** (below).

<numbering-rule>

Plain numeric numbering, always:

| Level | Format | Examples |
|---|---|---|
| Phase | `phase-N` | `phase-1`, `phase-2` |
| Ticket (within a phase) | `P.T` | `1.1`, `1.2`, `2.3` |
| Substep (within a ticket) | `N` | `1`, `2`, `3` |

**Never letter-number codes** — no `D0`, `P2a`, or `d1`/`d3`-style ids.

</numbering-rule>

Give each ticket a **Gate field**: `Gate: autonomous` (default) or `Gate: HITL`.

- `/hive-master` stops before any HITL ticket and waits for the human's go.
- The user flips tickets to HITL at breakdown approval (step 4).

Give each ticket a **Tier field**: `Tier: strict`, or omit it for the default loop.

- **Default** (untagged) — the cheap ticket loop: one broad review round, batched fixes, cheap re-reviews (`/hive-ticket-lead` §10.5). Correct for most tickets.
- **`strict`** — the full ceremony every round (`/hive-ticket-lead` §10.6). Tag a ticket `strict` on any of three grounds, named in the ticket body: **risk** (auth, payments, data integrity, security boundary), **blast radius** (many consumers, shared contract, migration), or **irreversibility** (destructive or one-way, hard to roll back).
- Tag deliberately and sparingly — `strict` on everything is the old cost back. A Lead may escalate a default ticket mid-run when evidence demands it; nobody downgrades a `strict` one.

#### 3c. Verification DOD

Give each ticket a **Verification DOD** — the checks `/hive-scout` later reads **verbatim** and runs in a browser. Strict format:

<verification-dod-rules>

- Numbered checks, each browser-observable: "Do X. Screen shows Y."
- Banned words in checks: "works", "correctly", "properly", "as expected". Every check names what is visible.
- Each check must be screenshot-able. Together the checks prove the ticket's behavior end to end.
- Each check **falsifiable** — must *fail* if the behavior is broken.
    - A check that passes no matter what the code does (or screenshots an unrelated "success") isn't a check.
    - The Inspector confirms each backing test fails without the change; the DOD holds the same bar.
- Verify the **persistent effect, not the acknowledgment** — "reload / re-query; the change is still there" beats "a 'Saved' toast appeared."
    - *A UI confirmation can render without the underlying effect happening.*
- **Bug tickets** include the regression check: "Repeat the old reproduction steps; the screen shows Y instead."
    - Class plausibly recurs → add ≥1 same-class check at a neighboring surface, drawn from `docs/agents/escaped-bugs.md` (read in step 1) and the Patcher's class-sweep findings when this ticket follows an in-run fix.
- No user-visible surface (pure backend, migrations) → mark the DOD `api-level`: checks run via curl or test output, same strict observable format.
- Tickets adding or touching an external input surface (route, form, upload, webhook) include ≥1 **negative security check**. Example shapes:
    - "Request `<endpoint>` with no token. Response is 401; the body contains no record data."
    - "Submit `' OR 1=1--` in `<field>`. Screen shows a validation error; the row count is unchanged."
    - More shapes: the profile's security rubric — `PROJECT-PROFILE.md`'s `Review lenses` entry names its path.
- **Invariant checks on failure-axis tickets** — a ticket touching any of the six failure axes must include invariant checks, not only happy-path checks.
    - The six axes: ownership & write authority, concurrency & interleavings, crash points & recovery, idempotency & retries, migration & deploy safety, tenancy & isolation. Canonical list + per-axis grill questions: `/hive-architect`.
    - Shape: "exactly one terminal event per turn", "a second concurrent start receives 409, no duplicate row", "a retried request yields exactly one logical record".
    - **Executability rule** — the Scout drives a browser (Playwright MCP or Cursor ComputerUse) or reads curl/test output. It cannot kill a process or inspect a raw DB row.
    - **Infra-manipulation invariants → `api-level` test as evidence** — an invariant needing infra manipulation (crash injection, forced concurrency, direct DB read) → `api-level` DOD whose evidence is the Builder's integration test performing it. Check = "Run `<the invariant test>`; output shows exactly one terminal event." The Scout *observes the test result* — never performs the kill/race itself.
    - **Directly observable invariants → UI/curl checks** — only invariants whose violation is genuinely browser- or curl-observable (e.g. "open two tabs and submit; the second shows a 409 toast") are written as UI/curl checks the Scout performs directly.
    - **Cover every spec invariant** — source = the spec's "Ownership and invariants" section (from `/hive-spec-writer`): every invariant listed there must be covered by ≥1 ticket's DOD in the breakdown. An uncovered invariant is a breakdown gap the user should see at the quiz step.
    - **Verify vs validate** — happy-path checks *verify* (the feature behaves); invariant checks *validate* (the failure mode is actually prevented). A ticket touching a failure axis with only verification-tier checks is a breakdown gap.

</verification-dod-rules>

#### 3d. Wide refactors

Wide refactors are the exception to vertical slicing.

A **wide refactor** = one mechanical change (rename a column, retype a shared symbol) whose **blast radius** fans across the whole codebase — one edit breaks thousands of call sites, and no vertical slice lands green.

Don't force it into a tracer bullet. Sequence it as **expand–contract**:

| Stage | What happens |
|---|---|
| **Expand** | Add the new form beside the old. Nothing breaks. |
| **Migrate** | Move call sites over in batches sized by blast radius (per package, per directory). Each batch is its own ticket, blocked by the expand. CI stays green batch to batch — the old form still exists. |
| **Contract** | Delete the old form once no caller remains. One ticket, blocked by every migrate batch. |

Batches that can't stay green alone → keep the sequence, but put them in **one review phase** — their shared integration unit (§3b's phase-is-a-unit rule) — and all block a final integrate-and-verify ticket in that phase. Green is promised only there.

### 4. Quiz the user

Present the breakdown sectioned by review phase, tickets numbered per the numbering rule. Per ticket show:

| Field | Shows |
|---|---|
| **Title** | short descriptive name |
| **Blocked by** | which other tickets (if any) must complete first |
| **What it delivers** | the end-to-end behaviour this ticket makes work |
| **Gate** | `autonomous` or `HITL` |
| **Tier** | `default`, or `strict` with the ground named |
| **Verification DOD** | the numbered checks |

Spec carries an "Ownership and invariants" section → also show the coverage map §3c's DOD rules require:

| Spec invariant | Covered by ticket/check |
|---|---|
| `<invariant>` | `1.2 DOD 3` |

An invariant with no row is the gap — surface it here, don't let the user discover it only via the panel (step 5).

Ask the user:

- Does the granularity feel right? (too coarse / too fine)
- Are the phases and blocking edges correct?
    - Does each ticket depend only on tickets that genuinely gate it?
    - Is each phase a coherent unit to review as one diff?
- Should any tickets be merged or split further?
- Which tickets should be `Gate: HITL` — where do they want `/hive-master` to stop and wait for them?
- Does each DOD prove the ticket — are the checks the right observable evidence?
    - The user approves each DOD here, before any code exists. `/hive-scout` reads it verbatim later.
- Is every spec invariant covered by a falsifiable check, per the coverage map above?

Iterate until the user approves the breakdown.

### 5. Invariant trace, then panel (conditional)

Runs after step 4's quiz, before step 6 publishes, so invariant→DOD coverage is independently checked on what will actually publish.

**Two stages, deterministic first. Never dispatch the panel before the trace has run** — the trace is what tells the panel where to look, and a panel run without it re-derives the mapping at model prices.

#### 5a. TRACE — deterministic, one cheap leaf

**One cheap leaf checks the invariant/AC → ticket → DOD mapping mechanically.** No judgement, no prose. Dispatch it through the wrapper, `review` class at the cheap tier (`/hive-lead` §4.3):

```
hive-dispatch run --role lens --model <id> --prompt-file <f> --run-dir "$HIVE_MEM/<run-key>" --
```

*(Pre-kickoff there is often no mount, but **`--run-dir` is required regardless** — the wrapper refuses a `run` without one. Pass a throwaway local dir beside the gap list — `--run-dir todo/trace-work-<slug>`, `/hive-ideate`'s `ideate-work/` pattern — and **never a path under `.hive-workflow/`**, which kickoff mounts shared memory onto and `git worktree add` refuses if it is non-empty. A local board is safe **only** here: one leaf, no branch, no siblings, so the tripwire and the branch lock have nothing to coordinate. The gap list itself still lands beside the documents, exactly as `/hive-panel`'s document mode handles an absent `$HIVE_MEM`.)*

**Per invariant and per accepted acceptance criterion, three lookups:**

| Check | Gap when |
|---|---|
| **Owning ticket** | no ticket claims it |
| **Falsifying DOD** | no DOD exists that could **fail** if the invariant were violated — a DOD that passes either way does not count |
| **Contradiction** | two tickets state incompatible behaviour for it |

**Output = a structured gap list, no prose** (shape in `references/templates.md`). Every row is settled by lookup, so every row is a **`mechanical`** finding in `/hive-panel`'s sense — admitted on the facts, never sent to a debater.

#### 5b. Panel — only over what the trace could not settle

**Scope the panel to the uncovered and contradictory invariants the trace found**, plus the always-retained lenses (`/hive-panel`'s *Adaptive depth*: security/isolation and destructive-migration, whenever the scope touches their territory).

**A certified ADR + approved breakdown gets a focused trace review; a full panel is for new or unsettled architecture.** Never run a full architecture panel over an ADR that already passed its own reviews — that re-litigates settled decisions and pays for the privilege. **Settledness sizes the panel; on its own it never removes one**: only the emptied scope below does that, and it takes more than settledness to empty. `/hive-panel` owns the depth table; you own handing it the trace output and the right scope.

**5b runs over a scope, and an empty scope is not a run.** Three conditions empty it, all three mechanical, all three checked against artifacts, and only all three together:

1. **5a's gap list has zero rows**: the `no gaps` shape in `references/templates.md`. Count rows; never judge them.
2. **Neither always-retained lens has territory in this scope**: security/isolation and destructive-migration. `/hive-panel` retains those two *at every depth* precisely because no depth reduction may drop them, and skipping 5b outright is the largest depth reduction there is. Scope touches either → those lenses **are** the scope, and 5b runs over them on a clean trace like any other.
3. **The architecture is settled by a nameable artifact**: an ADR or prior review whose own Pass verdict you can cite by path. **Unrecorded is unsettled**, whatever anyone believes: "certified" means a review happened and left a pointer, so if nothing is nameable this condition simply fails.

All three → nothing is left for a panel to review. Record in the published breakdown (step 6):

```
invariant panel: not run · empty scope · gaps:0 · retained-lenses:none-in-scope · settled:<ADR or panel-verdict path> · <gap-list path>
```

**Four distinct recorded states, and each names which one happened:** `not run · empty scope` (here) · `not triggered` (no trigger, below) · `waived by human` · a real panel verdict. **Never collapse one into another, and never represent any of the first three as a Pass.** A skip with no pointer to the gap list and the settling artifact is unauditable, which is the self-certifying escape hatch this corpus has already deleted once. Write the line or run the panel.

- **Trigger:** spec carries a real "Ownership and invariants" section → run 5a, then dispatch only one `/hive-panel` conductor in document mode, scoped per 5b, **unless the three conditions above emptied that scope** (never blanket-scoped over the complete spec once a trace exists). Never dispatch lenses or debaters directly, and never run one in-process. The conductor owns its roster, models, leaf dispatch, and debate procedure.
    - **Dispatch the conductor as an agent type that carries the `Agent` tool** (`general-purpose` / `claude` — **not** `Explore` / `Plan`, which are defined as "all tools except `Agent`"), or it **silently cannot spawn**, and a non-spawning conductor reads exactly like one that found nothing.
    - This skill runs pre-kickoff, so there is **no `master.md`** for the conductor to read — **state the host family and the leaf mechanism explicitly in its dispatch**, plus `$SKILLS_DIR`, the two document paths (spec, approved breakdown), **and 5a's gap list with its scope**, rather than leaving it to infer any of them.
    - The conductor owns debate and reconciliation. Read only its distilled verdict; never open its payload artifact.
- **When triggered, mandatory, and it has exactly two exits**: the empty scope above (mechanical, three conditions, recorded with its pointers) and the human's explicit waiver. **Never by default, never on judgement, and never because a panel looks expensive here.** No trigger → skip and record `invariant panel: not triggered`; not a waiver, nothing to record as one.
- **Edits loop back** — any edits after the panel re-run the invariant→DOD coverage check on the changed tickets.
- **"Not applicable" is not a free pass** — a spec declaring "Not applicable: no shared mutable state" doesn't skip scrutiny.
    - Verify the claim during codebase exploration (background jobs? queues? cross-process writers? concurrent users on one aggregate?).
    - Challenge it at the quiz step if anything contradicts it.
- **Reading the verdict.** The panel's document-mode return is **two parts — a distilled line, then the fix-plan table beneath it** (`/hive-panel`'s *Return*; this skill is not its source — check there if the exact shape drifts). Route on these, never the artifact itself:
    - **verdict** — `Pass`, zero proceed-blocking → publish (step 6). Any proceed-blocking finding → fix the named tickets' DODs or slices, re-quiz the changed tickets, re-run the panel.
    - **the fix-plan table** — `root cause | affected tickets | minimal correction`, one row per root cause (**no failure-class column in document mode**: the diff-mode table carries one for the gate-Lead's `--failure-class`, and no Patcher is dispatched here), returned beneath the line. Route off those rows; **never expect a bare `F#` list** — the line's `blocking:[…]` is the panel's gate check, not its fix plan — and never open the artifact to reconstruct one.
    - **must-address count** — untouched pre-existing blockers, never proceed-blocking itself but never invisible either: when non-zero, surface the count and the artifact pointer in your human-facing report so **the human** can open it — you still don't.
    - **artifact pointer** — carry it in your report; never open it yourself.
- **Budget: 2 panel rounds**, then stop and take the remaining proceed-blocking findings to the human with what each round tried. Never publish over an open proceed-blocking finding.
- **Waived** → record `invariant panel: waived by human` in the published breakdown (step 6), naming who waived it. A waiver is not a Pass, and not the empty-scope exit — never represent it as either downstream.

### 6. Publish the tickets to the configured tracker

Publish the approved tickets. `PROJECT-PROFILE.md`'s Tracker entry selects one of three modes — rehearsal, real-tracker, or GitHub-issues (Law *D*) — same tickets either way, only the blocking-edge shape and which system(s) receive them change. Real-tracker publication goes to every tracker the profile declares, one or both; GitHub-issues publication is always the one system, on this repo — the profile decides *which* of the three you're on, never a mix.

#### Rehearsal

One `tickets.md` in the repo root, **sectioned by phase per the template**. **For exercising the hive workflow itself — dry runs, skill testing — never for work that ships**:
- The file carries no tracker keys, so it is not a tracker item; `/hive-master` records `mode: rehearsal` and the run never folds to `main` (Law *D*).
- Ordering comes only from `Blocked by` edges — a phase is not an ordering (§3b), so a phase-sectioned file is not in dependency order; each "Blocked by" lists the **ids** it depends on (the numbering rule defines them; ids are what `/hive-master` §8 schedules over — titles are not a parse target).

Real work takes one of the two tracker paths below, per the profile's declared mode.

#### Real work — real tracker

One issue per ticket, in dependency order (blockers first), so blocking edges reference real identifiers.

- **Create every ticket yourself, in **every tracker the profile declares** (Law *D*: one or both, never a mix across tickets), using the available tracker tools** — with two declared, mirror the same body into each and never rely on auto-mirror; with one, one key is the complete answer. Never ask the human to create it for you. Record every declared key on each ticket; `/hive-master` kickoff refuses a ticket missing a declared key.
- Record the phase per the platform's structure (epic, milestone, or a `phase-n` label).
- Use the platform's native blocking / sub-issue relationship where it has one; otherwise set each ticket's "Blocked by" to the blocking issues.
- Apply the `ready-for-agent` triage label unless instructed otherwise — these tickets are agent-grabbable by construction.
- On Jira, `Gate: HITL` tickets carry the `Gate` field in the body and a `hitl` label (a gate marker, not one of the five triage labels in `docs/agents/triage-labels.md`) — **not** `ready-for-human`, which means *requires human implementation* (`docs/agents/triage-labels.md`) and would contradict the `ready-for-agent` label these tickets already carry.
- **Also write `tickets.md`** — same file rehearsal mode uses, here the committed audit copy (tracker issues are the execution record; this is not). Every ticket carries every declared tracker key.

#### Real work — GitHub-issues

One GitHub issue per ticket, on this repo, in dependency order (blockers first), so blocking edges reference real issue numbers.

- **Create every ticket yourself, as a GitHub issue (`GH-xxxx`, the issue number), using the available tracker tools** — never ask the human to create it for you (Law *D*). Record the one key on each ticket; `/hive-master` kickoff refuses a ticket missing it. Never additionally create a Jira or Linear item to satisfy a "both" habit carried over from real-tracker mode — one system is the complete, correct publication here.
- Record the phase as a `phase-n` label.
- Use GitHub's native sub-issue/blocking relationship where available; otherwise set each ticket's "Blocked by" to the blocking issue numbers in the body.
- Apply the `ready-for-agent` triage label unless instructed otherwise — these tickets are agent-grabbable by construction.
- A `Gate: HITL` ticket carries the `Gate` field in the body and a `hitl` label, same rule and same reasoning as the real-tracker path above.
- **Also write `tickets.md`** — same file rehearsal mode uses, here the committed audit copy. Every ticket carries its `GH-xxxx` key.

#### Canonical copy and verification

Either mode: **commit and push `tickets.md` before offering the next skill.** Cloud Leads are separate checkouts cloned from git — an uncommitted file does not exist for them. **Verify it landed** (`git cat-file -e origin/<branch>:tickets.md`, resolving `<branch>` from the current published branch) rather than trusting the push's exit code; a gitignored path or a failing hook both exit 0 having committed nothing.

**Do NOT close or modify any parent issue.**

Templates for both forms — the `tickets.md` file shape and the per-issue shape — live in [references/templates.md](./references/templates.md). Follow them exactly.

Either form: avoid specific file paths or code snippets — they go stale fast.

- **Exception:** a prototype snippet encoding a decision more precisely than prose can (state machine, reducer, schema, type shape) → inline it, noting briefly it came from a prototype.
- Trim to the decision-rich parts — not a working demo, just the important bits.

After publishing, offer **`/hive-walkthrough`** — the comprehension gate whose sign-off is `/hive-master`'s go-signal. `/hive-master` runs only after it.

- Includes a **single-ticket** breakdown — `/hive-master` classifies it (standalone vs. inside a bigger run) and delegates it to one Lead like any other ticket; the gate at merge follows from that classification. Per `/hive-master` §3 and §9.
- Don't route hive tickets around the gates via `/implement`.
