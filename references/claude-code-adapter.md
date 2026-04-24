# Claude Code Adapter

This document explains how to map Claw Growth Coach into Claude Code or similar terminal agents.

## Suggested integration pattern

1. load the coach spec as a reusable system prompt, instruction block, or local skill file
2. persist memory in a host-controlled directory or memory store
3. on each new session, preload the coach memory if the host supports it
4. use any available automation layer to trigger lightweight hourly check-ins between `08:00-22:00`

## Memory recommendation

Use a host-specific memory root such as:

- `<agent_memory_root>/PROFILE.md`
- `<agent_memory_root>/ACTIVE.md`
- `<agent_memory_root>/CLAW_GROWTH_COACH.md`

If the host does not support automatic memory loading, the runner or wrapper script can inject the contents at session start.

## Scheduling recommendation

Claude Code itself may not provide the same heartbeat primitive as other hosts.
In that case, use one of these adapter patterns:

- OS-level scheduler
- Python timer
- external workflow runner
- message bridge / automation wrapper

## Porting rule

Keep the coaching logic unchanged.
Only replace the memory transport and wakeup mechanism.
