---
name: "fleet-runner-scheduling"
description: "Schedule agent pods across distributed self-hosted runner fleets with capacity management."
---

# Fleet Runner Scheduling Skill

Optimizes placement of agent workloads across distributed machines.

## Core Capabilities
- Evaluates runner health, round-trip latency, and active pod load.
- Matches agent hardware requirements with runner capabilities.
- Prevents resource contention by enforcing `max_concurrent_pods`.
