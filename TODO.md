# Abelian TODO

Tracked at the protocol level. Issues filed on GitHub mostly track items here.

## Open

### Smoketest of codex CLI primary path

The Claude Code path was smoketested 2026-04-28 (count_duplicate_pairs
campaign, full v2.8 protocol exercised). The codex CLI path **invocation
form** (cat SKILL.md INVARIANTS.md prompts/dissect.md → codex exec) has
not been exercised against a real codex CLI session. First user should
verify:

- codex `-s workspace-write` sandbox flag is correct for your codex CLI
  version (`codex exec --help | grep -A 2 sandbox`)
- codex's prompt budget is sufficient for SKILL.md + INVARIANTS.md +
  prompts/dissect.md + state.json (with accumulated rounds) + git diff
  in a single inlined prompt. v2.10 estimate: ~40K tokens at run start,
  growing ~2-5K per round. If codex CLI version caps below ~80K
  effective input, recommend stripping comments from SKILL.md or
  passing only recent N rounds in state.json.
- codex's adversary dispatch via fresh `codex exec` subprocess works as
  expected (isolated context, returns verdict, writes adversary.txt
  with header)

### Self-test artifacts in repo

Currently no `examples/` directory. Add a minimal smoketest target
(e.g., the count_duplicate_pairs campaign from 2026-04-28) so users can
copy-paste verify before running on their own codebase. Useful for
both drivers.

### Cross-family adversary reference wrapper

Documented as a sketch in `drivers/codex-cli/README.md`. If a real
team adopts abelian via codex CLI and needs cross-RLHF-family priors,
ship a working `cross-family-adversary.py` (anthropic SDK + tool-use
loop preserving the nonce-defense — main session does NOT write
adversary.txt directly; Claude does via tool use). ~80 lines Python.
Defer until concrete demand.

### Co-research diversity-collapse detection

INVARIANTS rule #6 says co-research plateau requires "candidate
edit-distance falling between peer mutations" alongside no eval
improvement. The mechanism for measuring edit distance (Levenshtein
on the diff strings? AST distance? semantic distance?) is not
specified. Current default: orchestrator may interpret loosely
("two peers proposing essentially the same change"). v2.11 may
specify a concrete metric.

### Examples / case studies

A few worked-through examples (speedup / refactor / API audit) in
`examples/` would help adoption. Currently the only documented
campaign is the smoketest commit history.

## Won't fix (decisions, not bugs)

- **No `--push` or `--pr` support.** Users review `git log BASE..HEAD`
  and decide what ships. Mirrors night-shift.
- **No daemon / continuous mode.** Abelian is bounded by mechanism
  convergence. For overnight autonomy use night-shift; for continuous
  iteration use ralph-loop.
- **No multi-target campaigns in one run.** One `program.md` = one
  Target = one campaign. Run multiple campaigns for multiple targets.
- **No bash wrapper script** (v2.10 removed). Both drivers are LLM
  agent harnesses orchestrating SKILL.md directly. Adding a shell
  layer is redundant.
