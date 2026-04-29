---
paths:
  - "src/gameplay/**"
---

# Gameplay Code Rules

- ALL gameplay values MUST come from external config/data files, NEVER hardcoded
- Use delta time for ALL time-dependent calculations (frame-rate independence)
- NO direct references to UI code — use events/signals for cross-system communication
- Every gameplay system must implement a clear interface
- State machines must have explicit transition tables with documented states
- Write unit tests for all gameplay logic — separate logic from presentation
- Document which design doc each feature implements in code comments
- No static singletons for game state — use dependency injection

## Examples (Angelscript / UE5)

**Correct** (data-driven, delta-time-respecting):

```as
UPROPERTY(EditDefaultsOnly, Category = "Combat")
UDataTable CombatTuning;

UPROPERTY(EditDefaultsOnly, Category = "Movement")
float MovementSpeedScalar = 1.0;

UFUNCTION(BlueprintOverride)
void Tick(float DeltaSeconds)
{
    FCombatTuningRow Row;
    if (CombatTuning.FindRow(n"BaseDamage", Row))
    {
        float Damage = Row.Damage;  // value from data
    }
    float Distance = MovementSpeed * MovementSpeedScalar * DeltaSeconds;
}
```

**Incorrect** (hardcoded gameplay numbers, no delta):

```as
UFUNCTION(BlueprintOverride)
void Tick(float DeltaSeconds)
{
    float Damage = 25.0;       // VIOLATION: hardcoded gameplay value
    float Distance = 5.0;      // VIOLATION: no DeltaSeconds, no data source
    Move(GetActorForwardVector() * Distance);
}
```

The principle is engine-agnostic. UE5 patterns: DataTables for tuning rows, DataAssets for typed configs, UPROPERTY exposed scalars for designer-tunable variants. Same rules in C++ and Angelscript.
