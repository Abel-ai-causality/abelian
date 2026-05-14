# Abelian

> **Most multi-agent LLM frameworks ship. Abelian disciplines.**

![license](https://img.shields.io/badge/license-MIT-blue) ![version](https://img.shields.io/badge/version-v3.0.0-green) ![status](https://img.shields.io/badge/status-production--ready-brightgreen) ![inspired-by](https://img.shields.io/badge/inspired%20by-Kahneman%20%2B%20Karpathy-purple)

|                       | Naive multi-agent | LangGraph / CrewAI | **Abelian**                        |
|---|---|---|---|
| Peer falsification    | None              | Optional           | **Mandated (rule #18)**            |
| Termination           | Round budget      | Round budget       | **Mechanism-converge (rule #6)**   |
| Self-judge bias       | Unaddressed       | Unaddressed        | **Schema-grounded (rule #8 v2.2)** |
| Stuck → next move     | Manual            | Manual             | **Frame-break Protocol (5-step)**  |
| Receipts in real runs | —                 | —                  | **6 gate-catches / 1 skill shipped** |

> Your multi-agent LLM team agrees because they're trained on the same data. We made them falsify each other instead. **18** INVARIANTS, **mechanism-converge** termination, **schema-grounded** self-judge, **5-step** Frame-break Protocol. The discipline we run on our own research — and ship the receipts.

```
═══════════════════════════════════════════════════════════════
  Round 4 — peer-A: dict-cache / peer-B: lazy-init
═══════════════════════════════════════════════════════════════
  Mutate     → A diff +18 / -3, B diff +24 / -7
  Eval       → A: 2.34 → 0.41 (5.7×), B: 2.34 → 0.38 (6.2×)
  Cross-attack → A finds B's empty-list panic; B finds A's race
  Mutual inspire → R5 A proposes "lazy-cache hybrid" from B's frame
  Champion   → B (better metric), with A's empty-list test added
  Confirm    → ✓ commit-gate passed → def5678
═══════════════════════════════════════════════════════════════
```

What you see above is one real abelian round. Two peers, two diffs, cross-attack, champion. Output = tractable doc + testable metric. Tasks without testable metric → use `ce-brainstorm`.

## What's inside

- **Asymmetric peer discipline** (rule #18) — PROPOSE mode innovates + grounds; COUNTER mode strictly probes (each attack converts to a runnable PASS/FAIL probe; argumentation without falsification target is forbidden).
- **Attack Class Libraries** — default-7 + 4 named domain libraries (`research-class` / `audit-class` / `decision-class` / `doc-class`); cite by name in program.md, non-code tasks must opt into ≥1.
- **Portfolio mode** (K > 1) — MAP-Elites-style Quality-Diversity. Objective shifts from "optimize one" to **"fill cells"** across behavior axes.
- **Eval Discipline hierarchy** — level 1 (shell number) > level 2 (test pass/fail) > level 3 (frozen rubric self-judge) > level 4 (vibes — refused). Self-judge requires schema-grounding (rule #8 v2.2).
- **Goal-authoring stage** (rule #17) — `abelian --mission "<fuzzy text>"` runs a 5-pass compiler from fuzzy mission to rule #16-compliant program.md draft.

## TL;DR

| Number | Plain English |
|---|---|
| **18** | Eighteen INVARIANTS hardening against agent-soup, fabrication, drift, propose/counter discipline asymmetry, and 14 other catalogued failure modes. |
| **2** | Two stages (goal-authoring + loop), auto-detected from `--mission` flag. Sharp program.md skips the first; fuzzy mission triggers it. |
| **5** | Five valid termination conditions (rule #6). Five forbidden stopping-preference rationales. Convergence is mechanism-backed or refused. |
| **0** | Zero rounds/budget/wallclock caps. The loop runs until it converges. No timer-based stopping. |

## 3 Takeaways

- **Multi-agent LLM teams are agent-soup unless you mandate cross-falsification.** Two LLMs agreeing 30/30 can both be wrong about the same thing. Abelian forces propose AND attack between peers, with cross-family option to break the same-training-data prior.
- **Adversarial collab without a falsification probe is just two LLMs vibing.** Rule #18 forbids argumentation without a runnable PASS/FAIL probe in COUNTER mode. Every attack must convert to a probe (regression test / benchmark / shell command / grep). Argumentation that can't be probed reverts the mutation.
- **Mechanism-converge termination > round budgets.** Your loop should stop when it's *done*, not when the timer is up. Rule #6 hard-refuses 5 stopping-preference rationales ("diminishing returns", "time-remaining", "cleaner-to-ship", "deferred-future", "foundation-in-place"). Plateau triggers LLM creativity (Frame-break Protocol), not loop quit.

## Bold Takes

- **If your agent team has no falsification probe, you don't have a system — you have a vibe.**
- **Multi-agent teams of correlation engines are just bigger correlation engines.**
- **Kahneman wrote about adversarial collaboration in 2009. We made it executable in 2026.**
- **Round budgets are vibe coding for loops. Mechanism-converge or your termination is rationalized.**

## Reading Ladder

- **30s**: Read the [Round 4 demo block](#abelian) at the top + the 3 takeaways above. You'll know if this fits your problem.
- **5min**: Scan the [INVARIANTS table](#invariants-18) + the [program.md skeleton](#programmd-skeleton). You'll know how to author your first run.
- **30min**: Install + author a program.md for a real problem (matmul example below). Run one round. Read a compound doc in `docs/solutions/`.

## Reproduce In 3 Commands

```bash
git clone https://github.com/Abel-ai-causality/abelian.git ~/abelian
cd ~/abelian
bash integrations/codex/install.sh    # Codex CLI; for Claude Code see Install below
```

## Fits / doesn't fit

| Fits when | Doesn't fit when |
|---|---|
| 5+ rounds expected (mutual inspiration pays off) | Trivial fix (typo / single-line) |
| Multiple defensible directions exist | One obvious approach (use unilateral review tools) |
| Output is doc + testable anchor | Pure narrative without metric |
| Domain has cross-attack surface | Pure metric without rationale to attack |

Examples: speedup at non-obvious algorithm level, alpha research (sharpe + rationale), audit (rubric + review.md), architecture redesign (complexity metric + ADR), training-recipe search (eval loss + recipe.md). Real runs in `docs/solutions/skills/` include `viral-gtm-ai-startup` v1.1 framework extraction (run 2026-05-13-1958) and the refined-proposal it descended from (run 2026-05-13-1832).

## Loop (co-research mode default)

Round-0 once: **Program Contract Gate** (rule #16) — checklist + Takeaway + baseline eval + program-peer-challenge + sha256 hash + confirmation.

Per round:
```
0. Refresh    cat INVARIANTS.md && cat state.json (rule #3)
1. Propose    peer-A and peer-B each generate one mutation, different angles
2. Implement  each on its own branch
3. Eval       execution gate + metric ratchet
4. Cross-attack  peer-A → peer-B, peer-B → peer-A
                 round-N/peer-{A,B}.txt with ABELIAN-PEER-v1 header (rule #11)
5. Verify     each attack converts to a probe; fail = revert that branch
6. Champion   best surviving progress_delta wins; loser preserved for inspiration
7. Inspire    each peer reads other's mutation + attacks; feeds R+1
8. Converge?  goal-met / no-proposal-after-K-frame-breaks / mutual-KILL
              if "stuck" (challenge-clean OR progress stalled OR all
              candidate_routes progress ≤0), fire Frame-break Protocol (5-step
              creative escape) BEFORE termination claim
```

Termination → post-campaign escalation review writes compound doc to `docs/solutions/<category>/<goal-slug>-<date>.md`. Future runs read this first.

Single-axis verification (typo fix, single-axis verify, ship-prep against known target) is out of abelian's scope — use a separate review tool. abelian's diversity engine has no value on single-axis tasks.

## INVARIANTS (18)

| # | Rule | Notes |
|---|---|---|
| 1 | Peer challenge output on disk | not conversation context |
| 2 | Commit-gate (10 always-on + 1 conditional) | peer files / nonce / mtime / verdict / drift / pre-files / eval / mission_thread (#14) / evidence_class (#15) / goal-progress (#14); +codex-review when program.md `Code review: on` |
| 3 | Per-round refresh | cat INVARIANTS.md + state.json |
| 4 | Drift check | expected_head + branch + dirty-tree before any commit/revert. v2.16 distinguishes `contract-drift-stopped` (rule #16 hash mismatch) from ordinary `drift-stopped` |
| 5 | Pre-files snapshot | git ls-files inventory (revert tax) |
| 6 | Forbidden termination rationales | 5 stopping-preferences refused. Valid: `goal-met / no-proposal-after-K-frame-breaks / mutual-KILL / interrupted` |
| 7 | Verbatim Goal/Target/Constraints | in adversary prompts (no paraphrase) |
| 8 | Self-judge discipline | concrete-ground (code) or fuzzy-ground (doc/research/audit/decision) per `Eval ground:`; v3.0 has no off-switch for peer challenge (always required) |
| 9 | Execution gate | peer silence alone insufficient |
| 10 | Production-runtime safety | cron/supervisor/watchdog edits need extra discipline |
| 11 | Peer challenge header block | `ABELIAN-PEER-v1` + nonce + timestamp + `evidence_class:`. Peers may add informational `alternative_routes:` after attacks. (v3.0 rename from `ABELIAN-ADV-v1`; legacy double-read during deprecation) |
| 12 | Code Review supplemental gate | opt-in via program.md `Code review: on`; refuse on `[P1]`/`[P2]` |
| 13 | Self-attack is not adversary | conversation-level "I attacked own propose" without spawn = unilateral self-judge (rule #8 degraded), NOT co-research. 17× catch-rate gap (2026-04-29) |
| 14 | Mission Thread per round (v2.15) | 7-field block; goal_paraphrase fresh; ≥2 candidate_routes; selection_reason cites trade-offs; mission_relevance traces Takeaway.Validated_by |
| 15 | Evidence Class enum (v2.15) | peer header gains `evidence_class:` `theoretical / paper / replay / settled / dry_run / live` |
| 16 | Program Contract Gate (v2.16) | round-0: hard checklist + Takeaway-as-derived-contract + baseline eval + program-peer-challenge + sha256 hash + TTY-aware confirmation. Single-axis triage exits before round-0 (out of abelian's scope) |
| 17 | Goal-Authoring Stage (v2.17, v3.0 fold-in) | `abelian --mission "<text>"`: 5-pass protocol (triage + outcome distillation + metric forge + lever surfacing + Takeaway derivation) compiles fuzzy mission to rule #16-compliant program.md draft. Native answer to OKR's hierarchical decomposition. v3.0 dropped `abelian sharpen` subcommand; goal-authoring is now a stage of the unified loop |
| 18 | Asymmetric peer discipline (v3.0) | PROPOSE mode: innovative + grounded (cite ≥1 file/command/output, no vibes). COUNTER mode: strictly verification-oriented (convert attack to probe, run, return PASS/FAIL or CONCEDED or NON-CODIFIABLE-ESCALATED). Argumentation without falsification target FORBIDDEN in counter |

Full text: [INVARIANTS.md](INVARIANTS.md).

**Frame-break Protocol** (v2.15): 5 mandatory steps when round looks "stuck" — reject-pool mining, attack-class library escalation, peer framing swap (co-research), goal re-paraphrase from current state, cross-peer alternative_routes mining (co-research). Only `no-proposal-after-K-frame-breaks` (default K=2) terminates on exhaustion. Plateau triggers LLM creativity, not loop quit.

## Install

**Claude Code**:
```
/plugin marketplace add Abel-ai-causality/abelian
/plugin install abelian@abelian
```
Or `git clone https://github.com/Abel-ai-causality/abelian.git ~/.claude/skills/abelian` and restart. Invoke: `/abelian program.md`. Details: [drivers/claude-code/README.md](drivers/claude-code/README.md).

**Codex CLI**:
```bash
git clone https://github.com/Abel-ai-causality/abelian.git ~/abelian
cd /your/project
codex exec -s workspace-write "$(cat ~/abelian/SKILL.md ~/abelian/INVARIANTS.md ~/abelian/prompts/dissect.md)

Run abelian on program.md per spec. Maintain state.json under abelian/runs/<RUN_ID>/. Run till mechanism-based converge per INVARIANTS rule #6."
```
Details: [drivers/codex-cli/README.md](drivers/codex-cli/README.md).

**Codex skill discovery**:
```bash
git clone https://github.com/Abel-ai-causality/abelian.git ~/abelian
bash ~/abelian/integrations/codex/install.sh
```
Restart Codex after installing. Do not symlink the repo root into a skills directory; the installer generates a Codex-clean skill package from the canonical repo files and symlinks `~/.agents/skills/abelian` to it. Override with `SKILLS_HOME=/path/to/.agents/skills` for a repo-local install.

## program.md skeleton

```markdown
# Speedup matmul

## Goal
Reduce wall-clock time of matmul(A, B) on N=1000 random matrices.

## Task class
code

## Target
- src/matmul.py

## Eval
```bash
python3 bench.py | tail -1
```

## Metric
- name: best_of_5_seconds
- direction: min
- baseline: 2.0
- tolerance: 0.05
- target: < 0.1

## Constraints
- Must satisfy bench.py asserts (correctness contract)
- Pure stdlib (no numpy)

## Strategy
1. Loop reordering (cache locality)
2. Block matrix multiplication
3. Strassen recursion

## Attack Classes
- default
- correctness: result must match O(N³) baseline
- edge-cases: empty / 1×1 / non-square
- fp-precision: don't lose >1e-9 vs baseline

## Takeaway
- Success looks like: best_of_5_seconds < 0.1 (Goal: matmul wall-clock; Metric direction: min)
- Validated by: `python3 bench.py | tail -1` returns < 0.1, asserts pass (Eval: bench.py)
- Constraints: pure stdlib, no numpy (Constraints: cited above)
```

Default peers auto-detected from driver (`claude+claude` Claude Code; `codex+codex` codex CLI). Cross-family `claude+codex`, search shape (chains/depth/candidates/portfolio), code-review supplemental — all declared in program.md (no CLI flags). Fuzzy mission: `abelian --mission "<text>"`. Mechanism-based termination per rule #6 (no rounds/budget/wallclock cap). Manual abort: SIGINT. Legacy v2.x flags deprecated → see [MIGRATION.md](MIGRATION.md).

Progress is direction-normalized: `max` metrics improve when they rise; `min` metrics improve when they fall. Champion selection and frame-break gates use `progress_delta`, not raw metric delta.

## Receipts (what abelian has actually shipped)

- **viral-gtm-ai-startup v1.1.0** — operator-grade launch playbook (https://github.com/Abel-ai-causality/viral-gtm-ai-startup). **138** candidate frameworks surveyed → **25** root-essential principle anchors + **12** institutional VC canon refs. Mock-equip walkthrough passed 8/8 mechanical gates. Built via 2 chained abelian runs (`docs/solutions/skills/viral-gtm-ai-startup-build-2026-05-13.md` chained from `viral-gtm-ai-startup-refine-2026-05-13.md`).
- **6 mechanical bugs caught at round-0 gates** during those runs — including fabricated citations (`First Round "Beyond Disruption"` wasn't a real article; `a16z16z.com` was a typo and the article title didn't resolve), `grep -c` line-hits-vs-distinct-count probe drift, and stale-reference holdovers in rubric thresholds. Each catch is in the run's `program-peer-challenge-v{N}.txt` artifact.
- **Polymarket alpha research, Round 1 (2026-04-22)** — 2 BLOCKER typos that self-judged 4/4 clean were immediately caught by codex SQL schema-grounding. The exact failure mode that motivated rule #8 v2.2 (schema-grounding required for self-judge).
- **Compound docs auto-shipped** to `docs/solutions/` on every termination. Each run starts where the last one ended — `docs/solutions/<category>/<goal-slug>-<date>.md`.

## Roadmap

- **v3.0.0** (2026-05, current): one skill, one loop, one discipline. 18 INVARIANTS. Goal-authoring as stage. Cross-family peers (claude+codex). Mechanism-converge termination.
- **v3.1.0** (planned): **chained-run pattern** — `program.md depends_on: <RUN_ID>` field, with loop verifying prior run's `state.status == completed` + champion artifact exists. Surfaced from real chained-run usage (viral-gtm-ai-startup build chain: 2026-05-13-1832 → 2026-05-13-1958).
- **v3.2.0** (planned): **round-0 hardening** — pre-flight cite verification + probe arithmetic consistency check. Catches 4 categories of arithmetic bugs surfaced in real runs (`grep -c` line-hits vs distinct count, stale-reference drift, "all 4" lists with 5 items, etc.).
- **v4.0.0** (planned): **mock-equip walkthrough** standardized as rule #9 doc-only execution-gate. Orchestrator clears mental state, loads ONLY champion artifact, simulates N synthetic scenarios with line-cited friction. Catches operational-usability gaps that cross-family peer rubric scoring can miss (the agent-soup risk inside the gate itself).
- Submit ideas via [GitHub Issues](https://github.com/Abel-ai-causality/abelian/issues) or [Discussions](https://github.com/Abel-ai-causality/abelian/discussions).

## Contributing / Questions

- **Bug or broken probe?** Open an [issue](https://github.com/Abel-ai-causality/abelian/issues).
- **Want a new INVARIANT?** Open a [discussion](https://github.com/Abel-ai-causality/abelian/discussions) with an **empirical anchor** — link a concrete failure mode the rule misses + grep-able trace from a real run. Speculative tightening without evidence is friction.
- **Want to integrate with a new agent platform?** Open a discussion describing the driver pattern. New drivers live under `drivers/<name>/` as a `README.md` only — no wrapper scripts. Both shipped drivers (Claude Code, Codex CLI) are LLM-driven self-orchestration.
- **Style**: terse, judgment-first. No hedging when a concrete claim works. Match the existing INVARIANTS prose density.

Abelian is the methodology backbone for [viral-gtm-ai-startup](https://github.com/Abel-ai-causality/viral-gtm-ai-startup) (operator-grade launch playbook) and several Abel internal research pipelines. Real run artifacts live in `docs/solutions/`.

## Version

Current: **v3.0.0** — one skill, one loop, one discipline. See `git log` or GitHub Releases for history.

## License + Citation

MIT — see [LICENSE](LICENSE). Built on Kahneman's *adversarial collaboration* (2009), dissect-style attack-class taxonomy (`prompts/dissect.md`), Karpathy's "compound iteration loop" framing.

Citation:

```text
Stephen (cauchyturing), Abel AI Lab.
abelian v3.0.0: adversarial collaboration loop for deep + innovative + long-horizon LLM iteration. 2026.
https://github.com/Abel-ai-causality/abelian
```
