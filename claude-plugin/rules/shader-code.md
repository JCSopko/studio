---
paths:
  - "assets/shaders/**"
---

# Shader Code Standards

All shader files in `assets/shaders/` must follow these standards to maintain
visual quality, performance, and cross-platform compatibility.

## Naming Conventions

UE5 conventions:

- `M_<Category>_<Name>` for materials (e.g., `M_Env_Water`)
- `MI_<Category>_<Name>` for material instances (e.g., `MI_Env_Water_Calm`)
- `MF_<Category>_<Name>` for material functions (e.g., `MF_Env_Caustics`)
- `NS_<Category>_<Name>` for Niagara systems (VFX)
- For custom HLSL shaders shipped via plugin or `.usf`: `<Category>_<Name>.usf`

Use descriptive names that indicate the material's purpose; the category prefix groups related shaders in the content browser.

## Code Quality
- All material parameters must have descriptive names and appropriate parameter info (Display Name, Description, Group)
- Group related parameters via the `Group` field in parameter metadata (e.g., "Surface", "Lighting", "Animation")
- Comment non-obvious calculations (especially math-heavy sections)
- No magic numbers — use named constants or documented uniform values
- Include authorship and purpose comment at the top of each shader file

## Performance Requirements
- Document the target platform and complexity budget for each shader
- Use appropriate precision: `half`/`mediump` on mobile where full precision isn't needed
- Minimize texture samples in fragment shaders
- Avoid dynamic branching in fragment shaders — use `step()`, `mix()`, `smoothstep()`
- No texture reads inside loops
- Two-pass approach for blur effects (horizontal then vertical)

## Cross-Platform
- Test shaders on minimum spec target hardware
- Provide quality-tier variants where the platform target spans (e.g., Mobile / Console / High-End PC)
- Document the rendering path the shader targets (Forward / Deferred Shading / Mobile Forward) — UE5 renderer choice affects which features are usable
- Do not mix shaders authored for different rendering paths in the same directory without explicit subdirectory naming

## Variant Management
- Minimize shader variants — each variant is a separate compiled shader
- Document all keywords/variants and their purpose
- Use feature stripping where possible to reduce build size
- Log and monitor total variant count per shader
