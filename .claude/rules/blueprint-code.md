---
paths: ["**/Content/**/*.uasset", "**/Blueprints/**"]
type: rule
description: Standards for Blueprint visual scripting in UE5 projects.
---

# Blueprint Code Rules

Auto-fires when working in Blueprint context (paths under `**/Content/**/*.uasset` or any `**/Blueprints/**` directory). Loads `@knowledge/blueprint.md` for full reference; the rules below are the enforced subset.

Reference: `@knowledge/blueprint.md`

## Naming convention

| Prefix | Asset type |
|---|---|
| `BP_` | Blueprint actor or component |
| `BPI_` | Blueprint interface |
| `BPFL_` | Blueprint function library |
| `BPM_` | Blueprint macro library |
| `WBP_` | Widget blueprint (UMG) |
| `ABP_` | Animation blueprint |
| `E_` | Enum defined in BP |
| `S_` | Struct defined in BP |

Asset folder: `Content/Blueprints/<Category>/`. Categories follow project structure.

## Required

- **Graphs ≤ 20 nodes per function or event graph.** Beyond 20, extract to a function (right-click → Collapse to Function) or interface call.
- **Comment blocks on every distinct logical block.** A graph without comments is a graph the next reader will rewrite from scratch.
- **Categorized variables.** Every BP variable has a `Category` field set; details panel groups by category.
- **Cached references for repeated casts.** Cast once on Begin Play, store the result in an instance variable, reuse.
- **Soft references for optional / on-demand assets.** Use `Soft Object Reference` / `Soft Class Reference` + `Async Load Asset` for assets used by some code paths only.
- **Server-side input validation on networked RPCs.** Server RPCs from clients sanity-check inputs and reject impossible values. Never trust the client.
- **`Replicated` flag on networked variables.** Pair with `RepNotify` (`OnRep_*`) when the client needs to react.

## Prohibited

- **Hard-referencing BP classes via Cast in tick paths.** The cast pulls the entire class dependency graph into memory at load time; in tick it also runs every frame.
- **`Get All Actors of Class` in tick.** Enumerates the world. Use a registry / manager instead.
- **String concatenation in tick.** Allocates. Use `FName` identifiers.
- **`Print String` in shipping builds.** Wrap in editor checks or delete before ship.
- **Deep BP inheritance chains (5+ levels).** Maintenance trap. Refactor.
- **Blueprint-implemented gameplay rules duplicated across 3+ BPs.** Factor to BP function library, parent class, or C++/Angelscript.
- **Heavy work in `Construction Script`.** Construction Script runs in editor on every property change; heavy ops make the editor unusable. Use `BeginPlay` for runtime-only setup.
- **`Property Bindings` for tickable widget data.** Tick every frame. Use event-driven updates (subscribe to model change events).

## When to graduate to C++ / Angelscript

Move logic out of BP when any of these are true:

1. The same BP function appears (copy-pasted) in 3+ Blueprints.
2. The function exceeds 30 nodes consistently.
3. Profiling shows the function in the top 10 cost contributors.
4. The logic is type-sensitive and refactor risk is rising.
5. The logic crosses the network — networked code benefits from text-diff history.
6. The function should be unit-testable.

Graduation procedure:
1. Author the C++/AS class with `BlueprintCallable` methods that mirror the BP function's signature.
2. Replace BP function calls with calls to the new class.
3. Delete the now-empty BP function.
4. Run regression test on affected BPs.

## When BP is the right home

Keep logic in BP when:

1. The logic is asset-local (one specific actor's specific behavior).
2. Designers iterate on it without programmer involvement.
3. The logic is mostly visual flow (timeline, sequence, parallel events).
4. Performance is non-critical and not in tick.
5. Refactoring to C++/AS would require duplicating UE-side machinery (lighting, rendering, anim graphs) that BP wraps cleanly.

## Performance discipline

- **Tick the right things at the right rate.** Disable Actor tick on BPs that don't need it; use timers for periodic work.
- **Branch on cheap conditions first.** Early-out path runs more often.
- **Profile before optimizing.** Use `stat blueprint_summary`, Unreal Insights, or the Blueprint Profiler plugin.

## Reference graph audit

Use the Reference Viewer (right-click any asset → Reference Viewer) to audit dependency graphs. Frequent surprise: a small UI widget pulls in hundreds of MB through transitive references. Convert frequent-but-loose dependencies to interfaces or soft references.

## Reference

Full BP↔C++/AS boundary judgment, performance considerations, asset-reference cost, animation patterns, and pitfall list live in `@knowledge/blueprint.md`. This rule is the auto-fired enforced subset.
