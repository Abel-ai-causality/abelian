# Abelian TODO

Tracked at the protocol level. Issues filed on GitHub mostly track items here.

## Open

### Smoketest of `--code-review=on` path (rule #12, v2.11)

`codex review --uncommitted` verified functional locally 2026-04-28 (via
`bun /home/bht/.npm-global/bin/codex review --help`; node not in PATH but
bun shim works per `~/.bashrc`). Not yet exercised end-to-end in an
abelian campaign — first user enabling `--code-review=on` will be the
smoketest. Specific points to verify:

- codex review respects `~/.codex/config.toml` defaults (model selection,
  sandbox mode) — orchestrator passes `-c 'model_reasoning_effort="high"'`
  override
- `codex-review.txt` output format: confirm `[P1]/[P2]/[P3]` markers
  follow expected pattern; if codex's output schema diverges, gate
  check needs adjustment
- Loop-until-clean semantics: night-shift uses fix → re-review → max
  10 rounds before revert. Abelian's current default is single-pass
  (P1/P2 → revert immediately, let mutator propose differently next
  round). May want to add `--code-review-max-rounds=N` knob if real
  campaigns show benefit from in-round iteration.
- bun shim ergonomics: when `node` not in PATH, orchestrator must use
  `bun codex review ...`. Document this clearly in dispatch path.

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

## v2.13 → future: NS-borrowable backlog (Stephen 2026-04-29 audit)

After v2.13 reframe (abelian = adversarial collaboration framework), 6 NS
features remain considered for borrow. Priority by abelian-fit:

**High priority**:
- **Plan-first ratchet** (NS Inner 1) —落盘 `round-N/plan.md` (files/approach/test-strategy/risks) before mutate; commit-gate adds 9th check (plan.md non-empty + Target files referenced). Prevents "想不清就开干". Earlier proposed (peer-A Vector A) but not shipped.
- **Codex review loop-until-clean** (NS Inner 3) — rule #12 升级: P1/P2 found → fix → re-review → max 10 rounds → revert. Single-pass current form misses NS's true effect source.

**Medium priority**:
- **Eval evidence completeness check** (NS Inner 4 reframed) — embed in step 3 Eval rather than separate step (peer-B F2.3 self-attack 学到): require deliverable existence + UI screenshot for UI tasks.
- **Subagent delegation rules** — co-research peer dispatch prompts MUST include "you may NOT git add/commit, you may NOT modify outside Target". Currently implicit.
- **Mid-run direction propose + adversary review** (NS #2/#3) — Strategy in program.md is pre-fixed; long-horizon innovative task may benefit from "round N: should we revise direction?" with adversary review.

**Low priority** (defer until empirical need surfaces):
- **Stop / Resume / Abandon** — only relevant if abelian truly runs 4h+ long-horizon campaign and gets interrupted.
- **"Never push to remote"** — explicit safety rule for autonomous loop. Quick add (1 line in INVARIANTS or Safety Rules section).

## v2.13 → future: abelian-specific gaps surfaced by dry-run (Stephen 2026-04-29) — RESOLVED in v2.14

Dry-run of abelian co-research on a doc-task (1-page abelian-vs-night-shift
selector) surfaced 3 fundamental gaps. All 3 resolved in v2.14 (this commit)
after peer-B R1+R2 cross-attack rounds (`/tmp/peer-b-attack.md`,
`/tmp/peer-b-attack-r2.md`).

- ~~**Schema-grounding for fuzzy ground sources** (rule #8 extension)~~ — resolved: INVARIANTS rule #8 expanded with **Fuzzy-ground self-judge** subsection. New `Eval ground:` declaration in program.md (≥1 of (b)/(c)/(d); option (a) self-ground supplementary only). Quote-grep gate replaces vibes-grounding; untraced dimensions auto-scored 0; contradictions flag round `fuzzy-ground-violation` and revert.
- ~~**Doc-task cross-attack quality**~~ — resolved: SKILL.md Co-Research Mode gains **Doc-task cross-attack** subsection. Five-criterion attack form with mandatory falsification statement (`"this is wrong if X, because the doc claims Y"`, X must be grep-able / runnable / countable; aesthetic / tone / rigor X auto-rejected). After 2 re-spawn failures, escalates to dispatched-single-adversary mode with `state.rounds[N].coresearch_degraded: true` provenance.
- ~~**Attack-class library by domain**~~ — resolved: SKILL.md Attack Class Checklist gains **Attack Class Library** subsection. 4 named libraries (research-class R1–R6; audit-class A1–A4; decision-class D1–D4; doc-class C1–C4) plus existing code-domain extensions. Namespace discipline reserves `*-class` suffix for libraries. New `task:` field in program.md drives mandatory-library check; loud-warn on absent field (defaults to `task: code` for v2.5 backwards-compat).

Verdict trail: v2.14 draft v1 (peer-B R1: MAJOR-REVISION-REQUIRED, 4
BLOCKERS / 4 MAJORS / 3 MINORS) → draft v2 (peer-B R2: ACCEPT-WITH-FIXES,
1 NEW BLOCKER / 2 NEW MAJORS / 2 NEW MINORS) → v2.14.0 commit (NEW
BLOCKER 1 resolved via loud-warn pattern; NEW MAJOR 1 via grep-able-X
requirement; NEW MAJOR 2 via `coresearch_degraded` schema field).

Positioning preserved: `adversarial collaboration on deep + innovative +
long-horizon iteration with tractable doc + testable metric`. Tasks with
no testable metric remain out of scope (use ce-brainstorm).

