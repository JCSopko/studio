---
paths:
  - "tests/**"
---

# Test Standards

- Test naming: `test_[system]_[scenario]_[expected_result]` pattern
- Every test must have a clear arrange/act/assert structure
- Unit tests must not depend on external state (filesystem, network, database)
- Integration tests must clean up after themselves
- Performance tests must specify acceptable thresholds and fail if exceeded
- Test data must be defined in the test or in dedicated fixtures, never shared mutable state
- Mock external dependencies — tests should be fast and deterministic
- Every bug fix must have a regression test that would have caught the original bug

## Examples (UE5 Automation Spec / Angelscript)

**Correct** (proper naming + Arrange/Act/Assert):

```cpp
// UE5 Automation Spec example — test_health_system_take_damage_reduces_health
BEGIN_DEFINE_SPEC(FHealthSystemSpec,
    "Studio.HealthSystem.TakeDamage.ReducesHealth",
    EAutomationTestFlags::ApplicationContextMask | EAutomationTestFlags::ProductFilter)
END_DEFINE_SPEC(FHealthSystemSpec)

void FHealthSystemSpec::Define()
{
    Describe("HealthSystem", [this]() {
        It("reduces health when TakeDamage is called", [this]() {
            // Arrange
            UHealthComponent* Health = NewObject<UHealthComponent>();
            Health->MaxHealth = 100;
            Health->CurrentHealth = 100;

            // Act
            Health->TakeDamage(25);

            // Assert
            TestEqual("CurrentHealth after 25 damage", Health->CurrentHealth, 75);
        });
    });
}
```

**Incorrect**:

```cpp
// VIOLATION: no descriptive name; no Arrange/Act split; imprecise assertion
void TestSomething()
{
    UHealthComponent* H = NewObject<UHealthComponent>();
    H->TakeDamage(25);
    check(H->CurrentHealth < 100);  // VIOLATION: imprecise — passes for any value < 100
}
```

The naming pattern is engine-agnostic; the framework varies (UE Automation Spec / NUnit / GdUnit4 / pytest). The Arrange/Act/Assert structure and the regression-test-for-every-bug rule apply universally.
