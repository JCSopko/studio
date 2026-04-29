# Studio — Skill & Agent Quality Rubric

Category-specific pass/fail metrics evaluated by `/skill-test category <name>`. Each rubric describes how a skill in that category should behave. The skill's text is checked against the rubric metrics; failures cite the exact gap.

## Skill rubrics

### gate

For phase-gate skills (`/gate-check`).

| Metric | Pass requires |
|---|---|
| G1 — Review mode read | Skill reads `production/review-mode.txt` (full / lean / solo) before deciding which gates fire. |
| G2 — Full mode discipline panel | In full mode, skill spawns gate verdicts from the relevant disciplines (programming, design, qa as applicable to the phase). |
| G3 — Lean mode: phase-gate only | In lean mode, only the phase-level GATE verdict fires, not per-discipline reviews. |
| G4 — Solo mode: no gates | In solo mode, the skill skips all gates, marks completion, and surfaces the override to the user. |
| G5 — No auto-advance | The skill never auto-advances the project to the next phase; the user always chooses. |

### review

For review skills (`/design-review`, `/architecture-review`, `/review-all-gdds`, `/code-review`).

| Metric | Pass requires |
|---|---|
| R1 — Read-only | Skill does not write files; it reports findings only. |
| R2 — Section-by-section | For document reviews, the skill walks each required section and reports per-section. |
| R3 — Verdict keyword | Output includes `PASS` / `CONCERNS` / `MAJOR REVISION` / equivalent verdict keyword. |
| R4 — Specific citations | Findings cite specific sections, files, or line numbers — not generalities. |

### authoring

For authoring skills (`/design-system`, `/quick-design`, `/architecture-decision`, `/art-bible`, `/create-architecture`, `/branch-specialist`, etc.).

| Metric | Pass requires |
|---|---|
| A1 — Question-first | Skill asks clarifying questions before authoring. |
| A2 — Section-by-section May-I-write | For long documents, skill writes section-by-section with approval gates per section, not as one final blob. |
| A3 — Skeleton-first | Skill creates a skeleton (headers + bullets) before prose. |
| A4 — Defers final calls to user | Skill recommends but never decides for the user. |

### readiness

For readiness skills (`/story-readiness`, `/story-done`).

| Metric | Pass requires |
|---|---|
| RD1 — Blockers surfaced | Skill identifies blocking dependencies and reports them explicitly. |
| RD2 — Discipline gate in full mode | In full review mode, relevant disciplines sign off (programming for code, qa for evidence, etc.). |
| RD3 — Test-evidence routing | For `/story-done`, evidence requirements are surfaced per the BLOCKING/ADVISORY routing in `qa.md`. |

### pipeline

For pipeline skills (`/create-epics`, `/create-stories`, `/dev-story`, `/map-systems`, `/qa-plan`, `/release-checklist`, etc.).

| Metric | Pass requires |
|---|---|
| P1 — Upstream dependency check | Skill verifies upstream artifacts exist (e.g., `/create-stories` checks epics; `/dev-story` checks stories). |
| P2 — Handoff path clear | Skill states what comes next and which discipline owns the next step. |
| P3 — Artifact format | Outputs match the project's expected structure (filename pattern, frontmatter, sections). |

### analysis

For analysis skills (`/consistency-check`, `/balance-check`, `/scope-check`, `/perf-profile`, `/asset-audit`, `/content-audit`).

| Metric | Pass requires |
|---|---|
| AN1 — Read-only | Skill does not modify files. |
| AN2 — Verdict keyword | Output includes a verdict (`COMPLIANT` / `DRIFT-FOUND` / `BLOCKING` / etc.). |
| AN3 — Actionable findings | Each finding names a file path and a specific recommendation. |

### team

For team-orchestration skills (`/team-combat`, `/team-narrative`, etc.).

| Metric | Pass requires |
|---|---|
| T1 — Required disciplines spawned | All disciplines required for the team task are invoked (e.g., `/team-combat` spawns programming + design + audio + qa). |
| T2 — Independent agents in parallel | Disciplines whose inputs are independent are spawned in parallel `Task` calls. |
| T3 — BLOCKED surfaced | If any discipline reports BLOCKED, the skill surfaces it immediately and does not silently skip. |
| T4 — Synthesis returned | Final output synthesizes per-discipline findings into a single coherent report. |

### sprint

For sprint skills (`/sprint-plan`, `/sprint-status`, `/milestone-review`, `/retrospective`, `/changelog`, `/patch-notes`).

| Metric | Pass requires |
|---|---|
| SP1 — Reads sprint data | Skill reads `production/sprints/` or equivalent before reporting/planning. |
| SP2 — Status keywords | Output uses the project's status keywords (e.g., NOT-STARTED / IN-PROGRESS / BLOCKED / DONE). |
| SP3 — Honest about scope | If sprint capacity vs scope is at risk, skill surfaces it; never papered over. |

### utility

For utility skills (`/help`, `/start`, `/setup-engine`, `/onboard`, `/skill-test`, `/skill-improve`, `/smoke-check`, `/bug-report`, etc.).

| Metric | Pass requires |
|---|---|
| U1 — Static checks pass | Skill passes the 7-check static linter (`/skill-test static <name>`). |
| U2 — Mode handling | If the skill has multiple modes, mode parsing is documented and matches `argument-hint`. |

## Agent rubrics

### discipline (the 6 Studio discipline agents)

| Metric | Pass requires |
|---|---|
| AD1 — Sub-persona modes documented | Agent body lists each sub-persona mode with "applies when / owns / does" structure. |
| AD2 — Cross-mode methods | Agent body lists at least 5 methods that apply across all sub-persona modes. |
| AD3 — Boundaries listed | Agent body has an explicit "Boundaries" section listing what the discipline does NOT do. |
| AD4 — Frontmatter conventions | Frontmatter has `name`, `description`, `tools`, `model`, `maxTurns`, `memory`. Description follows the "X owns/does Y. Use this agent for Z" two-part style. |
| AD5 — Opus default | `model: opus` (per Joe's tiering policy). |

### ue-specialist (the 5 retained UE specialists)

| Metric | Pass requires |
|---|---|
| US1 — UE5-specific scope | Agent body declares its UE5 sub-domain (GAS, Blueprint, Replication, UMG, or top-of-stack). |
| US2 — Coordination path | Agent body declares how it relates to Studio's `programming` discipline (e.g., "branches off programming for UE-specific work"). |
| US3 — Frontmatter conventions | Same as AD4. |

## Maintenance

This rubric is the testable contract for skills and agents in Studio. Update when:
- A new category of skill is added.
- A discipline agent's structural conventions change.
- A specific failure pattern recurs and warrants a new metric.

`/skill-improve` reports failures against this rubric; closing a failure means either the skill changes to satisfy the metric, or the metric is revised to reflect a justified divergence.
