# Studio — Discipline-Collapsed Game Studio Agent Architecture

Indie game development managed through 6 discipline-collapsed Claude Code agents plus optional engine-stack specialists. Each discipline owns a domain end-to-end with sub-persona modes; engine specialists branch off when project complexity warrants.

> Studio is a fork of `Donchitos/Claude-Code-Game-Studios`, distilled into a discipline-collapsed structure. Original CCGS shipped 49 agents across a director/lead/specialist hierarchy. Studio collapses those into 6 discipline agents (`programming`, `design`, `art`, `audio`, `qa`, `production`) plus 5 UE specialists retained for engine-deep work. New specialists can be branched on demand via `/branch-specialist`.

## Technology Stack

- **Engine**: [CHOOSE: Unreal Engine 5 / UE5 + Hazelight Angelscript]
- **Language**: [CHOOSE: C++ / Blueprint / Angelscript / mix]
- **Version Control**: Git with trunk-based development
- **Build System**: [SPECIFY after choosing engine]
- **Asset Pipeline**: [SPECIFY after choosing engine]

> **Note**: Studio targets UE5. Godot and Unity tracks were removed from the upstream CCGS template. UE specialists ship in this repo for stack-deep work.

## Discipline Agents (Studio core)

| Discipline | Owns | Sub-persona modes |
|---|---|---|
| **`programming`** | All code, infrastructure, ADRs | Tech-lead, Gameplay, Engine, AI, Network, Tools, UI, DevOps, Security, Performance |
| **`design`** | GDDs, formulas, levels, economy, narrative, lore, live-ops | Game-design, Systems, Level, Economy, Narrative-direction, Writing, World-building, Live-ops |
| **`art`** | Visual identity, technical art, UX | Art-direction, Technical-art, UX |
| **`audio`** | Sonic identity, sound design | Audio-direction, Sound-design |
| **`qa`** | Quality verification, test strategy, accessibility | QA-lead, QA-tester, Accessibility |
| **`production`** | Sprint flow, prototyping, analytics, community, release, localization | Production, Prototyping, Analytics, Community, Release, Localization |

Each agent contains the merged knowledge of its CCGS sub-roles; sub-persona modes activate based on the work in front of the agent.

## Engine Specialists (UE5 — retained as project-scope reference)

| Specialist | Scope |
|---|---|
| `unreal-specialist` | UE5 lead — Blueprint vs C++ decisions, UE subsystems, packaging |
| `ue-gas-specialist` | Gameplay Ability System — abilities, effects, attributes, tags, prediction |
| `ue-blueprint-specialist` | Blueprint visual scripting — BP/C++ boundary, graph standards, optimization |
| `ue-replication-specialist` | Networking — replication, RPCs, prediction, bandwidth |
| `ue-umg-specialist` | UI — UMG, CommonUI, widget hierarchy, data binding |

These remain because UE5 is the supported engine and the specialists encode UE-specific authority that doesn't generalize. Projects using vanilla UE5 use these directly. Projects using Hazelight Angelscript should use `/branch-specialist` to create a `ue-as-specialist` (project-scope) that loads `@knowledge/angelscript.md`.

## Knowledge Files

Specialty knowledge ships as reference files the discipline agents and path-scoped rules load contextually:

- **`@knowledge/angelscript.md`** — Hazelight Angelscript idioms, AS↔UE bindings, hot-reload behavior. Auto-loaded by the `angelscript-code` rule when editing `.as` files.
- **`@knowledge/blueprint.md`** — Blueprint best practices, BP↔C++ boundary judgment, performance, asset references. Auto-loaded by the `blueprint-code` rule when working in BP context.

Additional knowledge files (replication, GAS, Niagara, performance-profiling, etc.) author on demand when the project shows the need.

## On-Demand Specialist Creation

When project complexity surfaces a need for deeper specialization than the discipline agent's general knowledge, use `/branch-specialist [name]` to create a project-scope specialist that branches off one of the six disciplines. The skill walks naming, parent-discipline selection, scope, knowledge-file wiring, and writes the new agent file. Specialists shadow their parent discipline's bare invocation only for their declared scope.

## Project Structure

@.claude/docs/directory-structure.md

## Engine Version Reference

@docs/engine-reference/unreal/VERSION.md

## Technical Preferences

@.claude/docs/technical-preferences.md

## Coordination Rules

@.claude/docs/coordination-rules.md

## Collaboration Protocol

**User-driven collaboration, not autonomous execution.**
Every task follows: **Question → Options → Decision → Draft → Approval**

- Agents MUST ask "May I write this to [filepath]?" before using Write/Edit tools
- Agents MUST show drafts or summaries before requesting approval
- Multi-file changes require explicit approval for the full changeset
- No commits without user instruction

See `docs/COLLABORATIVE-DESIGN-PRINCIPLE.md` for full protocol and examples.

> **First session?** If the project has no engine configured and no game concept, run `/start` to begin the guided onboarding flow.

## Coding Standards

@.claude/docs/coding-standards.md

## Context Management

@.claude/docs/context-management.md

## Path-Scoped Rules

The `.claude/rules/` directory holds path-scoped rules that auto-fire when files matching their `paths:` glob are touched. Active rules:

- `ai-code.md` — `src/ai/**`
- `data-files.md` — `assets/data/**`
- `design-docs.md` — `design/gdd/**`
- `engine-code.md` — `src/core/**`
- `gameplay-code.md` — `src/gameplay/**`
- `narrative.md` — `design/narrative/**`
- `network-code.md` — `src/networking/**`
- `prototype-code.md` — `prototypes/**`
- `shader-code.md` — `assets/shaders/**`
- `test-standards.md` — `tests/**`
- `ui-code.md` — `src/ui/**`
- `angelscript-code.md` — `**/*.as` and `**/Script/**` (loads `@knowledge/angelscript.md`)
- `blueprint-code.md` — `**/Content/**/*.uasset` and `**/Blueprints/**` (loads `@knowledge/blueprint.md`)
