# ClawCoach — FFXIV AI Growth Coach

[![Rust](https://img.shields.io/badge/Rust-1.70%2B-orange?logo=rust)](https://www.rust-lang.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Build](https://img.shields.io/badge/build-passing-brightgreen)]()
[![Platform](https://img.shields.io/badge/platform-Windows%20%7C%20Linux%20%7C%20macOS-blue)]()

**ClawCoach** is a Rust-based AI growth coach for *Final Fantasy XIV (FFXIV)*. It ingests game logs (combat, gathering, crafting), evaluates your performance against extensible skill benchmarks, stores results in SQLite, and exposes actionable metrics through a web dashboard and REST API.

---

## Table of Contents

- [Features](#features)
- [Architecture](#architecture)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [Configuration](#configuration)
- [Log Parsing](#log-parsing)
- [Skill Evaluation System](#skill-evaluation-system)
- [Storage Layer](#storage-layer)
- [Web Dashboard & API](#web-dashboard--api)
- [Project Structure](#project-structure)
- [Contributing](#contributing)
- [License](#license)

---

## Features

- **High-throughput log parsing** — Native Rust parsers for Combatant, Gatherer, and Crafter logs processing **30,000–50,000 lines/sec**.
- **Trait-based skill evaluation** — Extensible `Skill` trait lets you define custom benchmarks (DPS thresholds, perception targets, rotation patterns). Skills are executed by a runner engine and results are persisted automatically.
- **SQLite persistence** — Sessions, parsed events, and skill results stored locally with analysis queries for DPS trends, percentile rankings, and improvement tracking.
- **Web dashboard** — Built-in HTML dashboard served by a Warp-based HTTP server for visualizing metrics and trends.
- **REST API** — JSON endpoints for programmatic access to session data, skill results, and trend analysis.
- **AI-powered coaching** — Claude API integration generates natural-language feedback and growth recommendations based on your parsed data.
- **Configurable memory system** — Persistent AI memory with JSON schema validation tracks progress across sessions.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        ClawCoach Core                           │
│                                                                 │
│  ┌──────────────┐   ┌──────────────────┐   ┌─────────────────┐ │
│  │  Log Parser  │──▶│ Skill Evaluation │──▶│    Storage      │ │
│  │  30-50k l/s  │   │   Runner Engine  │   │   (SQLite)      │ │
│  └──────────────┘   └──────────────────┘   └────────┬────────┘ │
│         │                    │                      │          │
│         ▼                    ▼                      ▼          │
│  ┌──────────────┐   ┌──────────────────┐   ┌─────────────────┐ │
│  │ FFXIV Logs   │   │  Skill Benchmarks│   │  Skill Results  │ │
│  │ Combat/Gather│   │  (SKILL.md)      │   │  Session Data   │ │
│  │ Craft        │   │  Extensible      │   │  Trend Queries  │ │
│  └──────────────┘   └──────────────────┘   └────────┬────────┘ │
│                                                     │          │
│                                        ┌────────────▼────────┐ │
│                                        │   Web Server (Warp) │ │
│                                        │  ┌──────┐ ┌──────┐  │ │
│                                        │  │HTML  │ │REST  │  │ │
│                                        │  │Dash  │ │API   │  │ │
│                                        │  └──────┘ └──────┘  │ │
│                                        └─────────────────────┘ │
│                                                     │          │
│                                        ┌────────────▼────────┐ │
│                                        │   AI Coach (Claude) │ │
│                                        │  Feedback & Advice  │ │
│                                        └─────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

---

## Installation

### Prerequisites

- **Rust** 1.70 or later ([install via rustup](https://rustup.rs/))
- A valid FFXIV game log file (typically found in your game's log directory)
- An **Anthropic API key** for the AI coaching features (get one at [console.anthropic.com](https://console.anthropic.com/))

### Build from Source

```bash
git clone https://github.com/your-org/claw-growth-coach.git
cd claw-growth-coach
cargo build --release
```

The compiled binary will be located at `target/release/claw-growth-coach.exe` (Windows) or `target/release/claw-growth-coach` (Unix).

### Dependencies

SQLite is bundled via the `rusqlite` crate with the `bundled` feature — no external database installation is required.

---

## Quick Start

1. **Set your Anthropic API key** in the config file or environment variable:
   ```bash
   export CLAW_ANTHROPIC_API_KEY="your-api-key-here"
   ```

2. **Configure your FFXIV log path** in `agents/openai.yaml`:
   ```yaml
   log_path: "C:/Users/You/Documents/FFXIV/CombatLog.log"
   ```

3. **Run the coach**:
   ```bash
   cargo run --release
   ```

4. **Open the dashboard** in your browser at `http://localhost:3000` (or your configured port).

---

## Configuration

ClawCoach is configured via `agents/openai.yaml`:

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `api_key` | `string` | — | Anthropic Claude API key |
| `api_secret` | `string` | — | Optional API secret |
| `log_path` | `string` | — | Absolute path to your FFXIV game log |
| `db_path` | `string` | `clawcoach.db` | SQLite database file path |
| `port` | `u16` | `3000` | Web dashboard & API server port |
| `host` | `string` | `127.0.0.1` | Server bind address |

Example:

```yaml
api_key: "sk-ant-your-key-here"
log_path: "C:/Users/You/Documents/FFXIV/CombatLog.log"
db_path: "clawcoach.db"
port: 3000
host: "127.0.0.1"
```

---

## Log Parsing

ClawCoach includes high-performance native Rust parsers for three FFXIV log types:

| Parser | Data Type | Key Metrics |
|--------|-----------|-------------|
| **Combatant** | Combat encounters | DPS, hit rate, crit rate, direct hit rate, damage taken, healing |
| **Gatherer** | Gathering sessions | Gather attempts, success rate, perception-based yields, node types |
| **Crafter** | Crafting attempts | Craft quality, durability management, step efficiency, success/fail |

### Performance

The parsers are zero-copy where possible and process **30,000–50,000 lines per second** on modern hardware. They handle:

- Combat encounter detection and grouping
- Gathering session aggregation
- Craft attempt lifecycle tracking
- Automatic timestamp parsing and normalization

---

## Skill Evaluation System

The skill evaluation system is the core of ClawCoach's coaching capabilities. It is built around a trait-based architecture defined in [`SKILL.md`](SKILL.md).

### How It Works

1. **Define a skill** — Each skill implements the `Skill` trait (see `src/skills/base.rs`), specifying:
   - A unique `skill_id` and human-readable `name`
   - Benchmark targets (e.g., DPS ≥ 8000, perception ≥ 400)
   - Evaluation logic that runs against parsed session data

2. **Register & run** — Skills are registered with the `SkillRunner` (`src/skills/runner.rs`), which:
   - Loads applicable skills for the player's current job/role
   - Executes each skill's evaluation against session data
   - Stores results (pass/fail, metrics, percentile ranking) in SQLite

3. **Track progress** — Results are persisted per-session, enabling:
   - Historical trend analysis
   - Improvement tracking over time
   - Percentile comparison against benchmarks

### Extending Skills

New skills can be added by implementing the `LogSkill` trait. See [`SKILL.md`](SKILL.md) for the full protocol definition and examples.

---

## Storage Layer

ClawCoach uses **SQLite** for all persistent storage, managed via the `rusqlite` crate with the `bundled` feature.

### Schema Overview

| Table | Purpose |
|-------|---------|
| `sessions` | Player sessions with metadata (job, timestamp, duration) |
| `combat_events` | Individual combat actions parsed from logs |
| `gather_events` | Gathering session events |
| `craft_events` | Crafting attempt events |
| `skill_results` | Skill evaluation results per session |

### Analysis Queries

The storage layer includes built-in queries for:
- **DPS trends** over time
- **Percentile rankings** against defined benchmarks
- **Improvement tracking** between sessions
- **Aggregated statistics** per job/role

---

## Web Dashboard & API

ClawCoach ships with a built-in Warp-based HTTP server that serves both an HTML dashboard and a JSON REST API.

### Dashboard

Access the dashboard at `http://<host>:<port>/` to view:
- Recent session summaries
- Skill evaluation results
- DPS and performance trends
- AI coaching feedback

### REST API

All endpoints return JSON. The full schema is available in [`references/api-schema.json`](references/api-schema.json).

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/sessions` | List all recorded sessions |
| `GET` | `/api/sessions/:id` | Get a specific session with full event data |
| `GET` | `/api/skills` | List all available skills |
| `GET` | `/api/skills/results` | Get skill evaluation results |
| `GET` | `/api/skills/trends` | Get skill trend analysis over time |
| `POST` | `/api/analyze` | Trigger a new log analysis run |
| `GET` | `/api/stats` | Get aggregated player statistics |

### Error Responses

All API errors follow a consistent format:

```json
{
  "error": "Session not found",
  "code": 404
}
```

---

## Project Structure

```
claw-growth-coach/
├── src/
│   ├── main.rs              # Binary entry point
│   ├── lib.rs               # Library root — re-exports all modules
│   ├── config.rs            # Configuration loading & constants
│   ├── error.rs             # Custom error types (ClawError)
│   ├── models/              # Core data models
│   │   ├── mod.rs
│   │   ├── combatant.rs     # Combat data structures & DPS analysis
│   │   ├── gatherer.rs      # Gathering data & perception metrics
│   │   └── crafter.rs       # Crafting data structures
│   ├── log_parser/          # FFXIV log ingestion
│   │   ├── mod.rs
│   │   └── parser.rs        # High-throughput native parsers
│   ├── skills/              # Skill evaluation system
│   │   ├── mod.rs
│   │   ├── base.rs          # Skill trait & LogSkill implementation
│   │   └── runner.rs        # Skill execution engine
│   ├── storage/             # SQLite persistence layer
│   │   ├── mod.rs
│   │   └── db.rs            # Schema, CRUD, and analysis queries
│   └── web/                 # Web server
│       ├── mod.rs
│       ├── dashboard.rs     # HTML dashboard templates
│       └── api.rs           # REST API routes (warp)
├── agents/
│   └── openai.yaml          # Configuration file
├── references/              # Design documentation
│   ├── api-schema.json      # OpenAPI-style API schema
│   ├── design-doc.md        # System architecture design
│   ├── memory-schema.md     # AI memory system specification
│   ├── phase1-3-summary.md  # Development phase summaries
│   └── ui-prototype.html    # Dashboard UI prototype
├── SKILL.md                 # Skill protocol definition
├── Cargo.toml               # Rust package manifest
├── Cargo.lock               # Dependency lock file
└── README.md                # This file
```

---

## Contributing

Contributions are welcome! Here's how to get started:

1. **Fork** the repository and create your feature branch:
   ```bash
   git checkout -b feature/my-awesome-skill
   ```

2. **Make your changes** — follow the existing code style and ensure `cargo clippy` passes without warnings.

3. **Test your changes**:
   ```bash
   cargo test
   ```

4. **Add new skills** (optional) — implement the `LogSkill` trait following the [`SKILL.md`](SKILL.md) protocol.

5. **Submit a pull request** with a clear description of your changes.

### Guidelines

- Follow Rust idioms and the [Rust API Guidelines](https://rust-lang.github.io/api-guidelines/)
- Write tests for new functionality
- Document public APIs with `///` doc comments
- Keep commits atomic and well-described

---

## License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for details.

---

*ClawCoach is not affiliated with or endorsed by Square Enix. Final Fantasy XIV is a registered trademark of Square Enix Holdings Co., Ltd.*
