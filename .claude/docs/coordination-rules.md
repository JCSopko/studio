# Agent Coordination Rules

Studio operates with 6 discipline agents (`programming`, `design`, `art`, `audio`, `qa`, `production`), 5 retained UE specialists (`unreal-specialist`, `ue-gas-specialist`, `ue-blueprint-specialist`, `ue-replication-specialist`, `ue-umg-specialist`), and project-scope specialists branched on demand via `/branch-specialist`.

## Coordination principles

1. **Discipline ownership**: Each discipline owns its files (programming → `src/**` etc.); cross-domain changes require coordination through the production discipline. See each discipline agent's "Boundaries" section for owned paths.
2. **Horizontal consultation**: Disciplines may consult each other but must not make binding decisions outside their domain. `programming` doesn't decide game design; `design` doesn't decide architecture.
3. **Conflict resolution**: When two disciplines disagree, escalate to the project owner (Joe / Jon) directly. Vision-level conflicts in projects with FDP at project scope route through FDP. Otherwise resolution is at the user level — there are no creative-director or technical-director agents in Studio.
4. **Change propagation**: When a change spans multiple disciplines (e.g., a design change affecting programming, art, and audio), the `production` discipline walks the dependency graph and surfaces the propagation list. The user makes the call on what propagates and when.
5. **No unilateral cross-domain changes**: A discipline must never modify files outside its designated paths without explicit user approval.
6. **Project-scope specialists shadow disciplines**: A specialist created via `/branch-specialist` (e.g., `ue-as-specialist` branching from `programming`) shadows the parent discipline's bare invocation only for the specialist's declared scope. Namespaced calls (`studio:programming`) always reach the discipline directly.

## Model Tier Assignment

Studio defaults all agents to Opus per Joe's tiering policy. Opus is the default for all work; Sonnet is opt-in only for explicitly mechanical sub-tasks (bulk frontmatter edits, routine code changes following an unambiguous spec, raw research gathering before synthesis). Haiku is reserved for trivial read-only operations.

| Tier | Model | When |
|---|---|---|
| **Opus** | `claude-opus-4-7` | All discipline agents, all judgment-heavy skill work, all synthesis. **Default.** |
| **Sonnet** | `claude-sonnet-4-6` | Opt-in only for: research gathering before synthesis, bulk file operations, index/search operations, routine code changes following an explicit unambiguous spec. |
| **Haiku** | `claude-haiku-4-5` | Reserved for trivial read-only status / formatting operations where no judgment is involved. |

If a task doesn't clearly fit the Sonnet allowlist, use Opus. When in doubt, Opus.

This is a departure from CCGS's defaults — CCGS spread models across agents (1 haiku, 9 sonnet, 1 opus in programming alone). Studio's Opus default reflects Joe's policy that quality of reasoning in user-visible work is always worth the cost.

## Cross-discipline workflows

The `production` discipline orchestrates these recurring patterns. See `agents/production.md` for the full pattern catalog. The most common:

- **New feature**: FDP / design vision check → design GDD → production schedule → programming impl → art / audio / qa as needed → production close.
- **Bug fix**: qa files report → qa triages → production schedules → programming fixes → programming reviews → qa verifies.
- **Balance adjustment**: production analytics → design eval → design + economy mode update → production close.
- **Sprint cycle**: all internal to production discipline.
- **Release pipeline**: production owns end-to-end (release mode + qa coordination).

## Subagents vs Agent Teams

Studio supports both single-session subagent spawning (via `Task` tool within a session) and agent teams (multiple coordinated sessions, opt-in).

### Subagents (default)

Spawned via `Task` within a single Claude Code session. Subagents share the session's permission context, run sequentially or in parallel within the session, and return results to the parent.

**When to spawn in parallel**: If two subagents' inputs are independent (neither needs the other's output to begin), spawn both `Task` calls simultaneously. Example: `/review-all-gdds` Phase 1 (consistency) and Phase 2 (design theory) are independent — spawn both at the same time.

### Agent Teams (experimental — opt-in)

Multiple independent Claude Code *sessions* running simultaneously, coordinated via a shared task list. Each session has its own context window and token budget. Requires `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` environment variable plus `--enable-teams --in-process` CLI flags.

**Use agent teams when**:
- Work spans multiple subsystems that will not touch the same files
- Each workstream would take >30 minutes and benefits from true parallelism
- A coordinator (production discipline) needs to coordinate 3+ specialist sessions working on different epics simultaneously

**Do not use agent teams when**:
- One session's output is required as input for another (use sequential subagents)
- The task fits in a single session's context (use subagents instead)
- Token budget is a concern — each team member burns tokens independently

## Parallel Task Protocol

When an orchestration skill spawns multiple independent agents:

1. Issue all independent `Task` calls before waiting for any result.
2. Collect all results before proceeding to dependent phases.
3. If any agent is BLOCKED, surface it immediately — do not silently skip.
4. Always produce a partial report if some agents complete and others block.
