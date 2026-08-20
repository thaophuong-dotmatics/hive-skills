# hive-skills

Planning-only subset of Thao's `hive-testbed` skill corpus: everything needed to go
**profile → grill → spec → tickets(+ Jira/Linear/GitHub template) → walkthrough**,
with none of the dispatch/build machinery.

Extracted 2026-08-20 from `hive-testbed` (`hive-flow-v2` branch). That repo is the
source of truth for anything beyond planning — this is a deliberate one-way snapshot,
not a synced copy.

## What's here and why

| skill | role |
|---|---|
| `hive-profile-builder` | scans a repo, writes `PROJECT-PROFILE.md` — the router every other skill here reads for tracker/docs info. Run this first in a new repo. |
| `hive-architect` | the grill — relentless interview to sharpen a plan/design, recording every settled decision as an ADR + glossary term via `domain-modeling` as it goes, and triggering `hive-second-opinion` at major solution-path forks — both are real, in-file calls, not aspirational |
| `hive-second-opinion` | called on demand by `hive-architect` at a major solution-path fork (approach choice, not a clarifying question) — an independent cross-model read, surfaced before the human decides |
| `hive-ideate` | generates options past the obvious ones, meant to run *before* the grill on high-stakes plans (`approach` mode). **Not actually wired in** — `hive-architect`'s own file never mentions it; the hand-off is an open TODO inside `hive-ideate`'s file. Today a human runs `/hive-ideate` first and hands the survivors to architect manually. Its `pre-ship` mode is left in verbatim but its downstream consumer (`/hive-scout`) is a build-phase skill not included here |
| `hive-spec-writer` | turns the sharpened conversation into a spec/PRD, publishes to the tracker |
| `hive-issue-planner` | breaks the spec into tracer-bullet tickets with blocking edges + Verification DOD; `references/templates.md` holds the **Jira/Linear/GitHub issue template** and the rehearsal `tickets.md` template |
| `hive-panel` | heavy adversarial review, called on demand at issue-planner's step 5 — only when the spec carries a real "Ownership and invariants" section, or scope touches security/isolation or a destructive migration (those two lenses always run when relevant). Skipped when the ADR behind the ticket already passed its own review. Its other normal use, reviewing a built unit's diff, doesn't apply in a planning-only repo |
| `hive-walkthrough` | final comprehension gate — briefs a human on the settled plan before any code gets written. Doesn't offer a next step, it gates: only an unambiguous sign-off unlocks `/hive-master` (build, outside this repo) |
| `hive-report` | shared voice/format rules — used in 4 of the 5 pipeline stages (architect, spec-writer, issue-planner, walkthrough all cite it by name for how they address the human). `hive-profile-builder` skips it, since it mostly writes a repo-facing file, not a human report |
| `grilling`, `domain-modeling` | generic (non-hive) dependencies `hive-architect` invokes directly; bundled so this repo is self-contained |
| `hive-rules.mdc` | the full rulebook (Laws A–M) from the parent corpus, included for the Law references inside `hive-panel`/`hive-ideate`. Most Laws govern the excluded dispatch/build system (Master, Leads, Builder, Inspector, Scout, Gatekeeper, PR-bugbot-triage) and don't apply here — the planning skills mainly lean on Law *D* (tracker discipline) and Law *F* (workflow edits get their own branch). |

## Deliberately excluded

Everything that builds or reviews *code*, and everything whose core mechanism is the
`hive-dispatch` CLI wrapper for orchestrating a multi-agent build run: `hive-master`,
`hive-lead`, `hive-ticket-lead`, `hive-gate-lead`, `hive-gatekeeper`, `hive-builder`,
`hive-inspector`, `hive-scout`, `hive-pr-bugbot-triage`, and the `bin/hive_dispatch.py`
wrapper itself. None of that is copied here.

`hive-panel` and `hive-ideate` still mention `hive-dispatch fanout`/`run` for their own
internal sub-agent fan-out — but both also document a no-CLI fallback (native
`Agent`/`Task`-tool dispatch, the same carve-out the parent corpus calls Law *A*'s
no-CLI path), so they work in a repo that never installs the wrapper.

## Known dangling references (verbatim-copy tradeoff)

These files were copied byte-for-byte from `hive-testbed`, so a few sentences point at
things this repo doesn't have:

- `hive-architect`: "the repo's bootstrap skill" (`PROJECT-PROFILE.md` entry 1) may name
  a dispatch skill from a fuller hive install. If entry 1 says "none" — the normal case
  here — create the branch/worktree yourself, per the skill's own fallback instructions.
- `hive-ideate`'s `pre-ship` mode says it "feeds `/hive-scout`" — not present here; treat
  its output as informing a human review instead.
- `hive-panel` and `hive-issue-planner` reference `/hive-lead` section numbers and
  `$HIVE_MEM` (shared run memory) for their dispatch mechanics — irrelevant to the
  planning-only path (issue-planner step 5's Panel pass over a spec+breakdown), relevant
  only if you later wire the no-CLI fallback for real sub-agent fan-out.

## Pipeline order

The fixed sequence — every plan walks all five, in order:

```
hive-profile-builder   (once, per repo)
        ↓
hive-architect
        ↓
hive-spec-writer
        ↓
hive-issue-planner
        ↓
hive-walkthrough
```

`hive-ideate`, `hive-second-opinion`, and `hive-panel` are deliberately **not** drawn into
this chain — they're tools architect and issue-planner reach for situationally (trigger
conditions above), not steps every run passes through. `domain-modeling` and `hive-report`
*do* run on every pass, but continuously inside a stage rather than as a step of their own.

### Self-driving handoff

Nothing external routes this pipeline — each stage's own file names what to run next, in
its own words, at the end of its run:

- **`hive-architect`**: *"Offer `/hive-spec-writer` to turn the outcome into a spec. Also
  mention optional `/hive-panel`."*
- **`hive-spec-writer`**: offers a branch — run `/hive-panel` over the spec now while it's
  cheap to change, or go straight to `/hive-issue-planner`.
- **`hive-issue-planner`**: *"After publishing, offer `/hive-walkthrough` — the
  comprehension gate whose sign-off is `/hive-master`'s go-signal."*
- **`hive-walkthrough`**: doesn't offer, it gates — only an explicit human sign-off
  unlocks `/hive-master` (build, outside this repo).
- **`hive-profile-builder`**: the one exception, no self-narrated handoff — the pointer to
  architect lives one level up, in the `PROJECT-PROFILE.md` it writes.
