---
name: branch-specialist
description: "Branch a discipline agent into a project-scope specialist agent when the project shows the need. Defers specialist proliferation until evidence justifies it. Walks the user through naming, parent discipline, scope, knowledge-file wiring, and writes the new agent file."
argument-hint: "[name]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, AskUserQuestion
---

# Branch Specialist

This skill creates a new project-scope specialist agent that branches off one of Studio's six discipline agents (`programming`, `design`, `art`, `audio`, `qa`, `production`). Use it when the project's complexity surfaces a need for deeper specialization than the discipline agent's general knowledge — e.g., dedicated Angelscript work, GAS-heavy UE5 game, complex level-design ownership.

The skill produces an agent that:
- Lives at `<project>/.claude/agents/<name>.md` (project-scope by default).
- Inherits the parent discipline's methods and boundaries.
- References relevant knowledge files (`.claude/knowledge/<topic>.md`) automatically.
- Shadows the parent discipline's bare invocation only for the declared scope; namespaced calls (`studio:<discipline>`) still work.

## Phase 1 — Parse Arguments

Read the `[name]` argument. If missing, prompt interactively:

> What name should this specialist agent have? Use kebab-case (e.g., `ue-as-specialist`, `gas-specialist`, `level-pacing-specialist`). The name will become the file `<project>/.claude/agents/<name>.md` and the bare invocation in this project.

If a name is provided, validate it: kebab-case (lowercase, hyphens), no extension, doesn't collide with an existing agent file in `<project>/.claude/agents/` (Glob to check). If a collision exists, surface the existing file and ask whether to overwrite or pick a different name.

## Phase 2 — Determine Parent Discipline

Ask which discipline this specialist branches from:

```
AskUserQuestion:
  question: "Which discipline does this specialist branch from?"
  multi: false
  options:
    - id: programming
      label: programming
      description: "Tech-lead, gameplay/engine/AI/network/tools/UI, devops, security, performance"
    - id: design
      label: design
      description: "Game design, systems, levels, economy, narrative, writing, world-building, live-ops"
    - id: art
      label: art
      description: "Art direction, technical art, UX"
    - id: audio
      label: audio
      description: "Audio direction, sound design"
    - id: qa
      label: qa
      description: "QA lead, QA tester, accessibility"
    - id: production
      label: production
      description: "Production, prototyping, analytics, community, release, localization"
```

Read the parent discipline's agent file at `.claude/agents/<discipline>.md` to get its frontmatter conventions, methods, and boundaries — the specialist will inherit these.

## Phase 3 — Define Scope

Ask:

> What is this specialist's scope? Describe in one or two sentences. Examples: "Hazelight Angelscript work — UCLASS macros, UE bindings, hot-reload behavior" / "GAS abilities, gameplay effects, attribute sets" / "Level pacing and encounter flow for action games."

Capture the scope as the specialist's primary description.

## Phase 4 — Knowledge File Wiring

Glob `.claude/knowledge/*.md` to list available knowledge files. For each, read the frontmatter `name` and `description`. Show the list to the user with:

> The following knowledge files are available. Which (if any) should this specialist load when active?

```
AskUserQuestion:
  question: "Which knowledge file(s) should this specialist load?"
  multi: true
  options:
    - id: <name>
      label: <name>
      description: <description>
    ... (one per knowledge file)
    - id: none
      label: "(none — author from scratch)"
      description: "Specialist won't auto-load any knowledge file"
```

If the user selects one or more, the specialist's body will include `@knowledge/<name>.md` directives.

If the project doesn't have a knowledge file matching the specialist's scope, offer:

> No knowledge file matches this scope. Would you like to: (a) author a new knowledge file at `.claude/knowledge/<name>.md` as part of this specialist creation, (b) skip and add the knowledge file later, or (c) reference an external doc URL?

If (a), spawn a knowledge-file authoring sub-task with frontmatter (`type: knowledge`, `version-pinned-to`, `last-reviewed`) and a body skeleton the user fills in.

## Phase 5 — Placement

Ask:

```
AskUserQuestion:
  question: "Where does this specialist live?"
  multi: false
  options:
    - id: project-scope
      label: "project-scope (Recommended)"
      description: "<project>/.claude/agents/<name>.md — overrides studio:<discipline> for this scope only on this project"
    - id: plugin-graduate
      label: "plugin (rare)"
      description: "Studio plugin canonical — requires Joe approval per migration rule"
```

If `plugin-graduate`, surface a flag:

> Plugin-layer specialists graduate per the Prometheus governance migration rule: project-agnostic content + multi-project benefit + Joe explicit approval. Has Joe explicitly approved this graduation? If not, recommend project-scope and revisit later when multi-project use is demonstrated.

Default to project-scope unless the user is firm.

## Phase 6 — Draft the Specialist Agent

Compose the specialist agent file with this structure:

```yaml
---
name: <name>
description: "<scope description>. Branches from studio:<discipline>. Use this agent when the project's <discipline-scope> work specifically requires <specialty>."
tools: <inherited from parent discipline, possibly trimmed>
model: opus
maxTurns: 20
memory: project
parent-discipline: <discipline>
knowledge-loads: [<knowledge-file-names>]
---

# <Name in Title Case>

Branches from `studio:<discipline>`. Specializes in <scope>. Inherits the parent discipline's methods and boundaries; this file extends the parent with specialty-specific knowledge and adjusts the parent's behavior in this scope.

## Specialty scope

<paragraph: what this specialist owns and what it does>

## Inherited from parent discipline

This specialist inherits the cross-mode methods and boundaries of `studio:<discipline>`. Read that file for the foundation. The specialty-specific overrides below extend (do not replace) the parent.

## Specialty-specific patterns

<bullets or paragraphs from the user's scope description, with knowledge-file references>

For full reference: @knowledge/<knowledge-file>.md
(or other relevant knowledge files)

## Override semantics

This specialist shadows `studio:<discipline>` for the declared scope only. When the work falls inside <specialty scope>, bare invocation of <name> handles it. When the work is general <discipline> outside this scope, the parent discipline handles it via `studio:<discipline>`. Namespaced calls always work.

## Frontmatter notes

- `parent-discipline: <discipline>` documents the branching relationship for `harness-health` and audit purposes.
- `knowledge-loads: [...]` declares which knowledge files this specialist auto-loads.
- Other frontmatter fields inherited from parent discipline conventions.
```

Show the draft to the user:

> Here's the draft for `<project>/.claude/agents/<name>.md`. Does this look right? Edit any section before I write the file.

## Phase 7 — Write and Confirm

After approval, write to:

- **project-scope:** `<project>/.claude/agents/<name>.md`
- **plugin-graduate (if approved):** `claude-plugin/agents/<name>.md` (or `<studio-repo>/.claude/agents/<name>.md` for the canonical Studio plugin)

Confirm:

> Written `<filepath>`. Test by invoking `<name>` from a session in this project.

## Phase 8 — Optional CLAUDE.md Agent Table Update

Ask:

> Update the project's CLAUDE.md agent table to include this specialist?

If yes, locate the agent table in `<project>/CLAUDE.md` (look for "Agents" section or "## Agent Table"). Append a row matching the project's existing table format.

## Phase 9 — Recommended Next Steps

After write:

- "Smoke-test the specialist by invoking `<name>` and asking a representative scoped question."
- "If the specialty involves a new file path the project edits often, consider adding a path-scoped rule at `<project>/.claude/rules/<topic>-code.md` that auto-fires the relevant knowledge file."
- "If usage proves out across multiple projects, propose graduating to plugin layer per the migration rule."

## Verdict

PASS / CREATED — agent file written, project ready to use.
