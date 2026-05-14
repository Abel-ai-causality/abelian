# Changelog

## v4.0.0 - Doc-only execution gate as Mock-Equip Walkthrough (MAJOR, planned)

Major rewrite of rule #9. Doc-only opt-out (`termination_requires_execution_gate: false`) now requires an explicit Mock-Equip Walkthrough with structured trace and 8 mechanical pass criteria. The historical vague "downstream-confirmation step" wording is closed.

- Mock-equip positioned as **degraded-mode execution evidence**, not a third LLM judge. Stands in for real execution when real execution is structurally unavailable (research / decision / human-doc artifacts).
- Protocol: orchestrator clears mental state → loads ONLY champion artifact → walks each scenario step-by-step → writes structured trace to `downstream-confirmation/mock-equip-trace.md`.
- Pass criteria (8 mechanical): file exists, ≥3 scenarios, per-scenario friction sections, 0 BLOCKER, MAJOR ≤ ceil(N×2/3), MINOR ≤ N×2, line-cite count == friction count, exactly 1 Summary.
- Line-cite HARD-REJECTION: any friction entry missing `skill_line_cited: <integer>` is auto-rejected. No vibes-scoring.
- Forbidden patterns: vibes-scoring, champion-built-by-walker (no mental-state clear), cherry-picked scenarios.

**v4.0 grace period**: programs with `termination_requires_execution_gate: false` and no `## Mock-equip scenarios` get LOUD WARNING + escalations.md note. v4.1 (planned): same condition becomes `gate-failed-terminal: doc-only-without-mock-equip`.

**Real-run evidence**: run 2026-05-13-1958 used mock-equip on viral-gtm-ai-startup SKILL.md champion. 3 scenarios (abel-mcp / Decision Token vertical / abel-graph-computer). 0 BLOCKER, 0 MAJOR, 4 MINOR. All 8 mechanical pass criteria satisfied. v4.0 formalizes that ad-hoc protocol into rule #9 doc-only path.

**Backwards compatibility**: code-task and other executable-task targets unchanged (still require real execution). Doc-only programs without mock-equip get warning in v4.0, hard-fail in v4.1.

## v3.0.0 - Unified loop (origin)

One skill, one loop, one discipline. 18 INVARIANTS. Mechanism-converge termination. Goal-authoring as stage of unified loop.
