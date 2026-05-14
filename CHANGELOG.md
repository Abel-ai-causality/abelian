# Changelog

## v3.1.0 - Chained-run dependency gate (planned)

Adds new rule #19 Chained-run dependency gate. Run-to-run state passing is hard-verified.

- New optional program.md field: `Depends on: <RUN_ID>`. When present, rule #19 verifies prior run completed, top-level canonical `champion.artifact_path` exists, resolved path is non-empty + inside prior run dir.
- New state.json top-level block: `prior_run_dependency = {depends_on, verified_at, champion_path_resolved, champion_artifact_sha256}`. SHA-256 captured at verification time.
- Canonical champion schema: ONE top-level `champion` object with `{peer, artifact_path, artifact_kind, conservative_score, verified_at}`. Aliased fields allowed but `artifact_path` is canonical.
- Mission Thread (rule #14) extension: `prior_run.champion` is a valid grounding source class with built-in provenance.
- Auto-Compound: optional `Chained from: <RUN_ID> / <path> / <sha256>` header.
- Migration: archived state.json files with duplicate-key bug use heuristic (first top-level champion + aliased-field fallback) at resolution time. New v3.1 runs MUST write single canonical champion only.

**Real-run evidence**: viral-gtm-ai-startup build chain (run 2026-05-13-1832 produced refined-proposal champion → run 2026-05-13-1958 built skill.md from it) used ad-hoc `prior_run_dependency` field. Codex audit found archived state.json files have duplicate top-level `champion` keys; `jq .champion` returns null. v3.1 formalizes the chain + fixes the schema bug.

**Backwards compatibility**: programs without `Depends on:` work unchanged. Archived runs use migration heuristic only when consumed as a dependency. v3.1 writes new state files in canonical schema.

## v3.0.0 - Unified loop (origin)

One skill, one loop, one discipline. 18 INVARIANTS. Mechanism-converge termination. Goal-authoring as stage of unified loop.
