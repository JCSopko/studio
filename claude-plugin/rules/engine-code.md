---
paths:
  - "src/core/**"
---

# Engine Code Rules

- ZERO allocations in hot paths (update loops, rendering, physics) — pre-allocate, pool, reuse
- All engine APIs must be thread-safe OR explicitly documented as single-thread-only
- Profile before AND after every optimization — document the measured numbers
- Engine code must NEVER depend on gameplay code (strict dependency direction: engine <- gameplay)
- Every public API must have usage examples in its doc comment
- Changes to public interfaces require a deprecation period and migration guide
- Use RAII / deterministic cleanup for all resources
- All engine systems must support graceful degradation
- Before writing engine API code, consult `docs/engine-reference/` for the current engine version and verify APIs against the reference docs

## Examples (Angelscript / UE5)

**Correct** (zero-alloc hot path — pre-allocated container reused each tick):

```as
// Pre-allocated cache reused per tick
private TArray<AActor> NearbyCache;

UFUNCTION(BlueprintOverride)
void Tick(float DeltaSeconds)
{
    NearbyCache.Empty(NearbyCache.Max());  // reset count, retain capacity
    SpatialGrid.QueryRadius(GetActorLocation(), QueryRadius, NearbyCache);
}
```

**Incorrect** (allocating per tick):

```as
UFUNCTION(BlueprintOverride)
void Tick(float DeltaSeconds)
{
    TArray<AActor> Nearby;  // VIOLATION: allocates every tick
    UGameplayStatics::GetAllActorsOfClass(this, AEnemy::StaticClass(), Nearby);  // VIOLATION: world enumeration every tick
}
```

The principle is engine-agnostic: pre-allocate, pool, reuse. Examples shown in Angelscript; equivalent C++ patterns apply identically with `TArray.Reset()` (count-only reset) over `TArray.Empty()` (allocation-shrinking).
