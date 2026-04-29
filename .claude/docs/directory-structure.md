# Directory Structure

```text
/
├── CLAUDE.md                    # Master configuration
├── .claude/                     # Studio harness
│   ├── agents/                  # 6 disciplines + 5 UE specialists + project-scope branches
│   ├── skills/                  # Slash commands (incl. /branch-specialist)
│   ├── knowledge/               # Reference knowledge (angelscript, blueprint, ...)
│   ├── rules/                   # Path-scoped rules (auto-fire by glob match)
│   ├── hooks/                   # Session and tool hooks
│   ├── docs/                    # Coordination, standards, templates, workflow catalog
│   ├── agent-memory/            # Per-discipline persistent memory (project-scoped)
│   └── settings.json            # Permissions and hook registration
├── src/                         # Game source code
│   ├── core/                    # Engine internals (rules: engine-code.md)
│   ├── gameplay/                # Gameplay systems (rules: gameplay-code.md)
│   ├── ai/                      # AI systems (rules: ai-code.md)
│   ├── networking/              # Netcode (rules: network-code.md)
│   ├── ui/                      # UI plumbing (rules: ui-code.md)
│   └── tools/                   # Internal tooling
├── assets/                      # Game assets
│   ├── art/                     # Art assets
│   ├── audio/                   # Audio assets
│   ├── vfx/                     # Visual effects
│   ├── shaders/                 # Materials / shaders (rules: shader-code.md)
│   └── data/                    # Data-driven config (rules: data-files.md)
├── design/                      # Game design documents
│   ├── gdd/                     # System GDDs (rules: design-docs.md)
│   ├── narrative/               # Narrative architecture (rules: narrative.md)
│   ├── art/                     # Art bible, asset specs, reference imagery
│   ├── audio/                   # Sound bible
│   ├── ux/                      # UX specs, accessibility requirements
│   ├── levels/                  # Level designs
│   ├── economy/                 # Economy design
│   └── registry/                # Cross-system entity registries (entities.yaml)
├── docs/                        # Technical documentation
│   ├── architecture/            # ADRs and architecture docs
│   └── engine-reference/        # Curated engine API snapshots (version-pinned)
│       └── unreal/              # UE5 reference (Studio targets UE)
├── tests/                       # Test suites (rules: test-standards.md)
│   ├── unit/                    # Logic-type story evidence
│   ├── integration/             # Integration-type story evidence
│   ├── smoke/                   # Smoke check (qa-lead-owned)
│   └── regression/              # Regression suites
├── tools/                       # Build and pipeline tools
├── prototypes/                  # Throwaway prototypes (rules: prototype-code.md)
└── production/                  # Production management
    ├── sprints/                 # Sprint plans
    ├── milestones/              # Milestone definitions
    ├── risk-register/           # Risk tracking
    ├── analytics/               # Telemetry event taxonomy
    ├── community/               # Patch notes, dev blogs
    ├── localization/            # i18n config + locale tests
    ├── qa/                      # Bug reports, accessibility audits, evidence
    ├── release/                 # Release pipeline records
    ├── session-state/           # Ephemeral session state (active.md — gitignored)
    └── session-logs/            # Session audit trail (gitignored)
```

## Path-scoped rule registry

Rules under `.claude/rules/` auto-fire when files matching their `paths:` glob are touched:

- `ai-code.md` → `src/ai/**`
- `data-files.md` → `assets/data/**`
- `design-docs.md` → `design/gdd/**`
- `engine-code.md` → `src/core/**`
- `gameplay-code.md` → `src/gameplay/**`
- `narrative.md` → `design/narrative/**`
- `network-code.md` → `src/networking/**`
- `prototype-code.md` → `prototypes/**`
- `shader-code.md` → `assets/shaders/**`
- `test-standards.md` → `tests/**`
- `ui-code.md` → `src/ui/**`
- `angelscript-code.md` → `**/*.as` and `**/Script/**` (loads `@knowledge/angelscript.md`)
- `blueprint-code.md` → `**/Content/**/*.uasset` and `**/Blueprints/**` (loads `@knowledge/blueprint.md`)

## Knowledge file registry

Knowledge files under `.claude/knowledge/` are reference material loaded by discipline agents and path-scoped rules contextually:

- `angelscript.md` — Hazelight Angelscript idioms, declarations, hot-reload behavior.
- `blueprint.md` — Blueprint best practices, BP↔C++ boundary judgment, performance, asset references.

Author additional knowledge files (replication, GAS, Niagara, performance-profiling, etc.) on demand when project complexity surfaces a need. The `/branch-specialist` skill offers to wire matching knowledge files into newly-created specialists.
