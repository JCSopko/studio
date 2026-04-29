---
name: blueprint
description: "Reference knowledge for UE5 Blueprint visual scripting. BP↔C++ boundary judgment, performance, asset references, common patterns and pitfalls."
type: knowledge
version-pinned-to: "UE 5.4"
last-reviewed: 2026-04-28
---

# Blueprint (UE5) — Knowledge Reference

This file is loaded by the `studio:programming` agent's BP sub-persona mode and by the `blueprint-code` path-scoped rule when working in BP-relevant context. Reference material — patterns, judgment, idioms — not a tutorial. For exhaustive Blueprint documentation, see Epic's UE5 docs.

## When this knowledge applies

Blueprint is UE5's visual scripting system. Files are `.uasset` (binary, opaque to text diff). Logic lives in event graphs, function libraries, blueprint interfaces, and macro libraries. Blueprints subclass UObject/AActor types defined in C++ or Angelscript and add behavior in the visual graph.

## File and asset structure

- BP assets live alongside the content they extend, typically under `Content/Blueprints/<Category>/`.
- Naming convention (Epic-recommended, widely adopted):
  - `BP_` — Blueprint actor or component (e.g., `BP_PlayerCharacter`).
  - `BPI_` — Blueprint interface (e.g., `BPI_Damageable`).
  - `BPFL_` — Blueprint function library (e.g., `BPFL_MathHelpers`).
  - `BPM_` — Blueprint macro library.
  - `E_` — Enum defined in BP.
  - `S_` — Struct defined in BP.
  - `WBP_` — Widget blueprint (UMG).
  - `ABP_` — Animation blueprint.
- Data-only Blueprints: BP assets that override defaults of a parent class with no logic. Use these for variant content (different stats, meshes, materials) without spawning unique class hierarchies.

## BP↔C++/AS boundary judgment

The fundamental question for any new logic: which language? Decision rules:

| Use C++ or Angelscript when... | Use Blueprint when... |
|---|---|
| Logic is foundational and stable (combat math, save format, replication contracts) | Logic is content-shaped and likely to iterate (specific actor's behavior, level-specific scripts) |
| Performance-critical (per-frame, many instances, hot loops) | Designer needs to tune values and rewire connections |
| Type-strictness or refactor-safety matters | Visual flow makes the logic clearer than text |
| Multiple BP types need the same logic (factor to a helper) | A single asset's behavior is local to that asset |
| Logic crosses the network and needs versioning discipline | Quick prototype or one-off scripted event |
| You need version-controlled diff-friendly source | Designer ownership is the goal |

**Anti-pattern**: implementing core gameplay rules in Blueprint and then duplicating the rule across 30 BP assets. Refactor to C++/AS once duplication appears 3+ times.

**Anti-pattern**: implementing trivial actor-local glue logic in C++/AS just because "C++ is the right place." Blueprints are designed for this; use them.

## Graph cleanliness

A widely-used informal limit: **≤20 nodes per function or event graph** before the function should be split. Crowded graphs are unmaintainable and unreviewable.

- Extract a sequence of nodes into a function (right-click → Collapse to Function).
- Use Blueprint Macros for inline logic that doesn't need a separate function (faster but less reusable).
- Use Blueprint Interfaces for cross-class polymorphism instead of casting chains.

Wires:
- Avoid wires crossing each other; rearrange nodes.
- Reroute nodes (`/`) for long wires.
- Comments (`C` then drag) on every distinct logical block.

Variables:
- Categorize variables (`Category=` field on each) for grouping in the details panel.
- Local variables for function-scoped state; instance variables only for true class state.
- `Replicated` variables for networked state; pair with `RepNotify` callbacks for client-side reactions.

## Casting vs interfaces

`Cast<>` to a concrete BP class creates a hard reference — that BP and all its dependencies load when the casting BP loads. For loose coupling:

- **Blueprint Interfaces (BPI)**: define a contract (functions/events). Any BP implementing the interface can be invoked via the interface without casting. No hard reference to specific implementations.
- **Function dispatchers (Event Dispatchers)**: per-instance callback lists. The publisher fires; subscribers register at runtime.
- **Game Instance / Game State / Subsystems**: globally accessible via `Get Game Instance` / `Get Subsystem` nodes; references are resolved at runtime.

**Rule of thumb**: cast only when you actually need access to derived-class-specific state. If you just need behavior, use an interface.

## Data binding (UMG / Widgets)

- Avoid binding properties via `Binding` dropdown on widgets — these tick every frame.
- Prefer event-driven updates: subscribe to model changes (event dispatchers, property change notifications), update widget on event.
- ViewModel pattern: separate the data class from the widget. Widget reads/writes through the ViewModel, not directly to game state.
- `MVVM` plugin (UE 5.1+): formal ViewModel system; recommended for any non-trivial UI.

## Performance considerations

The BP→native cost gap is real but manageable. Key rules:

- **Tick the right things at the right rate.** Default Actor tick is per-frame. For BPs that don't need it, disable tick (`Set Actor Tick Enabled = false`) and use timers (`Set Timer by Function Name`) for periodic work.
- **Branch on cheap conditions first.** If a function early-outs 95% of the time, put the cheap check first.
- **Avoid casting in tick.** Cache the cast result on Begin Play; reuse the cached reference.
- **Avoid `Get All Actors of Class` in tick.** It enumerates the world. Use a registry/manager instead.
- **String operations are expensive.** Avoid string concatenation per frame. Prefer `FName` identifiers.
- **Looping in BP is slower than in C++.** For loops over thousands of items, factor the loop body into C++/AS.
- **Native events vs BP events.** UE-defined `BlueprintNativeEvent`s have a C++ default implementation that BPs can override. Default impl runs in C++ speed; BP override only runs when overridden.

Profiling tools:
- Stat blueprint summary (`stat blueprint_summary` console command).
- Unreal Insights (UE 5.x trace) — most accurate.
- Blueprint Profiler plugin — node-level.

## Asset references — the dependency cost

Hard references in Blueprint pull the entire dependency chain into memory at load time. Casting to a BP class hard-references it. Variable types of "object reference to specific class" hard-reference. Direct mesh/material references in default values hard-reference.

Soft references (`Soft Object Reference`, `Soft Class Reference`) defer loading. Pair with `Async Load Asset` / `Async Load Class` to load on demand. Use for:

- Asset selection dropdowns (level lists, character selection).
- Optional content (DLC, post-launch).
- Heavy assets used by only some code paths.

Audit dependency graph with the Reference Viewer (right-click any asset → Reference Viewer). Common surprise: a small UI widget pulls in 200MB because it references a BP that references a BP that references a complete environment scene.

## Event-driven patterns

- **Event Dispatchers (Multi-cast delegates)**: fire-and-forget broadcast from a BP instance. Subscribers `Bind Event` at runtime; unbind on cleanup.
- **Game Mode/State events**: per-match/level events flow through Game Mode (server) and Game State (replicated to all).
- **Subsystem events**: long-lived state (audio mixing, persistent unlocks, achievements) lives in subsystems with event dispatchers; widgets/actors subscribe.

## Replication in Blueprint

Networked BPs:
- Set `Replicates: true` on the BP class default.
- Mark replicated variables with the `Replicated` flag.
- Use `Replicated` (always) or `RepNotify` (replicated + client callback) per variable.
- Server-only logic uses `Authority` switch (`Switch Has Authority`).
- Custom replication: `Server`/`Client`/`NetMulticast` events with `Reliable`/`Unreliable` and `Validated`.
- Always validate server RPCs — never trust the client. RPCs from client to server should sanity-check inputs and reject impossible values.

## Animation Blueprints

- Event graph for state-driving logic (which state to transition to).
- Anim graph for state-blending logic (how poses combine).
- Avoid heavy logic in Anim graph — it runs on the worker thread; performance matters.
- Property access nodes (UE 5.x) — designed for fast read of pawn state without casts.

## Common pitfalls

- **Hard-referencing every cast result.** Audit Reference Viewer; convert frequent-but-loose dependencies to interfaces or soft refs.
- **Tick functions doing 200ms of work occasionally.** Profile per-frame budget; stagger work across frames or move to a timer.
- **Spawning Blueprints with `Spawn Actor from Class` in tight loops.** Each spawn does a class lookup + UObject creation; pool spawned actors instead.
- **`Print String` left in shipping builds.** Compiles into shipping by default; delete or wrap in `#if WITH_EDITOR` (C++) or check `is editor` in BP.
- **Variable named `temp`/`x`/`a`.** Months later, no one knows what it is. Rename early.
- **Blueprint inheritance chains 5+ levels deep.** Refactor; deep BP inheritance is a maintenance trap.
- **Logic duplicated across 10 BPs.** Factor into a parent class, BP function library, or interface.
- **Calling `Get Player Controller` repeatedly per frame.** Cache it on Begin Play.
- **`Construction Script` that does heavy work.** Construction Script runs in editor on every property change. Heavy ops there make the editor unusable.
- **`OnConstruction` / `BeginPlay` ambiguity.** Construction Script runs at editor edit-time AND at spawn; BeginPlay only at runtime spawn. Don't put runtime-only logic in Construction Script.

## When to graduate Blueprint logic to C++/AS

Signs that BP logic should move:

1. The same BP function appears (copy-pasted) in 3+ Blueprints.
2. The function exceeds 30 nodes consistently.
3. Profiling shows the function in the top 10 cost contributors.
4. The logic is type-sensitive and refactor risk is rising.
5. The logic crosses the network — networked code benefits from text-diff history.
6. The function should be unit-testable.

Graduation path:
1. Author the C++/AS class with `BlueprintCallable` methods.
2. Replace the BP function calls with calls to the new class.
3. Delete the now-empty BP function.
4. Run regression test on affected BPs.

## When to keep logic in Blueprint

Signs that BP is the right home:

1. The logic is asset-local (one specific actor's specific behavior).
2. Designers iterate on it without programmer involvement.
3. The logic is mostly visual flow (timeline, sequence, parallel events).
4. Performance is non-critical and not in tick.
5. Refactoring it to C++/AS would require duplicating UE-side machinery (lighting, rendering, anim) that BP wraps cleanly.

## Authoritative external references

- Epic UE5 official Blueprint documentation.
- Tom Looman's UE blog (community-respected source for BP/C++ patterns).
- Project's own BP examples — open `Content/Blueprints/Examples/` (if exists) or pick a representative BP and read its graph.

## Maintenance

This file's `version-pinned-to:` declares which UE version the patterns reflect. UE5 minor versions can change Blueprint plumbing (the MVVM plugin, FastArrays, etc.). The `prometheus:harness-health` skill flags this file when pinned-to drifts from the project's declared engine version.
