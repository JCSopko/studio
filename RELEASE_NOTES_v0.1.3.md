# Studio v0.1.3 — Release Notes

**Tag:** `studio-v0.1.3` (pending — not yet pushed/tagged).
**Branch:** `feat/art-mcp-tool-surface` (off `distill`).
**Predecessor:** [`studio-v0.1.2`](https://github.com/JCSopko/studio/releases/tag/studio-v0.1.2) at commit `4841e1a`.
**Date:** 2026-05-01.

A small feature release that opens the `studio:art` agent's tool surface to project-provided MCP servers for Blender and Unreal Engine. Backwards-compatible: pure addition to the agent's tool allowlist; nothing removed, no agent behavior changed for projects that don't enable those MCP servers.

## Why

During Cozy Op-5 RQST 1 (placeholder mesh authoring), the `studio:art` agent was tasked with driving Blender via Cozy's project-scope `cozy-blender-bridge.py` MCP server (registered as `mcp__blender__*` in the project's `.mcp.json`). The agent failed to access `mcp__blender__execute_code` despite the project's `enabledMcpjsonServers` correctly listing the bridge.

Root cause: the `tools:` allowlist in `claude-plugin/agents/art.md` (and its dual-mirror `.claude/agents/art.md`) was an explicit list of built-in tools — `Read, Glob, Grep, Write, Edit, Bash, WebSearch` — with no MCP entries. Claude Code's two-layer gate evaluates **both** the project's `enabledMcpjsonServers` (server-level enable) **and** the agent's `tools:` field (per-tool allowlist). Project-level enablement was sufficient for non-agent harness use, but spawning `studio:art` inherited the plugin's restrictive allowlist and stripped the MCP tools out of the spawned agent's surface.

The same pattern would block any UE5 work the art agent might do via project-scope Unreal MCP servers — FBX import, in-editor screenshot capture, material assignment via `MaterialEditingLibrary` — none of which were reachable.

Op-5 unblocked itself with a workaround. This release is the upstream fix.

## What changed

`tools:` field on `art.md` (both mirrors) extended from:

```
tools: Read, Glob, Grep, Write, Edit, Bash, WebSearch
```

to:

```
tools: Read, Glob, Grep, Write, Edit, Bash, WebSearch, mcp__blender__*, mcp__unreal__*, mcp__monolith__*, mcp__runreal__*
```

The four MCP server prefixes cover the canonical art-agent tool fleet:

- `mcp__blender__*` — placeholder mesh authoring, vertex-color pipelines, viewport capture, FBX export driven from Blender.
- `mcp__unreal__*` — generic Unreal MCP server prefix (in-editor screenshot, asset import).
- `mcp__monolith__*` — Cozy's monolithic UE5 plugin MCP (asset operations, material editing, scene queries).
- `mcp__runreal__*` — alternative UE5 Python-Remote-Execution-style server.

The frontmatter notes section in `art.md` was updated in lockstep to document the addition + rationale.

## Architectural choice: explicit per-server listing, not `mcp__*` wildcard

The original Op-5 brief considered a `mcp__*` wildcard — broader, automatically gated by the project's `enabledMcpjsonServers`. We chose **explicit per-server listing** instead, for three reasons:

1. **Studio convention.** No agent in the plugin uses MCP wildcards; all use explicit allowlists of built-in tools. The wildcard approach would be a precedent-setting choice better made deliberately on its own merits, not as a side effect of this fix.
2. **Surface scoping.** A project enabling `mcp__mattermost__*` or `mcp__gmail__*` for production/community-comms work shouldn't automatically grant the art agent inbound message access. Explicit MCP listing keeps the agent's reach bounded to its domain (Blender + Unreal).
3. **Reviewability.** A `git diff` on the `tools:` field shows exactly what art is allowed to do. With a wildcard, the surface is implicit in whatever MCP servers any consuming project chooses to enable — visible only by reading the consumer's `.mcp.json`.

A future v0.2 architectural review may revisit wildcard adoption across the plugin if the explicit-list pattern accumulates maintenance burden.

## Impact

Any project that enables `mcp__blender__*`, `mcp__unreal__*`, `mcp__monolith__*`, or `mcp__runreal__*` MCP servers via its `enabledMcpjsonServers` setting will now see those tools available inside `studio:art` agent sessions. Projects with no MCP servers enabled, or with only unrelated MCP servers (e.g., `mcp__mattermost__*`), see no behavior change.

## Versioning

`0.1.2 → 0.1.3` per semver. Feature addition (`tools:` allowlist extended), backwards-compatible, no migration required. The `claude-plugin/.claude-plugin/plugin.json` `version` field bumps from `0.1.1` (which v0.1.2 left at `0.1.1` — see Known limitations in v0.1.2 notes) to `0.1.3` to align manifest with release tag.

## Files changed

| Area | Files |
|---|---|
| Agent (canonical) | `claude-plugin/agents/art.md` |
| Agent (dual-mirror) | `.claude/agents/art.md` |
| Plugin manifest | `claude-plugin/.claude-plugin/plugin.json` (version `0.1.1` → `0.1.3`) |

## Upgrading from v0.1.2

If you pin Studio via `studio.lock`-style mechanism (Cozy harness pattern):

1. Push branch + tag (Joe's responsibility — `feedback_no_autonomous_git_push` in effect).
2. Bump `Cozy/.claude/studio.lock` to the new commit/tag.
3. Restart any running `studio:art` agent sessions to pick up the new tool surface.

If you cloned Studio directly: `git fetch && git checkout studio-v0.1.3`.

## Known limitations

- Other discipline agents (`programming`, `production`, `qa`, etc.) still have built-in-only `tools:` allowlists. A future release should either (a) audit each agent's MCP needs and extend per-discipline, or (b) make the architectural choice to adopt `mcp__*` wildcards plugin-wide. This release is scoped to `art` specifically because it was the Op-5 unblock.
- The dual-mirror (`claude-plugin/` ↔ `.claude/`) drift risk noted in v0.1.2 remains. Both mirrors were edited in this release.

## Lineage references

- v0.1.2 release: <https://github.com/JCSopko/studio/releases/tag/studio-v0.1.2>
- Op-5 origin: pair-launched Iji + Jodot session 2026-05-01, Cozy `cozy-blender-bridge.py` MCP rollout for placeholder mesh authoring.
- Initiative: [studio-template-development](https://github.com/JCSopko/iji-vault/blob/main/3-initiatives/P1/studio-template-development.md)
