---
name: angelscript
description: "Reference knowledge for the Hazelight Angelscript fork of UE5. Idioms, declarations, AS↔UE bindings, hot-reload behavior, common pitfalls."
type: knowledge
version-pinned-to: "UE 5.4 + Hazelight angelscript-ue4 main as of 2026-04"
last-reviewed: 2026-04-28
---

# Angelscript (Hazelight UE Fork) — Knowledge Reference

This file is loaded by the `studio:programming` agent's AS sub-persona mode and by the `angelscript-code` path-scoped rule when editing `.as` files. It is reference material — patterns, gotchas, idioms — not a tutorial. For an authoritative external reference, see Hazelight's `angelscript-ue4` repository docs.

## When this knowledge applies

Angelscript is Hazelight's source-built UE5 fork that adds first-class scripting via the AngelScript language with deep UE bindings. It is distinct from vanilla UE5 (C++ + Blueprint) and from custom UE plugins. Code lives in `.as` files; classes registered via `UCLASS`/`UFUNCTION`/`UPROPERTY` macros are first-class UObjects from UE's perspective.

## File and class structure

- One primary class per `.as` file. File name matches the primary class name without the type prefix: `EventBus.as` → `class UEventBus`.
- Files live under `<Project>/Script/...`. Subfolder convention mirrors namespace intent (`Script/Subsystems/`, `Script/Components/`, `Script/Actors/`, `Script/Data/`).
- Classes use UE prefixes: `U` for UObject-derived, `A` for AActor-derived, `F` for plain structs, `E` for enums, `I` for interfaces.
- Classes declare inheritance with `:`: `class UEventBus : UGameInstanceSubsystem`.

## Class declaration patterns

```as
UCLASS(Abstract, BlueprintType)
class UCozyEvent : UObject
{
    UPROPERTY(BlueprintReadOnly)
    FName EventType;
}
```

Common `UCLASS` keywords:
- `Abstract` — class can't be instantiated directly.
- `BlueprintType` — class is exposed to Blueprint as a usable type.
- `Blueprintable` — Blueprints can subclass it.
- `NotPlaceable` — class can't be placed in editor world (typical for components).
- `EditInlineNew` — class instances editable inline in details panel.

## Property declarations

```as
UPROPERTY(BlueprintReadOnly)
FName EventType;

UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Cozy|Tuning")
float DispatchTimeoutSeconds = 0.5;

UPROPERTY()
private TMap<FName, TArray<FCozyEventDelegate>> Subscribers;
```

Common `UPROPERTY` keywords:
- `EditAnywhere` / `EditDefaultsOnly` / `EditInstanceOnly` — editor visibility.
- `BlueprintReadOnly` / `BlueprintReadWrite` — Blueprint access level.
- `VisibleAnywhere` — read-only in editor.
- `Category = "Foo|Bar"` — organizes the details panel; pipes denote nesting.
- `Replicated` / `ReplicatedUsing=OnRep_X` — replication; pair with `OnRep_*` callbacks.

`private` and `protected` are AS-language keywords; they enforce visibility within the class but do NOT remove Blueprint exposure if `BlueprintReadOnly` is also set. Use `UPROPERTY()` (no metadata) for fields that should be UE-tracked but not editor- or BP-exposed.

## Function declarations

```as
UFUNCTION(BlueprintCallable)
void Subscribe(FName EventType, FCozyEventDelegate Callback)
{
    // ...
}

UFUNCTION(BlueprintOverride)
void Initialize(FSubsystemCollectionBase& Collection)
{
    // overrides UEngineSubsystem::Initialize
}

UFUNCTION(BlueprintPure)
int GetCurrentRecursionDepth() const
{
    return CurrentRecursionDepth;
}
```

Common `UFUNCTION` keywords:
- `BlueprintCallable` — invokable from BP graphs.
- `BlueprintOverride` — overrides a UE virtual method (Initialize, Tick, Deinitialize, etc.).
- `BlueprintPure` — node has no exec pin; treated as a pure expression.
- `BlueprintImplementableEvent` — declared in AS, implemented in BP.
- `Server` / `Client` / `NetMulticast` + `Reliable` / `Unreliable` — RPC.
- `Category = "Foo"` — node placement in Blueprint context menus.

## Delegates

```as
delegate void FCozyEventDelegate(UCozyEvent Event);

UPROPERTY(BlueprintAssignable)
FCozyEventDelegate OnEventDispatched;

void Fire()
{
    OnEventDispatched.Broadcast(SomeEvent);
}

void Bind()
{
    OnEventDispatched.AddUFunction(this, n"HandleEvent");
}
```

- `BlueprintAssignable` — BP can bind handlers.
- `Broadcast()` fires all subscribers; `ExecuteIfBound()` is the safe-invoke for single-cast.
- Bind by `n"FunctionName"` literal (n-prefixed name literal) or by direct reference where supported.
- Always prefer `ExecuteIfBound` over `Execute` for single-cast delegates that may not be bound at dispatch time.

## Subsystems (the canonical persistent-state pattern)

```as
class UEventBus : UGameInstanceSubsystem
{
    UFUNCTION(BlueprintOverride)
    void Initialize(FSubsystemCollectionBase& Collection) { /* setup */ }

    UFUNCTION(BlueprintOverride)
    void Deinitialize() { /* teardown */ }

    UFUNCTION(BlueprintPure)
    static UEventBus Get(UObject WorldContextObject)
    {
        UGameInstance GI = UGameplayStatics::GetGameInstance(WorldContextObject);
        if (GI == nullptr) return nullptr;
        return Cast<UEventBus>(GI.GetSubsystem(UEventBus::StaticClass()));
    }
}
```

Subsystem types and their lifetimes:
- `UEngineSubsystem` — engine lifetime; survives map travel and PIE.
- `UGameInstanceSubsystem` — game-instance lifetime; one per running app, cleared when game ends.
- `UWorldSubsystem` — world lifetime; instantiated per loaded world.
- `ULocalPlayerSubsystem` — per local player.

The `static Get(UObject WorldContextObject)` accessor is the standard idiom — pass `this` from any actor/component or `GetWorld()` from contexts that have one.

## Type system essentials

| AS type | UE underlying | Notes |
|---|---|---|
| `FName` | UE FName | Interned string ID. Cheap copies, hash-friendly. Compare with `==` not string-equals. `IsNone()` to test empty. |
| `FString` | UE FString | Mutable string; allocates. Use `FName` for identifiers, `FText` for player-facing localizable. |
| `FText` | UE FText | Localizable. `FText::FromString("...")` for non-localized; `LOCTEXT` macro in C++ context. |
| `int` / `int32` | int32 | Signed 32-bit. |
| `int64` | int64 | Signed 64-bit. |
| `float` / `double` | UE float types | Default to `float` unless precision required. |
| `bool` | bool | |
| `TArray<T>` | UE TArray | Dynamic array. `Add`, `Remove`, `Num`, `Contains`, `[i]`. |
| `TMap<K, V>` | UE TMap | Hash map. `Add`, `Contains`, `Remove`, `Num`, `[key]`. |
| `TSet<T>` | UE TSet | Hash set. |
| `FVector` / `FRotator` / `FQuat` | UE math types | Standard. |

`nullptr` is the null literal. `UObject` references are pointer-like but use `.` not `->` for member access.

## Casting

```as
UEventBus Bus = Cast<UEventBus>(GI.GetSubsystem(UEventBus::StaticClass()));
if (Bus != nullptr) { /* use */ }
```

`Cast<T>(x)` returns `nullptr` on failure (does not throw). Always null-check after cast.

## Defensive invocation

Prefer `ExecuteIfBound` over `Execute` for delegates that may be unbound:

```as
Callback.ExecuteIfBound(Event);  // safe if Callback is unbound
```

For multi-cast delegates, `Broadcast` is always safe (it iterates the bound list).

## Hot-reload behavior

- Saving an `.as` file in the editor with the AS plugin enabled triggers a hot recompile.
- Class layout changes (adding/removing properties) generally hot-reload cleanly; breaking changes (renaming classes, changing inheritance) may require an editor restart.
- Subsystem state survives hot reload only if the class identity is preserved. Renaming or moving a subsystem class across files clears state on next reload.
- Compile errors block hot reload; the previously-compiled version remains active until the error is fixed.
- Hot reload is editor-only. Packaged builds compile AS to its final form at cook time.

## Editor binding patterns

- BP can subclass AS `Blueprintable` classes; AS can override BP-exposed methods marked `BlueprintImplementableEvent`.
- BP graphs can call any `UFUNCTION(BlueprintCallable)` AS method.
- AS classes appearing as components: declare `UCLASS()` deriving from `UActorComponent` or specialized component classes.
- Editor-tickable classes use `UFUNCTION(BlueprintOverride) void Tick(float DeltaSeconds)` after the parent class enables ticking.

## Common pitfalls

- **Forgetting `UFUNCTION` on a method called from BP** — the method exists in AS but is invisible to BP. Always tag BP-exposed methods.
- **`UPROPERTY()` missing on a UObject reference field** — UE's GC will not track the reference; the object can be garbage-collected unexpectedly. Always tag UObject references.
- **Using `==` for string content comparison** — `FString a = "x"; FString b = "x"; a == b` works but is content-comparison every call. Prefer `FName` for identifiers.
- **Direct delegate `Execute()` on potentially-unbound delegate** — crashes if unbound. Use `ExecuteIfBound` for single-cast or `Broadcast` for multi-cast.
- **Returning a local UObject without tracking** — local UObject creation via `NewObject` requires the caller to keep a reference; otherwise GC reclaims.
- **Forgetting `WorldContextObject` parameter** — many UE static helpers (`GetGameInstance`, `GetPlayerController`, etc.) need a world context. Pass `this` from an actor/component or `GetWorld()` if available.
- **`BlueprintPure` on a side-effecting function** — pure nodes execute every time their output is read; hidden side effects accumulate.
- **Using `TArray` indexing without bounds check** — `arr[i]` on out-of-range crashes. `arr.IsValidIndex(i)` to test.
- **Subsystem `Initialize` doing heavy work** — `Initialize` runs at engine/game-instance startup. Defer heavy setup to lazy-init or a separate `Start()` method.
- **Replicated property without `UPROPERTY(Replicated)`** — value never crosses the wire even if the class is replicated. Pair with `GetLifetimeReplicatedProps` if needed.
- **Hot reload after class rename** — subsystem state lost; old subscribers point to the dead class. Restart editor after structural changes.

## BP↔AS interop boundary

- Logic that needs strong typing, refactor-safety, or version control of the implementation → AS.
- Logic that benefits from designer iteration in the editor (visuals, blueprint event graphs, content authoring) → BP, calling AS-defined classes/methods.
- Data-only assets (Curves, DataTables, Datasets) → still BP-side; AS reads them via `UFUNCTION(BlueprintCallable)` getters.
- Don't reimplement BP-native helpers in AS unless you need cross-platform AS testing — use the BP nodes via AS's UE-binding layer.

## Performance notes

- AS function-call overhead is higher than C++ but lower than BP. For per-frame hot paths, prefer C++ if the engine version supports mixed AS+C++; otherwise keep AS hot paths simple and avoid `TMap` lookups in tick.
- AS `TArray` operations are the same cost as C++ TArray operations.
- Delegate broadcast cost scales linearly with subscribers. Snapshot-iterate to allow safe Unsubscribe during dispatch (see `EventBus.as` reference implementation).

## Authoritative external references

When the knowledge here doesn't cover a case:

- Hazelight's angelscript-ue4 GitHub repo and its docs/ folder.
- UE5 C++ documentation for the underlying UFUNCTION/UPROPERTY semantics — they apply identically.
- The project's own AS code under `<Project>/Script/` as a reference for project-specific conventions.

## Maintenance

This file's `version-pinned-to:` frontmatter declares the engine + plugin version it accurately covers. The `prometheus:harness-health` skill flags this file when its pin is older than the consuming project's declared engine version. Update by reviewing release notes between versions and revising patterns that changed.
