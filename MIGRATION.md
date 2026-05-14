# Migration: v2.x → v3.0

v3.0 collapses the v2.x mode lexicon into ONE loop with peer configuration. Legacy flags warn + map; legacy header reads accepted during deprecation window; behavior unified.

## Flag deprecations

| v2.x flag | v3.0 behavior |
|---|---|
| `--mode=co-research` | DEPRECATED no-op (warn). Co-research IS the loop in v3.0; no flag needed. |
| `--mode=unilateral` | DEPRECATED warn + EXIT non-zero. v3.0 has no unilateral mode. Use a different review tool for single-axis verification. |
| `--adversary=dissect` | DEPRECATED warn + map → `--peers=claude+claude` (Claude Code default) or `--peers=codex+codex` (codex CLI default). |
| `--adversary=codex` | DEPRECATED warn + map → `--peers=claude+codex` (cross-family). |
| `--adversary=both` | DEPRECATED warn + map → `--peers=claude+codex` AND `--code-review=on` (rule #12 supplemental gate). True 3-peer N>2 architecture is v4 territory. |
| `--adversary=off` | REFUSED. abelian without peer challenge is not abelian. |
| `abelian sharpen "<mission>"` (subcommand) | DEPRECATED. Replace with `abelian --mission "<text>"` flag. |

Loop emits clear warning to stderr + writes to `escalations.md` for every deprecated flag observed. Mapping happens once at startup.

## Header deprecation

| Header magic | v3.0 behavior |
|---|---|
| `ABELIAN-PEER-v1` | Preferred. Emitted by all v3.0 peer challenges. |
| `ABELIAN-ADV-v1` | Legacy. Commit-gate accepts during deprecation window for archived v2.x runs being resumed. New peer calls MUST emit PEER-v1. |

After 2 minor versions (target: v3.2), legacy `ABELIAN-ADV-v1` acceptance removed; runs with that header become inaccessible for resume (must be archived).

## File layout deprecation

| Path | v3.0 behavior |
|---|---|
| `$RUN_DIR/round-N/peer-A.txt` + `peer-B.txt` | Preferred. All v3.0 runs produce these. |
| `$RUN_DIR/round-N/adversary.txt` | Legacy from v2.x unilateral mode. Read-only acceptance for resume of archived runs; not produced. |

## What stayed the same

- All commit-gate checks 1-11 (rule #18 extends check 8 with PROPOSE-grounding requirement)
- All program.md schema (Goal/Target/Eval/Eval ground/Metric/Constraints/Strategy/Cells/Attack Classes/Takeaway)
- All state.json structure (rounds[], round_0, sharpening, frame_break_count_consecutive — keys unchanged)
- Mission Thread (rule #14), Evidence Class (rule #15), Frame-break Protocol, Round-0 Authoring Gate (rule #16), Goal-authoring 5-pass mechanics (rule #17 internals)
- "Adversarial collaboration framework" umbrella name (Kahneman-anchored)

## Rule #16 A v3.0 amendment

The v2.17 rule #16 A exception (`Strategy=1 IFF single-axis triage AND --mode=unilateral`) is REVERTED in v3.0. With unilateral mode dropped, single-axis triage now exits at Pass 0 with reroute diagnostic ("Mission is single-axis verification; abelian's diversity engine has no value here. Use a separate review tool."). v3.0 honest exit instead of v2.17's degraded-run-as-abelian.

## v3.x → v3.1 (chained-run dependency gate, archived state schema)

v3.1 introduces rule #19 Chained-run dependency gate and canonical `champion.artifact_path` schema in state.json.

**For programs that do NOT declare `Depends on:`**: no migration required. Behavior unchanged.

**For programs that DO declare `Depends on: <RUN_ID>` consuming a v3.0.x archived run**:

Archived state.json may contain duplicate top-level `champion` keys (a historical schema looseness). Symptom: `jq -r '.champion' state.json` returns `null` even though a champion object exists earlier in the file.

Rule #19 step 4 detects this and applies the migration heuristic (defined in INVARIANTS rule #19 Migration section). Summary:

1. Detect duplicate top-level `champion` keys.
2. Use the FIRST top-level occurrence (chronologically written first).
3. Resolve `artifact_path` directly, or fall back to aliased fields (`refined_proposal_path` → `skill_md_path` → first path-shaped non-null field).
4. Emit warning to current run `escalations.md`: `archived-champion-schema-migrated: <RUN_ID> via <which-heuristic>`.

**New v3.1 runs**: state.json MUST be written with a single top-level `champion` object containing the canonical schema. The loop driver enforces this on every state.json mutation.

**Future migration tool**: a script (`scripts/migrate-archived-state.sh`) MAY be added in a follow-up PR to rewrite archived state.json files into canonical form. Out of scope for v3.1.0.

## v3.x → v4.0 (doc-only execution gate becomes Mock-Equip Walkthrough)

v4.0 rewrites rule #9 doc-only opt-out path. Existing v3.x programs that set `termination_requires_execution_gate: false` must migrate.

**For programs with executable targets (typically task class `code`, or `mixed` when the champion is runnable)**: no migration required. Real execution (level 1-2 eval, deterministic non-LLM, peer saw output) remains the default gate.

**For doc-only programs (`doc`, `research`, `audit`, `decision`) declaring `termination_requires_execution_gate: false`**:

1. ADD `## Mock-equip scenarios` section to program.md with at least 3 scenarios. Each scenario is one prompt the champion artifact will be invoked on during mock-equip walkthrough.
   ```
   ## Mock-equip scenarios
   - Scenario A: <substituted prompt 1>
   - Scenario B: <substituted prompt 2>
   - Scenario C: <substituted prompt 3>
   ```

2. CONFIGURE state.json to receive the mock-equip trace under `state.execution_gate.mock_equip`.

3. On termination, the orchestrator clears mental state, loads ONLY the champion artifact, walks each scenario step-by-step, writes structured trace to `$RUN_DIR/downstream-confirmation/mock-equip-trace.md`.

4. Verify all 8 mechanical pass criteria (see INVARIANTS rule #9 / SKILL.md Execution Gate).

**v4.0 grace period**: programs that set `termination_requires_execution_gate: false` but provide no `## Mock-equip scenarios` will receive a LOUD TERMINATION WARNING with `escalations.md` note:

```
WARNING [v4.0]: doc-only target without ## Mock-equip scenarios.
v4.0 termination accepted with warning + escalations.md note.
v4.1 makes this hard gate-fail.
Migration path: add 3+ scenarios per program.md template, re-run with mock-equip walkthrough.
```

**v4.1 (planned)**: same condition becomes hard `gate-failed-terminal: doc-only-without-mock-equip`. Migrate before v4.1.

**Reference implementation**: `abelian/runs/2026-05-13-1958/program.md` (the ad-hoc precedent) + `abelian/runs/2026-05-13-1958/downstream-confirmation/mock-equip-trace.md` (the trace artifact). v4.0 formalizes this pattern into rule #9.
