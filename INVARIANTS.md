# Abelian INVARIANTS — read at start of every round

These rules are NON-NEGOTIABLE. Context compaction is not an excuse.
"Time efficiency" is not an excuse. "Trivial round" is not an excuse.
"Self-judge eval is small" is not an excuse.

If you are an LLM running this loop and you find yourself rationalizing
why a rule below "doesn't apply this time" — that is the exact failure
mode the rule exists to catch. Stop and re-read.

## 1. Adversary output must be on disk

Each round's adversary call writes `$RUN_DIR/round-N/adversary.txt`
(unilateral) or `$RUN_DIR/round-N/peer-A.txt` + `peer-B.txt`
(co-research) BEFORE the call returns. Empty file = adversary was not
actually run. Conversation-only adversary output is invalid and fails
commit-gate (rule #2).

## 2. Commit-gate (7 checks, all must pass before `git commit`)

1. `$RUN_DIR/round-N/adversary.txt` exists and is non-empty.
2. The file starts with the standard adversary header block (rule #11)
   and its `nonce` field equals `state.rounds[N].adversary_nonce`.
3. The file's mtime is later than `state.rounds[N].adversary_started_at`
   and earlier than `now()`. (`stat -c %Y adversary.txt` vs ISO parse.)
4. `state.rounds[N].verdict_line` appears verbatim in `adversary.txt`
   body (`grep -qF "$VERDICT" adversary.txt`). Closes the "compacted
   agent fabricates a clean review" hole.
5. Drift check passes (rule #4).
6. `$RUN_DIR/round-N/pre-files.txt` exists (rule #5).
7. Eval ran in this round's process and produced the metric value
   recorded in `state.rounds[N].metric_value`.

Any failure → revert this round (`git checkout` + scoped clean of
new files via pre-files diff), mark round `gate-failed` in state, do
NOT commit. Hard gate.

## 3. Per-round refresh (anti-compaction)

At step 0 of every round, before any other action:

```bash
cat $SKILL_DIR/INVARIANTS.md
cat $RUN_DIR/state.json
```

Re-read from disk, not from your memory of the conversation. After 3+
rounds your memory of these rules is wrong; the file is the source of
truth. Skipping this read because "I remember the rules" is a
rationalization (see header note).

## 4. Drift check before any write/commit/revert

Git mode only. Before any commit, revert, or rollback:

- `git rev-parse HEAD` == `state.expected_head`
- `git branch --show-current` == `state.branch`
- The set of dirty files matches this round's plan. Compute it via
  three SEPARATE git commands. **Do NOT parse `git status --porcelain`**
  — wrapping it in `.strip().splitlines()` eats the leading space of
  the first ` M file` entry, off-by-one drops the first character of
  the filename, and a smoketest false-positive ("low.py" instead of
  "slow.py") confirmed this gotcha 2026-04-28. Use:

  ```python
  modified  = run(['git','diff','--name-only']).splitlines()
  staged    = run(['git','diff','--cached','--name-only']).splitlines()
  untracked = run(['git','ls-files','--others','--exclude-standard']).splitlines()
  dirty = sorted(set(modified + staged + untracked))
  ```

  Allowed dirty: files this round's plan declares as Target, plus any
  file under `abelian/runs/<RUN_ID>/`. Anything else = drift.

Any mismatch → set `state.status = "drift-stopped"`, write nothing
else, terminal-only summary, exit. Do not attempt to "recover" — the
human must investigate.

## 5. Pre-files snapshot before mutate

Before step 2 (Mutate) writes anything:

```bash
mkdir -p $RUN_DIR/round-N
{ git ls-files -z; git ls-files -z --others --exclude-standard; } \
  | sort -zu > $RUN_DIR/round-N/pre-files.txt
```

Used by scoped revert to remove new files cleanly. Missing pre-files
= commit-gate refusal (rule #2 check 4). Missing pre-files = revert
cannot run cleanly = drift-stopped on first failure.

## 6. Forbidden termination rationales

Abelian runs **till converge**. There is no `--rounds` cap, no `--budget`
flag, no wallclock cap (v2.9 removed all of these). The only fallback for
"stop now" is the user sending SIGINT/SIGTERM — manual emergency abort,
marked `status=interrupted`, NOT a valid termination signal.

The loop MUST NOT terminate or write a "done" claim if the
load-bearing reason for stopping reduces to ANY of:

- "Diminishing returns" / "remaining work is lower-value"
- "Time remaining is short" / "tokens running out" / "running long"
- "Deferred to future campaign / TODO / next session"
- "Foundation in place" / "natural stopping point" / "good break here"
- "Cleaner to ship what we have than fold in more"

These are stopping preferences, not goal-fulfillment. Termination is
justified only by mechanism (N=3 is the hardcoded internal default for
plateau / exhaustion thresholds — not a user flag):

- **Goal met** — eval ≥ target (unilateral) OR champion ≥ target (co-research)
- **Adversary exhausted across attack classes for N=3 consecutive rounds** + execution gate (rule #9). Each attack class addressed with no concrete attack across N rounds.
- **Plateau** — N=3 consecutive rounds with no eval improvement. Co-research mode adds: AND candidate edit-distance falling between peer mutations (diversity collapse).
- **Mutual KILL deadlock** — N=3 rounds where every peer attack succeeds on both sides (co-research only).

If a mechanism signal would not fire by round 3, the loop has not
actually converged. Either tighten program.md (target/eval) or wait
for the user to abort. "Running long" is a forbidden rationale —
either fire a real mechanism signal or let the user SIGINT.

## 7. Verbatim Goal/Target/Constraints in adversary prompts

The adversary subagent prompt MUST include the literal text of
`program.md` Goal / Target / Constraints / Attack-Classes — quoted
inline in the prompt or read from the actual file. Paraphrasing
forbidden. Agent-rewritten goal in adversary's head is the
source-of-truth-drift escalation trigger; verbatim quoting prevents
the silent rewrite.

This applies to:
- Adversary calls (every round)
- Self-judge calls (every round, when used)
- Co-research peer-attack prompts (both directions)
- Post-campaign escalation review

## 8. Self-judge discipline

- `--adversary=off` + Eval=`self-judge` → **hard refuse to start**.
  No degradation path. Zero LLM check on vibes eval = structurally
  unsafe.
- Self-judge MUST verify external schema (file paths, columns, API
  contracts, function signatures) by reading actual source before
  scoring (v2.2 schema-grounding). Self-judge that scored ≥ rubric_max
  without grounding step is auto-rescored to 0 on affected dimensions.
- Self-judge runs in isolated subagent, no shared context with mutator.
- Rubric frozen in `program.md` Metric BEFORE loop starts; mid-run
  rubric drift forbidden.

## 9. Execution gate (termination requirement)

Termination requires at least one round per cell produced an artifact
that:

1. Was actually executed in this loop (level 1 or 2 eval — shell
   number or test-suite, not self-judge).
2. Eval at execution time was deterministic non-LLM.
3. Adversary saw the execution output, not just the spec or diff.

Adversary-exhaustion alone is necessary but not sufficient. Two LLMs
reaching mutual silence on a spec-only target ≠ artifact survived
real execution. Doc-only mode requires explicit
`termination_requires_execution_gate: false` + a downstream-confirmation
step.

## 10. Production-runtime safety

If Target includes a file imported by a continuously-running process
(cron, supervisor, systemd watchdog, hot-reload server, FastAPI
auto-reload), at least one of these MUST hold for the campaign:

- (a) Suspend the production process for the campaign window. Resume
  + verify one full cycle clean before claiming the cell done.
- (b) Eval against a snapshot of deployed state alongside fresh-fixture
  eval (e.g., copy of prod sqlite at campaign start; re-run eval
  against it after each cell).
- (c) Idempotent-ALTER assertion on every schema column added: `git
  diff` of the round MUST show ALTER TABLE … ADD COLUMN coverage for
  every new column. `CREATE TABLE IF NOT EXISTS` alone silently skips
  against an incumbent schema.

Diagnostic for whether this rule applies: search the cron / supervisor
config for the Target file path. If found, the rule applies. If
unsure, suspend cron — cheap insurance.

## 11. Adversary header block (mandatory format)

Every `adversary.txt` (and `peer-A.txt` / `peer-B.txt` in co-research)
MUST begin with this header block, written by the adversary subagent
itself, before any attack content:

```
ABELIAN-ADV-v1
run_id: <state.run_id>
round: <N>
peer: unilateral | peer-A | peer-B
nonce: <state.rounds[N].adversary_nonce>
started_at: <ISO 8601 with milliseconds>
verdict: <single-line verdict, identical to state.rounds[N].verdict_line>
---
<attack content begins here>
```

**Mutator protocol**:

1. Before calling `Agent(adversary)`, generate:
   - `nonce = python3 -c "import secrets; print(secrets.token_hex(8))"`
     (16 hex chars)
   - `started_at = $(date +%Y-%m-%dT%H:%M:%S.%3N%z)` (ISO ms)
   Write both to `state.rounds[N].{adversary_nonce, adversary_started_at}`.

2. The adversary prompt MUST include `run_id`, `round`, `peer`,
   `nonce`, `started_at` verbatim and instruct the subagent:
   > "Write the ABELIAN-ADV-v1 header block (these EXACT field
   > values) as the first lines of `$RUN_DIR/round-<N>/adversary.txt`,
   > then `---`, then the attack list. Do not omit the header. Do
   > not change the nonce or started_at."

3. After subagent returns, extract the verdict line from the body
   (or accept it via subagent return value), record in
   `state.rounds[N].verdict_line`. Commit-gate (rule #2) verifies the
   header before commit.

**Honest scope**: this is friction-level defense, not cryptographic.
A mutator deliberately bypassing the adversary call could fabricate
the entire header (it just wrote the nonce to state). The header's
real value is anti-Threat-1 (compaction oversight): an R5+ compacted
mutator that "forgot" it skipped adversary cannot silently hand-write
adversary.txt — it must first cat state.json to find the round's
nonce, which triggers the rule #3 reading reflex (cat INVARIANTS) and
surfaces rule #1 ("must actually call adversary"). The friction
converts a silent skip into a deliberate sabotage, and deliberate
sabotage is loud + traceable in post-campaign review of round
artifacts.

For Threat-2 (deliberate fabrication) and Threat-3 (lazy adversary
output), see attack-class checklist (program.md) and post-campaign
escalation review respectively.

## 12. Code Review supplemental gate (opt-in, v2.11+)

`--code-review=on` enables an additional code-quality review layer that
runs **after** rule #1's dissect/codex adversary call and **before**
rule #2's commit-gate. It uses codex CLI's purpose-built `codex review`
subcommand (with its built-in P1/P2/P3 severity schema) — separate from
and supplemental to rule #1's domain-specific attack-class adversary.

**Why supplemental, not replacement**: dissect adversary covers
domain-specific attack classes the program.md author defined (e.g.
"correctness must match O(N²) baseline", "integer-overflow at N=4M").
codex review covers code-quality issues codex's built-in prompt is
tuned for (style, common bugs, missed edge cases). Different
maintenance domains; both useful when stakes are high.

**Invocation**:

```bash
codex review --uncommitted -c 'model_reasoning_effort="high"' \
  2>&1 | tee $RUN_DIR/round-N/codex-review.txt
```

`--uncommitted` reviews staged + unstaged + untracked changes — exactly
abelian's state at this loop step (mutation written to working tree, not
yet committed). If `node` is not in PATH, prefix with `bun
/path/to/codex` (per `~/.bashrc` shim convention).

**Commit-gate addition** (rule #2 grows from 7 → 8 checks when
`--code-review=on`):

8. `$RUN_DIR/round-N/codex-review.txt` exists, non-empty, AND contains
   no `[P1]` or `[P2]` severity markers.

**Loop semantics**: night-shift's "fix → re-review → max 10 rounds →
revert" pattern is allowed but optional. Simplest form: a single
review pass; if P1/P2 present, revert immediately and let mutator
propose differently next round. Up to the orchestrator's discretion;
the gate-check itself is binary.

**Graceful degradation** (loud, never silent):
- `--code-review=on` + codex CLI unavailable (binary missing OR
  `~/.codex/auth.json` absent) → skip rule #12 only (don't fail loop),
  write notice in 3 places (console + escalations.md + History row)
- Fall back to rule #2's 7-check gate with rule #12 marked "skipped:
  codex unavailable"

**Honest scope**: rule #12 has **no nonce header protection** (rule #11
does not apply — codex review's CLI doesn't accept custom prompt
injection that would let the orchestrator inject a header block). A
mutator that wanted to fabricate `codex-review.txt` could. The
practical mitigation: mutator cannot predict codex's actual review
wording, so fabrication requires writing a plausible P1/P2-free
review — non-trivial effort. This is friction-level defense, weaker
than rule #11's nonce protection but better than no gate.

**When to enable**: ship-prep, PR-level decisions, security-sensitive
mutations, code-quality ratchet campaigns. Default off because cost
doubles per round and most campaigns (speedup / refactor) are
already covered by dissect's attack-class checklist.

## 13. Self-attack is not adversary (v2.12)

A mutator agent attacking its own propose in conversation context
(no spawn, no isolated context, no nonce header) is unilateral
self-judge (rule #8 degraded mode), NOT co-research (rule #1 +
co-research mode default since v2.10). RLHF prior overlap = same-prior
collapse vector.

**Empirical anchor (2026-04-29 abelian self-audit dogfood)**: peer-A
(orchestrator Claude) self-attack on 5 mutation propose found 1
obvious finding (Validate-vs-Eval overlap). Spawned peer-B (Agent +
Skill('dissect')) on identical input found 17 attacks across 5
dimensions, including 4 foundational frame challenges peer-A could
not see (e.g., "abelian vs night-shift cosmetic copy" / "propose+verify
structurally not fit fuzzy task" / "Phase 2 dogfood polymarket =
best-fit case sampling bias"). 17 / 1 = 17× catch rate ratio confirms
RLHF prior overlap is severe — mutator and self-attacker share the
same prior over BOTH "what to mutate" and "how to attack mutations".

**Trigger**: any conversation-level abelian/autoresearch reference
that involves ANY of:

- ≥3 mutation proposals (protocol / architecture / strategy / memory)
- Protocol-level change (INVARIANTS / state.json schema / commit-gate)
- "verdict / done / keep / revert / accept / pareto / trade-off"
  vocabulary applied to mutation evaluation

**Required action when triggered**: spawn dispatched adversary (Agent
+ Skill('dissect') OR codex exec subprocess) writing nonce-headered
adversary file (rule #11) BEFORE reaching verdict. Self-attack in
conversation context does NOT count.

**Forbidden mode**: "I attacked my own propose and found these issues"
phrased as adversary substitute. This is mutator's same RLHF prior
attacking same RLHF prior — by construction, cannot find what the
prior overlooks.

**Graceful degradation if you genuinely cannot spawn**: write a single
file `unilateral-self-judge.md` (no nonce header, no rule #11
protection) with explicit `MODE: unilateral self-judge — degraded;
rule #1 not satisfied; verdict provisional only` header. Notify in 3
places (console + escalations.md + conversation log) — same loud-
degradation pattern as rule #8 / `--adversary=off`.

**Honest scope**: rule #13 has no nonce protection (orchestrator can
fake the "I'm self-judging" header just like rule #11's adversary
header). The structural gate IS the spawn requirement; if you skip
spawn, the only defense is loudness — declare degraded mode + tell
the user to run co-research peer manually.

**This rule applies even when**:

- Discussing abelian protocol changes (meta-task — do NOT cite
  `feedback_human_peer_beats_llm_team_for_design_work.md` "small surface
  design" memory to justify skipping rule #13; that memory covers
  human-as-peer-B reactive review, not LLM mutator's own self-attack)
- "Just one quick analysis" / "informal discussion" — these are
  forbidden-rationale variants of rule #6
- User asks for your "honest opinion" — opinion ≠ adversary; opinion
  is mutator output, adversary is dispatched challenge with isolated
  context

**Anti-pattern caught 2026-04-29**: peer-A used its own freshly-shipped
memory (`feedback_human_peer_beats_llm_team_for_design_work.md`) to
retroactively justify skipping co-research, treating Stephen's reactive
question as "human peer-B substitute". Stephen's question is review,
not active peer-attack. Real peer-B is a spawned agent with isolated
context. Self-justification by selectively invoking own memory is the
exact same-prior collapse rule #13 prevents.
