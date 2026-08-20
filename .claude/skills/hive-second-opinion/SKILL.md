---
name: hive-second-opinion
description: "Thao's hive workflow — cross-model check: an independent second opinion from a different model (a different family where the environment offers one) on a decision point. Agree → one unified recommendation; disagree → one debate round, then converge or present both views."
---

# Hive Second Opinion

Independent take on a decision point from a different model — different family where available.

**You** — the caller invoking this skill (`/hive-architect` or a Lead at rung 2) — run this procedure directly: pick the model, dispatch it, evaluate the result. The second-opinion model is a leaf you dispatch; it never runs this skill itself and spawns nothing.

**Input:** decision question, explicit options, your current recommendation, the model that produced it, compact context (constraints, code excerpts, doc pointers). Missing any of these → `blocked · incomplete second-opinion input · missing:<fields>`.

**Called by:** `/hive-architect`, at major solution-path decision points — approach choices, not clarifying questions. **And a Lead at escalation rung 2** (Law *I*), after a max-effort re-attack failed.

**Cadence:** one per decision. Also usable anywhere, on demand.

## Procedure

### Pick the model

Must differ from the model that produced the recommendation:
- Different **family** — if the environment offers one.
- Otherwise, a different **Claude** model.

*Which model exactly:* owned by `/hive-lead`'s **`skeptic` class table + cross-family rule** (`$SKILLS_DIR/hive-lead/SKILL.md` §4) (≠ the ticket's builder; pre-build, ≠ the model that produced the recommendation). Resolve the live id + effort there — this skill names no family roster of its own, so it can't drift from `/hive-lead`'s.

**No reachable model differs from the recommendation's** (host has only one family/model, or resolution fails) → never silently reuse the source model. Return `blocked · no independent model available · source:<model> · <reason>`.

**Read `$HIVE_MEM/<run-key>/master.md` first** (absolute — the bare relative path does not exist in a `-w` worktree; pull, then read): it records the host family, and that decides the mechanism — it is not a choice.

### Reach it

- **`cursor-*` host** → `cursor-agent -p`, invoked directly (per `docs/agents/hive-cloud-dispatch.md`). Reaches the full cross-family roster — genuinely independent. **Do not substitute** Cursor's Task tool or Claude Code's native subagent tool here: both expose a narrower model set (claude-family only, no gpt/xai — Law *B*) than the direct CLI does. Resolve the exact model + effort per `/hive-lead` §4; name both explicitly.
- **`claude-*` host** — no true cross-family model exists → **same-family fallback**: run a different Claude model (Sonnet vs an Opus recommendation, or vice versa) as adversarial reviewer.
- **No `master.md`** (a pre-hive `/hive-architect` call) → use any reachable model that differs from the source's — don't default to same-family just because there's no host record. Only fall back to a different Claude model if that's genuinely all the session can reach. Say which mechanism and model you used either way.

Same-family fallback prompt:

> *"an independent reviewer from a different team; challenge this recommendation, argue the strongest case for the alternatives before concluding."*

Say openly it's a **same-family fallback** (weaker independence) — never pass a same-model opinion off as cross-model.

### Board row and artifact

**Mounted run:** the calling Lead opens a board row for each call (role `second-opinion`, ids `so1`, `so2`… — **each call gets the next id, per `/hive-lead` §7's row schema; the debate round is a new call and takes the next id, never a reused row**) — a dispatch with no row is a defect (invariant 11). Write the decision, positions, and outcome to `$HIVE_MEM/<run-key>/second-opinion/<decision-slug>.md`; you are the sole writer, push before returning (`/hive-lead` §7's write-through rule).

**Standalone or pre-hive:** no board row, no `$HIVE_MEM` artifact. If a writable workspace exists, write `<decision-slug>-second-opinion.md` beside the design document or working file instead; otherwise the run has no artifact — `<artifact-path>` in the return is `n/a`. Either way, the distilled return line below carries **option names only**, never full reasoning — a human-facing call additionally renders both positions and reasoning in full (Output, below).

### Send the prompt

- **Send only:** question, explicit options, compact context, then the current recommendation last — nothing else, never the conversation transcript or your reasoning chain.
- **Order matters:** recommendation last so it doesn't frame the reviewer's read of the question and options.
- **Ask for:** a position — which option, why — then whether it agrees or disagrees with the recommendation it saw last.

*Independence is the product; an opinion that read your reasoning is an echo.*

## Output

The argument happens **before** the human sees anything.

**Agreement:** same recommended option, no material conflict in required safeguards or implementation path (an added non-blocking safeguard still counts as agree — fold it in). A different option, an outright rejection, or a conflict that changes whether/how the option proceeds — is disagreement.

**Insufficient context:** the reviewer can't responsibly choose given what it was sent — this is neither agreement nor disagreement, never force it into either. Return `blocked · insufficient decision context · source:<model> · reviewer:<model> · <artifact-path|n/a>`, naming the specific missing evidence in the artifact or human-facing output.

- **First pass agrees** → one unified recommendation + a line noting both models independently agree.
- **First pass disagrees** → **one** debate round, never more. The recommendation is already yours — no redispatch needed for that side, you weigh the second opinion's argument directly. Only the second-opinion model needs a fresh dispatch:
  1. Re-dispatch it with your reasoning for the recommendation, asking it to reconsider.
  2. It must respond with its (possibly revised) position **and the specific evidence for it** — a code path, a constraint, a trade-off. Restating the same position without new evidence doesn't move it.
  3. You reconsider your own recommendation against its argument, the same way.
  4. **Converged** = both of you now endorse the same option with materially compatible conditions — merge the reasoning into one recommendation. Either side still rejecting the other's option is **not** convergence.
  5. **Still split** — both views side by side, unmerged. You never break the tie yourself.

| | Source (recommendation) | Second opinion (<family>) |
|---|---|---|
| Position | ... | ... |
| Key reasoning | ... | ... |

**Called by a Lead (rung 2), no human present:** converged → the unified recommendation is the Lead's next move. Still split → do **not** wait for a decision; the split itself is the rung-2 result — the Lead climbs to rung 3 and returns its `blocked` line (grammar is the dispatching skill's — `/hive-master` §5), carrying both positions.

### Return

A distilled line — never the full argument transcript. Positions + reasoning live in the artifact where one exists (Board row and artifact, above); with no artifact, they exist only in the human-facing render.

```
agree · recommendation:<option> · models:<source>+<reviewer> · <artifact-path|n/a>
converged · recommendation:<option> · first-pass:disagreed · models:<source>+<reviewer> · <artifact-path|n/a>
split · source:<option> · second:<option> · models:<source>+<reviewer> · <artifact-path|n/a>
blocked · <reason> · source:<model> · reviewer:<model|unavailable> · <artifact-path|n/a>
```

For a human-facing call, additionally render the unified recommendation or the comparison table above — the return line is for a Lead's routing, the table is for the person deciding.
