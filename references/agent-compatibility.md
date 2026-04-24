# Agent Compatibility Guide

Claw Growth Coach is designed as a host-agnostic coaching spec.

## Minimum host requirements

A host agent should ideally support:

1. system prompt or skill injection
2. multi-turn conversation
3. optional persistent memory
4. optional recurring wakeup

The coach still works if only the first two are available.

## Capability mapping

Map host capabilities to the coach spec like this:

- `system prompt / skill`
  load `SKILL.md` or translate it into the host's instruction format

- `persistent memory`
  store `PROFILE`, `ACTIVE`, and `CLAW_GROWTH_COACH` in the host's preferred memory store

- `scheduler / automation`
  trigger a lightweight check-in between `08:00-22:00`, hourly by default

- `thread/session resume`
  wake the same user conversation when possible

## Portability rules

- avoid hardcoded absolute paths in the core spec
- treat memory root as `<agent_memory_root>`
- keep platform commands out of `SKILL.md`
- place platform behavior in adapter docs

## If a host lacks memory

Fallback behavior:

- infer context from the current session
- ask a compact reconfirmation when needed
- do not pretend past memory exists

## If a host lacks scheduling

Fallback behavior:

- only run on explicit user invocation
- still keep the scheduled-check-in response format available for manual use

## If a host lacks file I/O

Fallback behavior:

- use the host profile store, database, KV store, or thread metadata
- preserve the same conceptual schema

## Implementation principle

The coaching behavior is the product.
The platform is only the transport layer.
