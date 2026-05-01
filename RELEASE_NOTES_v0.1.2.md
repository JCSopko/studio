# Studio v0.1.2 — Release Notes

**Tag:** `studio-v0.1.2` at commit `3037cdd` on `distill`.
**Date:** 2026-05-01.
**Predecessor:** [`studio-v0.1.1`](https://github.com/JCSopko/studio/releases/tag/studio-v0.1.1) (2026-04-28) at commit `0066b5e`.

A small-but-cohesive patch release. Two themes: **plugin manifest schema fixes** (so the marketplace + plugin manifests validate against the current `code.claude.com` schema) and **knowledge-file corrections** for AngelScript and Blueprint authoring on the Hazelight UE5 fork. No breaking changes; pure-superset of v0.1.1.

## Summary by area

### Plugin marketplace + manifest schemas

- **`marketplace.json` schema fix** (`20abfb9`): added the required `owner` field, simplified the `source` form from an object to a string per the current `code.claude.com` docs. Without this, the marketplace failed `claude plugin validate` at the marketplace layer and consumers couldn't load Studio cleanly.
- **`plugin.json` author schema fix** (`75c47da`): converted the `author` field from a string to an object with `name` / `email` / `url`. Symmetric to the marketplace fix; brings the plugin manifest in line with the current schema.

These two are required for clean install on Claude Code 2.x harnesses; v0.1.1 worked but emitted validation warnings.

### AngelScript knowledge file (`claude-plugin/knowledge/angelscript.md` + `.claude/knowledge/angelscript.md`)

Three corrections + one new section against the Hazelight AngelScript fork:

- **Engine-variant awareness** (`9b19d81`): clarified that the knowledge file is calibrated against the Hazelight AS fork (`https://angelscript.hazelight.se/`), not stock UE5. Multi-rule corrections + specialist Engine-variant awareness.
- **Function library namespace strip + Save/GetGameMode canonical forms** (`0cc497b`): documented the namespace-strip rule (AS fork strips `UScript*` prefixes from Python-bound names) and the canonical AS forms for save subsystem access + game mode lookup.
- **Subsystem inheritance rule** (`722e3ad`): documented the `UScript*Subsystem` wrapper convention — the AS-callable façade type for engine subsystems differs from the stock UE5 names. Surfaced 2026-04-30 during Cozy AS Day-2 §3 diagnosis.
- **Subsystem `Get()` signature fix** (`98fc2d3`): corrected the documented `Get()` signature for engine / game-instance / world subsystems — these are parameterless in the AS fork (verified against `Bind_Subsystems.cpp`). Earlier doc had an incorrect signature lifted from stock UE5.

### Blueprint knowledge file (`claude-plugin/knowledge/blueprint.md` + `.claude/knowledge/blueprint.md`)

- **Asset operations API tier — `EditorAssetLibrary` vs `AssetRegistry → IAssetTools`** (`3037cdd`): new section documenting when to use the disk-load-coupled high-level API (`EditorAssetLibrary`) vs the in-memory low-level path (`AssetRegistry.get_asset()` + `IAssetTools.RenameAssets`). The low-level path survives `RF_Transient` and other unusual states where high-level operations re-load from disk and fail. Surfaced 2026-05-01 during HOFF 8 RF_Transient blueprint recovery.

This last entry is the load-bearing knowledge for any agent doing Python-driven asset operations under unusual conditions (post-recovery, mid-rebuild, transient flags, etc.).

## Files changed

| Area | Files |
|---|---|
| Marketplace manifest | `marketplace.json` |
| Plugin manifest | `claude-plugin/plugin.json` |
| AS knowledge | `claude-plugin/knowledge/angelscript.md`, `.claude/knowledge/angelscript.md` |
| BP knowledge | `claude-plugin/knowledge/blueprint.md`, `.claude/knowledge/blueprint.md` |

The dual-mirror pattern (`claude-plugin/` ↔ `.claude/`) is documented in v0.1.1's commit message and remains in place; v0.1.2 maintains parity across both trees.

## Commits in v0.1.2 (post-v0.1.1)

```
3037cdd  iji: BP knowledge — EditorAssetLibrary vs IAssetTools API tier (RF_Transient recovery)
98fc2d3  iji: AS knowledge — fix subsystem Get() signature (parameterless for engine/game-instance/world)
722e3ad  iji: AS knowledge — subsystem inheritance rule (UScript*Subsystem wrapper)
0cc497b  iji: AS knowledge — function library namespace strip + Save/GetGameMode canonical forms
9b19d81  iji: AS knowledge + rule corrections + specialist Engine variant awareness
75c47da  jodot: Fix plugin.json author schema (string -> object)
20abfb9  jodot: Fix marketplace.json schema (add owner, simplify source to string)
```

## Upgrading from v0.1.1

If you pin Studio via a `studio.lock`-style mechanism (as the Cozy harness does), bump the pinned SHA to `3037cdd` (or pinned-tag to `studio-v0.1.2`). No file moves, no configuration changes, no CLAUDE.md edits required on the consumer side. The two manifest schema fixes are entirely internal to Studio's metadata.

If you cloned Studio directly and want the v0.1.2 state: `git fetch && git checkout studio-v0.1.2`.

## Known limitations

- The `claude-plugin/` ↔ `.claude/` dual-mirror is drift-prone by construction. v0.1.2 preserves the duality; a future v0.2 may collapse it. Until then, edits should land in both trees.
- `studio-v0.1.1` git tag still points at the original commit `0066b5e` (pre-schema-fix); `studio-v0.1.2` is the first tag whose state is fully validated against the current `code.claude.com` plugin schema.

## Lineage references

- v0.1.1 release: <https://github.com/JCSopko/studio/releases/tag/studio-v0.1.1>
- v0.1.2 commit chain on `distill`: `git log studio-v0.1.1..studio-v0.1.2`
- HOFF 8 (origin of the BP knowledge addition): `iji-vault/7-memory/sessions/iji/2026-05-01-0405-monolith-pair-b.md`
- Initiative: [studio-template-development](https://github.com/JCSopko/iji-vault/blob/main/3-initiatives/P1/studio-template-development.md)
