# Changelog

## v3.2.0 - Round-0 pre-flight verification (planned)

Adds subsection A0 to rule #16 Program Contract Gate. Round-0 hardens before program-peer-challenge dispatch.

- **A0.1 Cite verification** - for each public-frame URL/ISBN citation in program.md, 30-sec resolve check with 1 retry. Failure -> `gate-failed-terminal: citation-unresolvable`. Human-override via `cite-override:` field requires reason + peer-attack in step 3.
- **A0.2 Probe arithmetic consistency** - for each probe-derived rubric threshold (`>=N = X% of M`, `all K`, distinct counts), execute the cited probe and confirm. Mismatch -> `gate-failed-terminal: probe-arithmetic-mismatch`.
- **A0.3 Artifact** - `$RUN_DIR/round-0/preflight-verification.txt` mandatory record.
- Rule #16 D extended: program-peer-challenge MUST execute cited probes, not just inspect text.

**Real-run evidence**: run 2026-05-13-1832 caught fabricated First Round article (~50K tokens via v1 peer-challenge); A0.1 would have caught at <30 sec. Run 2026-05-13-1958 spent 4 of 5 round-0 iterations on arithmetic mismatches A0.2 catches in seconds.

**Backwards compatibility**: programs with no public citations and no arithmetic probes pass unchanged. Programs with fabricated authority or stale count math now fail-fast before peer dispatch.

## v3.0.0 - Unified loop (origin)

One skill, one loop, one discipline. 18 INVARIANTS. Mechanism-converge termination. Goal-authoring as stage of unified loop.
