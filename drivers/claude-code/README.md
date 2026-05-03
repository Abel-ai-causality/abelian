# Abelian — Claude Code driver

Native skill path. Claude reads `SKILL.md` + `INVARIANTS.md` and orchestrates the loop via `Agent` tool dispatches.

## Install

```
/plugin marketplace add Abel-ai-causality/abelian
/plugin install abelian@abelian
```

Or `git clone https://github.com/Abel-ai-causality/abelian.git ~/.claude/skills/abelian` then restart Claude Code.

## Invocation

```
/abelian program.md
```

| Flag | Effect |
|---|---|
| `--mode=co-research` (default) | two Claude peers, different context-framing, propose+attack each round |
| `--mode=unilateral` | single mutator + dissect adversary (1× cost) |
| `--adversary=codex` | cross-family priors via `Bash` → `codex exec` subprocess (loud-degrades to `dissect` if unavailable) |
| `--code-review=on` | rule #12 supplemental gate via `codex review --uncommitted` |

Abort: Ctrl+C → `status=interrupted`.

Adversary dispatch: `Agent(general-purpose) + Skill('dissect')`. Spec: [`../../SKILL.md`](../../SKILL.md) + [`../../INVARIANTS.md`](../../INVARIANTS.md).
