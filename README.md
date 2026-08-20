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
| `hive-architect` | the grill — relentless interview to sharpen a plan/design, writing ADRs + glossary via `domain-modeling` as it goes, with `hive-second-opinion` at major decisions |
| `hive-second-opinion` | cross-model gut-check at a decision point, called by `hive-architect` |
| `hive-ideate` | generates options past the obvious ones (`approach` mode feeds the grill); its `pre-ship` mode is left in verbatim but its downstream consumer (`/hive-scout`) is a build-phase skill not included here |
| `hive-spec-writer` | turns the sharpened conversation into a spec/PRD, publishes to the tracker |
| `hive-issue-planner` | breaks the spec into tracer-bullet tickets with blocking edges + Verification DOD; `references/templates.md` holds the **Jira/Linear/GitHub issue template** and the rehearsal `tickets.md` template |
| `hive-panel` | heavy adversarial review — used here at issue-planner's step 5 as the invariant-trace panel over the spec + approved breakdown (its other normal use, reviewing a built unit's diff, doesn't apply in a planning-only repo) |
| `hive-walkthrough` | final comprehension gate — briefs a human on the settled plan before any code gets written |
| `hive-report` | shared voice/format rules every human-facing report above uses |
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

```
hive-profile-builder          (once, per repo)
        ↓
hive-architect  ──uses──▶ hive-ideate (approach mode), hive-second-opinion, domain-modeling
        ↓
hive-spec-writer
        ↓
hive-issue-planner  ──uses──▶ hive-panel (step 5, invariant trace)
        ↓
hive-walkthrough
```

`hive-report` is a cross-cutting dependency (voice/format) for every human-facing step
above.
