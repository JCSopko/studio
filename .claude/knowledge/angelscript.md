---
name: angelscript
description: "Pointer to canonical AngelScript knowledge — graduated to Prometheus plugin 2026-05-18."
type: knowledge
version-pinned-to: "n/a (pointer; canonical is prometheus:angelscript)"
last-reviewed: 2026-05-18
last-reviewer: iji
canonical-sources:
  - "prometheus:angelscript skill (C:/Dev/Prometheus/claude-plugin/skills/angelscript/SKILL.md)"
---

# Angelscript — see canonical at prometheus:angelscript

This file's content has been graduated to the **prometheus:angelscript** plugin skill as of 2026-05-18.

**Canonical location:** `C:/Dev/Prometheus/claude-plugin/skills/angelscript/SKILL.md`

When `studio:programming` (or any Studio context) needs AngelScript knowledge, load `prometheus:angelscript` — Prometheus is registered in Studio's marketplace per `studio/.claude/settings.json`. The merged canonical includes Studio's prior comprehensive content (subsystem traps, float-default landmine, casting rules, networking patterns) plus Iji-side additions from 2026-05-18 (scan-root constraint as critical AI Blind Spot + Print-vs-Log test rule + Testing section with BMO-graduated discipline + Cozy precedent reference).

## Why graduated

Three problems with the prior dual-canonical state:
1. Studio's file claimed Iji canonical in frontmatter, but Studio was 4x more comprehensive — inversion of authority
2. Updates landed in either Iji or Studio (or both) inconsistently — drift risk
3. Future consumers (TogetherUE5, ReleaseUE5, SEED, Jon-onboarded games) need a single source of truth

Prometheus's claude-plugin architecture (namespace-prefixed skills + marketplace distribution) is the natural home. Consumers reference `prometheus:angelscript` and get canonical content via plugin propagation.

## Maintenance

Future AS knowledge additions: edit `prometheus:angelscript` SKILL.md. Studio doesn't carry AS-specific content anymore. This pointer file remains to preserve the `studio:angelscript` reference path for any agent that still cites it; the path-scoped rule that loaded this file should be updated to load `prometheus:angelscript` instead.

## History

- **2026-04-29** — Original Studio canonical authored, replacing Iji's lighter skill at the time
- **2026-05-18** — Graduated to Prometheus (`prometheus:angelscript`). Merged Studio content + Iji's recent additions from F-027 incident. This file becomes a pointer.
