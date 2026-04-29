# Studio Skill & Agent Testing Framework (minimal rebuild)

This directory replaces the upstream `CCGS Skill Testing Framework/` that ships in the original Donchitos/Claude-Code-Game-Studios template. The CCGS framework was deleted in Studio's Phase 2 distillation pass because its 121 specs referenced the 49-agent CCGS hierarchy that Studio collapsed into 6 disciplines + 5 UE specialists.

This rebuild is intentionally minimal — registry + rubric + spec templates. Behavioral specs are authored on demand; coverage tracks in `catalog.yaml`.

## Files

| File | Purpose |
|---|---|
| `catalog.yaml` | Master registry for all agents and skills with category, spec path, and last-test tracking. |
| `quality-rubric.md` | Category-specific pass/fail metrics for `/skill-test category`. |
| `templates/agent-test-spec.md` | Template for writing new agent behavioral specs. |
| `templates/skill-test-spec.md` | Template for writing new skill behavioral specs. |
| `agents/<category>/<name>.md` | Behavioral spec for an agent. Author on demand. |
| `skills/<category>/<name>.md` | Behavioral spec for a skill. Author on demand. |
| `results/` | `/skill-test spec` writes results here. Gitignored. |

## How to use

```bash
# Static linter — works for any skill, no spec needed
/skill-test static <skill-name>
/skill-test static all

# Behavioral spec test — needs a spec at the catalog's `spec:` path for the skill
/skill-test spec <skill-name>

# Category rubric check — needs the skill's `category:` set in catalog
/skill-test category <skill-name>
/skill-test category all

# Coverage audit — works against catalog
/skill-test audit

# Improvement loop on a single skill (uses static + category baselines)
/skill-improve <skill-name>
```

## How to author a new spec

1. Pick the skill or agent to spec.
2. Copy `templates/skill-test-spec.md` (for skills) or `templates/agent-test-spec.md` (for agents).
3. Save at the spec path listed in `catalog.yaml` (e.g., `agents/discipline/programming.md`).
4. Update `catalog.yaml` if the spec path differs from the default.
5. Run `/skill-test spec <name>` to validate.

## Why this exists in Studio (not upstream CCGS form)

- **Scope match**: Studio has 11 agents (6 disciplines + 5 UE specialists), not 49. The catalog reflects Studio's actual roster.
- **Path discipline**: Lives at `.claude/testing/` rather than the top-level `CCGS Skill Testing Framework/` directory — keeps the harness layer at one tree.
- **Optional**: Indie devs adopting Studio don't need this folder to use the agents and skills. It's QA tooling for skill/agent authors. Delete it (`rm -rf .claude/testing`) and `/skill-test static` continues to work; the other modes degrade gracefully reporting "catalog.yaml missing."

## Maintenance

- When a new skill is added to `.claude/skills/`, add it to `catalog.yaml`'s `skills:` map with category and priority.
- When a new agent is added to `.claude/agents/`, add it to `catalog.yaml`'s `agents:` map.
- When a category-rubric metric proves insufficient or a new failure class recurs, update `quality-rubric.md`.
