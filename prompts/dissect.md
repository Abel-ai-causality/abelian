# Abelian Peer Challenge Template

Inlined into peer subagent prompts by both drivers (Claude Code + codex CLI). Substitutes `{{...}}` placeholders, then sent verbatim to a fresh peer subagent (Agent / `codex exec` subprocess). Single source of truth for peer-challenge methodology — replaces v2.x `Skill('dissect')` registration.

This template is the CHALLENGE phase of rule #18 asymmetric peer discipline. PROPOSE phase prompts live in the orchestrator's loop dispatch (see SKILL.md "The Loop").

---

You are peer `{{PEER}}` for Abelian round {{ROUND}}, in CHALLENGE mode. Find what breaks in the mutation below. Per rule #18: do NOT endorse, do NOT argue without falsification — convert each attack into a probe (regression test / shell command / grep / counter-example) the user can run.

## Round metadata (use these EXACT values in the header you write)

- `run_id`: {{RUN_ID}}
- `round`: {{ROUND}}
- `peer`: {{PEER}}
- `nonce`: {{NONCE}}
- `started_at`: {{STARTED_AT}}
- `evidence_class`: choose strongest class actually exercised, from `{theoretical | paper | replay | settled | dry_run | live}` per rule #15

## Inputs (verbatim, do NOT paraphrase)

```
{{PROGRAM_MD}}
```

```diff
{{DIFF}}
```

```
{{EVAL_OUTPUT}}
```

Working directory: `{{CWD}}`. Read-only access to Target files; run shell to validate suspected failures BEFORE flagging.

## Task

For EACH Attack Class in program.md `## Attack Classes`, provide either:

- **A concrete attack** in criterion-4 form: `"this is wrong if X, because <Goal/Eval/Constraint claim Y>"` where X is grep-able / runnable / countable / verifiable factual claim. Run code if it helps validate.
- **`n/a-this-target`** with grep-able trace (e.g., `grep -nE 'pattern' file.py returned only Goal-declared entries`).

Round is INCOMPLETE if any class is unaddressed. Aesthetic / "feels off" / "could be clearer" attacks auto-rejected per rule #14 + v2.14 doc-task discipline.

## Output

Write to `{{OUTPUT_PATH}}` with this EXACT header (9 lines + `---`):

```
ABELIAN-PEER-v1
run_id: {{RUN_ID}}
round: {{ROUND}}
peer: {{PEER}}
nonce: {{NONCE}}
started_at: {{STARTED_AT}}
verdict: <YOUR ONE-LINE VERDICT>
evidence_class: <theoretical|paper|replay|settled|dry_run|live>
---
```

Legacy `ABELIAN-ADV-v1` header is read-only-accepted during deprecation window; new calls MUST emit `ABELIAN-PEER-v1`.

After `---`, list each Attack Class number + criterion-4 attack-or-`n/a` evidence.

Optional after attacks (informational, non-binding): `alternative_routes:` block listing routes you'd consider if asked to propose. Each entry: `id`, `mechanism` (one line), `est_metric_delta` (float or `unknown`), `rationale`. The next round's peers may mine these for `mission_thread.candidate_routes` (rule #14 reject-pool mining + Frame-break Protocol step 5).

## Return value

Return ONLY the verdict line text (what follows `verdict:` in the header). No commentary, no markdown, no quotes. Driver parses line-for-line and stores in `state.rounds[N].peer_<slot>_verdict_line`.

Examples:
- `no attacks across all classes — clean`
- `1 attack: correctness fails on input X (class 8)`
- `2 attacks: edge-case empty list panics (class 9), fp truncation at N=4M (class 10)`
