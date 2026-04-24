# Codex Adapter

This document explains how to map Claw Growth Coach into Codex.

## Recommended files

- skill file:
  `${CODEX_HOME:-~/.codex}/skills/claw-growth-coach/SKILL.md`
- memory files:
  `${CODEX_HOME:-~/.codex}/memories/PROFILE.md`
  `${CODEX_HOME:-~/.codex}/memories/ACTIVE.md`
  `${CODEX_HOME:-~/.codex}/memories/CLAW_GROWTH_COACH.md`

## Codex-specific notes

- Codex supports skill discovery via `SKILL.md`
- Codex can use thread heartbeat automations
- Codex can also be woken by a local Python scheduler

## Suggested scheduling

- default window: `08:00-22:00`
- default frequency: hourly
- prefer lightweight check-ins over long coaching responses

## Local Python scheduler

A Codex-specific example scheduler can call:

```bash
codex exec resume <thread_id> "<prompt>"
```

This is an adapter implementation, not part of the core coach spec.
