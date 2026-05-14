# Changelog

## v3.2.0 - Round-0 pre-flight verification (planned)

Adds subsection A0 to rule #16 Program Contract Gate. Round-0 hardens before program-peer-challenge dispatch.

- **A0.1 Cite verification** - for each public-frame URL/ISBN citation in program.md, 30-sec resolve check with 1 retry. Failure -> `gate-failed-terminal: citation-unresolvable`. Human-override via `cite-override:` field requires reason + peer-attack in step 3.
- **A0.2 Probe arithmetic consistency** - for each probe-derived rubric threshold (`>=N = X% of M`, `all K`, distinct counts), execute the cited probe and confirm. Mismatch -> `gate-failed-terminal: probe-arithmetic-mismatch`.
- **A0.3 Artifact** - `$RUN_DIR/round-0/preflight-verification.txt` mandatory record.
- Rule #16 D extended: program-peer-challenge MUST execute cited probes, not just inspect text.

**Real-run evidence**: run 2026-05-13-1832 caught fabricated First Round article (~50K tokens via v1 peer-challenge); A0.1 would have caught at <30 sec. Run 2026-05-13-1958 spent 4 of 5 round-0 iterations on arithmetic mismatches A0.2 catches in seconds.

**Backwards compatibility**: programs with no public citations and no arithmetic probes pass unchanged. Programs with fabricated authority or stale count math now fail-fast before peer dispatch.

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
