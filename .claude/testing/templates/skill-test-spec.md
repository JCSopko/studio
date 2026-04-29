# Skill Test Spec — `/<skill-name>`

Behavioral spec for the `<skill-name>` skill. Used by `/skill-test spec <skill-name>` to evaluate whether the skill's instructions, when followed, satisfy the assertions below.

## Identity

- **Skill**: `<skill-name>`
- **Skill file**: `.claude/skills/<skill-name>/SKILL.md`
- **Category**: `<gate | review | authoring | readiness | pipeline | analysis | team | sprint | utility>`
- **Priority**: `<critical | high | medium | low>`
- **Author**: `<your name or agent>`
- **Date**: `YYYY-MM-DD`

## Test Cases

### Case 1: Happy Path — `<short name>`

**Fixture** (assumed state of project files at invocation):

- `<filepath>` exists and contains `<expected content>`.
- `<filepath>` does NOT exist.
- (etc.)

**Expected behavior** (high-level steps the skill should take):

1. Read the relevant input file(s).
2. Compose output in the correct format.
3. Ask before writing.
4. Write to the correct path on approval.
5. Surface the recommended next step.

**Assertions** (each evaluated PASS / PARTIAL / FAIL):

- [ ] Skill reads `<expected input>` before producing output.
- [ ] Skill output includes the verdict keyword `<keyword>` where applicable.
- [ ] Skill asks "May I write this to `<filepath>`?" before any Write/Edit.
- [ ] Skill writes to `<filepath>` matching the expected format.
- [ ] Skill surfaces the next step (e.g., "Recommended: run `/<next-skill>`").

### Case 2: Edge Case — `<short name>`

**Fixture**:

- `<filepath>` is missing or invalid.
- (etc.)

**Expected behavior**:

1. Skill detects the missing/invalid input.
2. Skill surfaces a clear error or remediation suggestion.
3. Skill does NOT proceed to write.

**Assertions**:

- [ ] Skill detects the invalid state.
- [ ] Skill surfaces a remediation suggestion (e.g., "Run `/<prerequisite-skill>` first").
- [ ] Skill does NOT write any files in this fixture.

### Case 3: Mode Variation — `<mode name>`

(For skills with multiple modes — e.g., `/<skill-name> static` vs `/<skill-name> spec`. Add one Case per mode; skip if the skill has only one mode.)

**Fixture**: arguments invoke the named mode.

**Expected behavior**: skill executes the mode's documented behavior.

**Assertions**:

- [ ] Skill correctly parses the mode argument.
- [ ] Skill executes the mode-specific path.

## Protocol Compliance Assertions

These apply to every skill regardless of category:

- [ ] Skill's frontmatter includes `name`, `description`, `argument-hint`, `user-invocable`, `allowed-tools`.
- [ ] Skill body has ≥2 numbered phases or `##`-level sections.
- [ ] Skill includes a verdict keyword somewhere (`PASS` / `FAIL` / `CONCERNS` / `BLOCKED` / etc., as appropriate).
- [ ] Skill uses ask-before-write language (`"May I write..."` or equivalent) before any Write/Edit usage.
- [ ] Skill ends with a recommended next step or follow-up section.

## Notes

- Specs describe **current behavior**, not ideal behavior. They are written by reading the skill, so they may encode bugs. When a skill misbehaves in practice, correct the skill first, then update the spec to match.
- Treat spec failures as "this needs investigation," not "the skill is definitively wrong."
