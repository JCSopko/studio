---
paths: ["**/*.as", "**/Script/**"]
type: rule
description: Standards for Hazelight Angelscript (.as) code in UE5 projects.
---

# Angelscript Code Rules

Auto-fires when editing files matching `**/*.as` or any path under `**/Script/**`. Loads `@knowledge/angelscript.md` for full reference; the rules below are the enforced subset.

Reference: `@knowledge/angelscript.md`

## Required

- **One primary class per file.** File name matches the primary class name without UE prefix (`EventBus.as` → `class UEventBus`).
- **UE prefixes on classes**: `U` (UObject), `A` (AActor), `F` (struct), `E` (enum), `I` (interface). No prefix-free class names except for delegates (which use `F` prefix by convention: `FCozyEventDelegate`).
- **`UPROPERTY()` on every UObject reference field** — UE's GC tracks references through tagged properties. Untagged UObject fields can be reclaimed unexpectedly.
- **`UFUNCTION()` on every method called from Blueprint** — bare AS methods are invisible to BP. Tag with `BlueprintCallable`, `BlueprintPure`, `BlueprintOverride`, etc. as appropriate.
- **Null-check after `Cast<T>()`** — Cast returns nullptr on failure; never assume success.
- **`ExecuteIfBound` on single-cast delegates that may be unbound** — direct `Execute()` crashes if unbound.
- **Explicit category on UPROPERTY/UFUNCTION** — `Category = "ProjectName|Subsystem"` for editor organization. Single-word categories acceptable for project-internal-only fields.
- **Subsystem accessor pattern**: callers use the AS-binding-layer's auto-generated `T::Get(WorldContextObject)`. **Do NOT declare `static T Get(...)` on the subsystem class** — Hazelight AS rejects user-declared static member functions. The binding layer provides the accessor for free.

## Prohibited

- **Hardcoded magic numbers in gameplay code paths**. Externalize to data assets (DataTable, DataAsset) or a config UPROPERTY exposed for edit.
- **Direct delegate `Execute()` on potentially-unbound delegates** — use `ExecuteIfBound` or `Broadcast` (multi-cast).
- **Local UObject creation without immediate reference** — `NewObject<T>()` must be assigned to a tracked UPROPERTY field or returned to a caller that will track it. Local-variable-only references are GC candidates the moment scope exits.
- **Using `==` on `FString` content in hot paths** — content compare allocates. Use `FName` for identifiers, reserve `FString` content compare for cold paths.
- **Heavy work in subsystem `Initialize`** — `Initialize` runs at engine / game-instance startup. Defer heavy setup to a separate `Start()` method invoked once the world is ready.
- **`BlueprintPure` on a side-effecting function** — pure nodes evaluate every read; hidden side-effects accumulate.
- **`TArray` indexing without bounds check in defensive paths** — use `IsValidIndex(i)` before `arr[i]` when input source is uncertain.

## Encouraged patterns

- **Use `event` over hand-rolled subscriber lists** — Hazelight AS provides multicast `event` declarations that handle dispatch + binding cleanly. Prefer `event FMyDelegate OnFoo;` + `OnFoo.Broadcast(...)` over a manual `TMap<FName, TArray<FCozyEventDelegate>>` subscriber registry.
- **Snapshot iteration on multi-cast delegate dispatch** — when implementing custom dispatch (e.g., for typed-event-bus where `event` doesn't fit), copy the subscriber list before iterating so subscribers may unsubscribe during dispatch without invalidating the loop.
- **`Math::` not `FMath::`** — Hazelight renames `FMath::*` to `Math::*` in script. `Math::Clamp`, `Math::Min`, `Math::Lerp`, etc.
- **`0.0f` literals in float32 contexts** — `float` defaults to 64-bit double in AS. Mixing `0.0` (float64) literals with `float32` member fields causes overload-resolution failures in `Math::Clamp` and friends.
- **Global `SpawnActor(Class, Loc, Rot)` for spawning** — class as value, NOT `Class::StaticClass()`. Returns the spawned actor reference directly.
- **`default` keyword for property initialization** — AS forbids constructors. Inline initializers via `default Field = value;` set defaults safely under hot-reload.
- **`BlueprintOverride` body without `Super::Method()`** — AS does NOT provide a `Super` namespace. The body is the override; parent invocation is implicit through BP's call graph.
- **Recursion-cap on event-bus-style dispatch** — explicit depth counter with a hard limit prevents unbounded re-entry.
- **`Replicated` UPROPERTY paired with `OnRep_*` callback** — when a property crosses the wire, the receiving side reacts in the OnRep handler.
- **`@knowledge/angelscript.md` referenced when in doubt** — the knowledge file holds idiomatic patterns the rule does not exhaustively enforce.

## Blueprint interop

- AS classes intended for BP subclassing declare `UCLASS(Blueprintable)`.
- AS classes used as data types for BP variables: leave `UCLASS()` bare. **`BlueprintType` is NOT a valid Hazelight AS UCLASS specifier** — UObject derivatives are implicitly Blueprint-usable as types. `BlueprintType` is a UE C++ specifier that AS does not accept.
- AS exposes virtual methods to BP via `UFUNCTION(BlueprintImplementableEvent)` (declared in AS, implemented in BP).

## Hot reload caveats

- Class layout changes (add/remove fields) hot-reload cleanly.
- Class identity changes (rename, file move) clear subsystem state on next reload — restart editor after structural changes.
- Compile errors block hot reload; the previously-compiled version remains active. Watch the editor's compile output.

## Engine version safety

Before suggesting AS bindings, check `docs/engine-reference/unreal/VERSION.md` for the project's pinned UE version + Hazelight angelscript-ue4 commit. Some bindings (Niagara, GAS, CommonUI) ship with specific minor versions. Flag if the API was added after the pin.

## Reference

Full idioms, type system, casting, hot-reload behavior, BP↔AS boundary, performance notes, and authoritative external sources live in `@knowledge/angelscript.md`. This rule is the auto-fired enforced subset.
