# Cripsis Logic — OSS Surface

This is the open-source edge of the Cripsis Logic stack: a clean, portfolio-safe chain built around OS-level entropy, HTTP wiring, and a Python boundary layer for job governance.

---

## About Me

I’m the founder of Cripsis Logic, where I build entropy-driven infrastructure, secure communications logic, and boundary orchestration systems. My current open-source surface is a three-layer chain: a Rust OS-RNG entropy engine, a Go HTTP gateway, and a Python control plane that gates jobs via Keep/Throttle/Kill decisions.

My background combines hands-on systems engineering with operational experience, so I care about code that survives real adversaries, not just benchmarks. I’m comfortable living across Rust, Go, Python, and Linux, wiring telemetry, CI, and clean interfaces between components. If you care about reliability, security, and clear control surfaces, we’re aligned.


## Repos

### 🔹 krypton-entropy-core (Rust)

[github.com/hellocripsis/krypton-entropy-core](https://github.com/hellocripsis/krypton-entropy-core)

- Rust library that wraps the OS RNG (`OsRng`) and produces entropy metrics.
- Exposes `Keep / Throttle / Kill` decisions via a small signals engine.
- Binary: `entropy_health` – prints JSON with `{ samples, mean, variance, jitter, decision }`.
- MIT-licensed, no proprietary algorithms.

---

### 🔹 gold-dust-go (Go)

[github.com/hellocripsis/gold-dust-go](https://github.com/hellocripsis/gold-dust-go)

- Go HTTP gateway sitting in front of Krypton.
- Endpoints:
  - `GET /health` → snapshots Krypton health and returns a structured JSON status.
  - `POST /jobs` → accepts `{ job_id, payload }` and returns `accepted / throttled / denied` based on Krypton’s decision.
- Env-driven config:
  - `GOLD_DUST_ADDR`
  - `GOLD_DUST_KRYPTON_MODE` (`none`, `http`, `binary`)
  - `GOLD_DUST_KRYPTON_URL`
  - `GOLD_DUST_KRYPTON_BIN`
- Includes unit tests around config loading and decision mapping.

---

### 🔹 krypton-boundary-orchestrator (Python)

[github.com/hellocripsis/krypton-boundary-orchestrator](https://github.com/hellocripsis/krypton-boundary-orchestrator)

- Python control-plane / boundary layer that uses Krypton’s `Keep / Throttle / Kill` decisions to gate jobs.
- Two integration paths:
  - Directly call the `entropy_health` binary.
  - Talk to the Go `gold-dust-go` gateway over HTTP (`/health`, `/jobs`).
- CLI commands:
  - `health` – one-shot Krypton snapshot.
  - `run-once` – single-iteration run with a dummy job.
  - `run-job` – run a named job from the job registry (`dummy`, `gateway`, ...).
  - `run-loop` – multi-iteration telemetry loop with aggregated decision/action stats.
- Backed by pytest suite and GitHub Actions CI.

---

## System Map

High-level wiring of the open-source chain:

```text
[ OS RNG ] 
    │
    ▼
[ krypton-entropy-core (Rust) ]
    │  entropy_health JSON
    ▼
[ gold-dust-go (Go HTTP gateway) ]
    │  /health, /jobs
    ▼
[ krypton-boundary-orchestrator (Python) ]
    ├─ health snapshots
    ├─ job gating (Keep / Throttle / Kill)
    └─ telemetry loops
