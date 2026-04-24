# Claw Growth Coach

Claw Growth Coach is an agent-agnostic growth coaching spec for AI assistants.

Its purpose is not to generate generic motivation, but to help users build a repeatable growth system with:

- onboarding
- memory
- action loops
- reflection
- lightweight recurring check-ins

It is designed to work across different hosts such as Codex, Claude Code, OpenAI Agents, or any environment that can load structured instructions and optionally persist memory.

## What It Does

Claw Growth Coach helps users:

- clarify long-term direction and current focus
- turn vague self-improvement intent into concrete action
- diagnose where a growth loop is breaking
- recover momentum during procrastination, drift, or low-energy states
- improve learning transfer, focus quality, and decision quality
- maintain continuity across sessions through stable memory

## Core Model

The coaching model assumes growth is a system rather than a burst of motivation.

Its default loop is:

`input -> structured understanding -> small practice -> real attempt -> feedback -> reflection -> second attempt -> identity reinforcement`

When a user is stuck, the coach tries to identify which part of the loop is broken and repair it with the smallest useful intervention.

## Core Capabilities

### 1. Onboarding

On first meaningful use, the coach builds a baseline profile instead of jumping straight into generic advice.

Suggested fields include:

- `north_star`
- `current_focus`
- `core_bottleneck`
- `strengths`
- `failure_patterns`
- `learning_domains`
- `time_budget`
- `energy_rhythm`
- `preferred_language`
- `support_style`
- `check_in_style`
- `non_negotiables`
- `definition_of_success`

The coach should summarize the baseline back to the user for confirmation before writing durable memory.

### 2. Memory

The spec supports persistent memory without requiring a specific platform.

Recommended memory layout:

- `<agent_memory_root>/PROFILE.md`
- `<agent_memory_root>/ACTIVE.md`
- `<agent_memory_root>/CLAW_GROWTH_COACH.md`

The coach-specific memory file should capture stable context such as goals, bottlenecks, working preferences, constraints, and current growth experiments.

### 3. Language Adaptation

The default language should follow the user's first meaningful interaction.

Rules:

- use stored `preferred_language` if already known
- otherwise infer from the user's first meaningful reply
- if the coach must speak before any user reply exists, default to English
- update memory if the user later explicitly switches languages

### 4. Loop Diagnosis

The coach identifies whether the user is mainly blocked by:

- input overload
- weak understanding
- missing action design
- avoidance of real attempts
- weak or delayed feedback
- missing reflection
- no second iteration
- no identity integration

### 5. Action Compression

The coach converts large goals into low-resistance next steps, such as:

- today's smallest move
- the next important action
- the next one-hour action
- a low-energy fallback action

### 6. Flow Recovery

When a user cannot focus, the coach checks whether the issue is:

- challenge too high
- challenge too low
- unclear target
- delayed feedback
- low energy
- too many distractions

Then it redesigns the task to make progress easier.

### 7. Decision Support

For important decisions, the coach helps:

- widen the option set
- test assumptions against reality
- separate short-term emotion from long-term values
- design a low-risk experiment when possible

### 8. Recurring Check-Ins

If the host platform supports scheduling, the coach can run lightweight recurring check-ins.

Default cadence:

- hourly
- between `08:00-22:00` in the user's local time

Scheduled check-ins should stay short and execution-oriented.

## Design Principles

This repository keeps the core coaching behavior separate from host-specific implementation details.

Principles:

- keep the core spec platform-neutral
- use abstract memory paths such as `<agent_memory_root>`
- keep platform commands out of the core prompt
- move host-specific behavior into adapter documents

## Repository Structure

```text
claw-growth-coach/
├── README.md
├── SKILL.md
├── agents/
│   └── openai.yaml
└── references/
    ├── agent-compatibility.md
    ├── claude-code-adapter.md
    ├── codex-adapter.md
    └── memory-schema.md
```

## Files

- `SKILL.md`
  Core host-agnostic coaching spec

- `agents/openai.yaml`
  Example agent metadata for hosts that support this format

- `references/memory-schema.md`
  Suggested structure for coach memory

- `references/agent-compatibility.md`
  General guidance for mapping the coach into different agent hosts

- `references/codex-adapter.md`
  Codex-specific adapter notes

- `references/claude-code-adapter.md`
  Claude Code-specific adapter notes

## Using This Project

You can use this repository in at least three ways:

1. Load `SKILL.md` directly as a reusable instruction set in your agent host
2. Adapt it into your own prompt/skill/plugin format
3. Use the adapter docs as a starting point for host-specific integrations

## Memory Schema

The main coach memory file is expected to be:

- `<agent_memory_root>/CLAW_GROWTH_COACH.md`

Suggested sections include:

- Initialization Status
- North Star
- Current Focus
- Current Project
- Main Bottlenecks
- Strengths
- Failure Patterns
- Learning Domains
- Time Budget
- Energy Rhythm
- Preferred Language
- Preferred Coaching Style
- Check-In Preferences
- Recurring Cadence
- Constraints
- Success Definition
- Active Experiments
- Reusable Notes

See [`references/memory-schema.md`](./references/memory-schema.md) for the full template.

## Adapters

The core spec is intentionally host-neutral.

Host-specific notes live in:

- [`references/agent-compatibility.md`](./references/agent-compatibility.md)
- [`references/codex-adapter.md`](./references/codex-adapter.md)
- [`references/claude-code-adapter.md`](./references/claude-code-adapter.md)
