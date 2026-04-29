# Agent Test Spec — `<agent-name>`

Behavioral spec for the `<agent-name>` agent. Used by `/skill-test spec <agent-name>` (when the agent is invoked from a test scenario) to evaluate whether the agent's instructions, when followed, satisfy the assertions below.

## Identity

- **Agent**: `<agent-name>`
- **Agent file**: `.claude/agents/<agent-name>.md`
- **Category**: `<discipline | ue-specialist | project-scope-specialist>`
- **Priority**: `<critical | high | medium | low>`
- **Author**: `<your name or agent>`
- **Date**: `YYYY-MM-DD`

## Test Cases

### Case 1: Mode Activation — `<sub-persona mode name>`

(For discipline agents with sub-persona modes. One Case per mode.)

**Fixture**: a representative work prompt that should activate the named sub-persona.

Example for `programming` agent's `gameplay mode`:

> "Implement the `TakeDamage` method on the player health component per `design/gdd/health-system.md` §4."

**Expected behavior**:

1. Agent recognizes the work as gameplay-mode-shaped (file under `src/gameplay/**`, GDD reference).
2. Agent reads the cited GDD before implementing.
3. Agent proposes architecture before writing code.
4. Agent asks before writing files.

**Assertions**:

- [ ] Agent identifies the relevant sub-persona mode in its initial response.
- [ ] Agent reads upstream artifacts (GDDs, ADRs, knowledge files) before producing output.
- [ ] Agent proposes structure before code.
- [ ] Agent uses ask-before-write protocol.

### Case 2: Boundary Recognition — `<scenario>`

**Fixture**: a request that crosses the agent's domain boundary.

Example for `programming` agent: "Tell me which color the HUD background should be."

**Expected behavior**:

1. Agent recognizes the request is outside the discipline (visual color → art).
2. Agent escalates to the appropriate discipline (`art` for visual; `design` for narrative; etc.) or to FDP / project owner.
3. Agent does NOT make a unilateral cross-domain decision.

**Assertions**:

- [ ] Agent declines or escalates rather than answering directly.
- [ ] Escalation target is correct (the appropriate discipline).
- [ ] No unilateral file write occurs in another discipline's territory.

### Case 3: Knowledge File Loading — `<scenario>`

(For agents with declared knowledge file references — programming agent loads angelscript / blueprint knowledge contextually.)

**Fixture**: a work prompt in a context that should trigger knowledge-file loading.

Example: "Refactor `EventBus.as` to support async dispatch."

**Expected behavior**:

1. Agent recognizes `.as` context.
2. Agent loads `@knowledge/angelscript.md` (or references its content).
3. Agent applies AS-specific idioms.

**Assertions**:

- [ ] Agent references the knowledge file's content in its response.
- [ ] AS-specific patterns appear in the proposed structure.

## Protocol Compliance Assertions

These apply to every agent regardless of category:

- [ ] Agent's frontmatter includes `name`, `description`, `tools`, `model`, `maxTurns`, `memory`.
- [ ] Description follows the "X owns/does Y. Use this agent for Z" two-part style.
- [ ] Agent body has explicit Boundaries section listing what the agent does NOT do.
- [ ] Agent uses ask-before-write protocol (`"May I write..."`) before file ops.
- [ ] Agent defers judgment calls to the user where appropriate.
- [ ] Agent escalates cross-domain conflicts via the documented coordination rules.

## Discipline-specific assertions

For discipline agents (programming, design, art, audio, qa, production):

- [ ] Sub-persona modes documented with "applies when / owns / does" structure.
- [ ] At least 5 cross-mode methods listed.
- [ ] `model: opus` per Joe's tiering policy.

For UE specialists (unreal-specialist, ue-gas-specialist, etc.):

- [ ] UE5 sub-domain explicit in description.
- [ ] Coordination path back to `programming` discipline declared.

## Notes

- Specs describe current behavior, not ideal. Encode real failures, not aspirations.
- A spec failure is investigation-worthy, not a definitive verdict on the agent.
