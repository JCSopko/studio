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
- **Subsystem accessor pattern**: every singleton-style subsystem provides `static T Get(UObject WorldContextObject)` for callers to look it up via game-instance.

## Prohibited

- **Hardcoded magic numbers in gameplay code paths**. Externalize to data assets (DataTable, DataAsset) or a config UPROPERTY exposed for edit.
- **Direct delegate `Execute()` on potentially-unbound delegates** — use `ExecuteIfBound` or `Broadcast` (multi-cast).
- **Local UObject creation without immediate reference** — `NewObject<T>()` must be assigned to a tracked UPROPERTY field or returned to a caller that will track it. Local-variable-only references are GC candidates the moment scope exits.
- **Using `==` on `FString` content in hot paths** — content compare allocates. Use `FName` for identifiers, reserve `FString` content compare for cold paths.
- **Heavy work in subsystem `Initialize`** — `Initialize` runs at engine / game-instance startup. Defer heavy setup to a separate `Start()` method invoked once the world is ready.
- **`BlueprintPure` on a side-effecting function** — pure nodes evaluate every read; hidden side-effects accumulate.
- **`TArray` indexing without bounds check in defensive paths** — use `IsValidIndex(i)` before `arr[i]` when input source is uncertain.

## Encouraged patterns

- **Snapshot iteration on multi-cast delegate dispatch** — copy the subscriber list before iterating so subscribers may unsubscribe during dispatch without invalidating the loop. See `EventBus.as` reference (Cozy `SquirrelTamagotchi/Script/Subsystems/EventBus.as`) for the canonical pattern.
- **Recursion-cap on event-bus-style dispatch** — explicit depth counter with a hard limit prevents unbounded re-entry.
- **`Replicated` UPROPERTY paired with `OnRep_*` callback** — when a property crosses the wire, the receiving side reacts in the OnRep handler.
- **`@knowledge/angelscript.md` referenced when in doubt** — the knowledge file holds idiomatic patterns the rule does not exhaustively enforce.

## Blueprint interop

- AS classes intended for BP subclassing declare `UCLASS(Blueprintable)`.
- AS classes intended only as data types for BP variables declare `UCLASS(BlueprintType)`.
- AS exposes virtual methods to BP via `UFUNCTION(BlueprintImplementableEvent)` (declared in AS, implemented in BP).

## Hot reload caveats

- Class layout changes (add/remove fields) hot-reload cleanly.
- Class identity changes (rename, file move) clear subsystem state on next reload — restart editor after structural changes.
- Compile errors block hot reload; the previously-compiled version remains active. Watch the editor's compile output.

## Engine version safety

Before suggesting AS bindings, check `docs/engine-reference/unreal/VERSION.md` for the project's pinned UE version + Hazelight angelscript-ue4 commit. Some bindings (Niagara, GAS, CommonUI) ship with specific minor versions. Flag if the API was added after the pin.

## Reference

Full idioms, type system, casting, hot-reload behavior, BP↔AS boundary, performance notes, and authoritative external sources live in `@knowledge/angelscript.md`. This rule is the auto-fired enforced subset.
