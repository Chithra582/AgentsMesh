# EXPLAINABILITY.md

This document explains the internal mechanisms, data lineage, operational boundaries, and governance framework of **AgentsMesh** (`agentsmesh`) in accordance with the **OpenGAP v0.1.0** specification for the **HiDevs GitAgent Passport** clearance pipeline.

> **Agent Name:** AgentsMesh (`agentsmesh`)  
> **Specification:** OpenGAP v0.1.0  
> **Category / Domain:** Agent Fleet Orchestration & Workforce Management  
> **Compliance Standard:** OpenGAP Checkpoint 2 (Explainability & Decision Governance), OWASP LLM Top 10, MITRE ATLAS  

---

## How the Agent Decides

AgentsMesh routes workloads, schedules pods across distributed runners, and supervises agent execution through a deterministic 5-stage decision pipeline.

### 1. Decision Architecture

The runtime intake, state classification, evaluation, and execution tracking operate across a deterministic, five-stage pipeline:

```
+-----------------------------------------------------------------------------------+
|                           5-STAGE DECISION PIPELINE                               |
+-----------------------------------------------------------------------------------+
|  [Stage 1: Workload Ingestion & Runner Fleet Heartbeat Evaluation]                |
|  - Parse task requirements, inspect runner capacity (max_concurrent_pods)         |
|                                     |                                             |
|                                     v                                             |
|  [Stage 2: Runner Affinity Scoring & Placement Calculation]                       |
|  - Compute affinity metric S_schedule across registered online runner nodes       |
|                                     |                                             |
|                                     v                                             |
|  [Stage 3: Sandbox Allocation & Git Worktree Isolation]                           |
|  - Provision isolated sandbox directory, checkout branch, bind private credentials|
|                                     |                                             |
|                                     v                                             |
|  [Stage 4: Threshold Evaluation & Pod Dispatch]                                   |
|  - Verify tau >= 0.70; verify quota limits, initialize pod execution daemon      |
|                                     |                                             |
|                                     v                                             |
|  [Stage 5: Autopilot Monitoring, Terminal Multiplexing & Reaping]                 |
|  - Monitor idle triggers, stream terminal output, harvest artifacts on completion |
+-----------------------------------------------------------------------------------+
```

### 2. Decision Logic & Routing Formulations

For a requested agent pod $p_k$ requiring resources $\langle c_{\text{cpu}}, c_{\text{mem}}, c_{\text{tool}} \rangle$ and a candidate runner $r_j$ with advertised capacity $C(r_j)$, the scheduling score $S_{\text{schedule}}(r_j, p_k)$ is formulated as:

$$S_{\text{schedule}}(r_j, p_k) = w_{\text{cap}} \left(1 - \frac{U(r_j)}{C(r_j)}\right) + w_{\text{lat}} L(r_j) + w_{\text{aff}} A(r_j, p_k) + w_{\text{health}} H(r_j)$$

Where:
- $U(r_j)$ is the number of active running pods on runner $r_j$, and $C(r_j) = \text{max\_concurrent\_pods}(r_j)$.
- $L(r_j) = \max\left(0, 1 - \frac{t_{\text{rtt}}(r_j)}{500\,\text{ms}}\right)$ represents network round-trip latency to the console.
- $A(r_j, p_k) \in [0, 1]$ represents repository cache affinity (presence of pre-cached Git object repository).
- $H(r_j) \in [0, 1]$ measures node health, uptime stability, and recent task success rate.
- Standard default weights: $w_{\text{cap}} = 0.40$, $w_{\text{lat}} = 0.20$, $w_{\text{aff}} = 0.25$, $w_{\text{health}} = 0.15$ with $\sum w = 1.0$.

Pod placement requires:

$$S_{\text{schedule}}(r_j, p_k) \ge \tau \quad (\tau = 0.70) \quad \land \quad U(r_j) < C(r_j)$$

### 3. Thresholding & Refusal Decision Criteria

AgentsMesh enforces strict operational boundaries and deterministic refusal thresholds:
- **Refusal on ERR_RUNNER_CAPACITY_EXCEEDED**: All available runners have $U(r_j) \ge C(r_j)$ halts execution with code `ERR_RUNNER_CAPACITY_EXCEEDED`.
- **Refusal on ERR_WORKSPACE_ISOLATION_FAILED**: Git worktree creation conflict or disk full halts execution with code `ERR_WORKSPACE_ISOLATION_FAILED`.
- **Refusal on ERR_AUTOPILOT_ITERATION_CAP**: Pod autopilot turn count exceeds $I_{\max}$ (default 25) halts execution with code `ERR_AUTOPILOT_ITERATION_CAP`.
- **Refusal on ERR_CREDENTIAL_UNAUTHORIZED**: Repository token lacks commit or push scope halts execution with code `ERR_CREDENTIAL_UNAUTHORIZED`.
- **Refusal on ERR_RUNNER_HEARTBEAT_LOST**: Runner heartbeat absent for $> 30\,\text{seconds}$ halts execution with code `ERR_RUNNER_HEARTBEAT_LOST`.

### 4. Fallback Decision Mechanism

Continuous operational stability is maintained through layered fault recovery:
- **Tier 1 (Alternative Runner Reallocation):** If the selected runner experiences transient overload or network disconnect, immediately rescore candidate runners and redispatch the pod.
- **Tier 2 (Pod State Snapshot & Graceful Restart):** If a pod crashes or stalls, snapshot the Git worktree diff and respawn a fresh container instance from the last clean commit.
- **Model Fallback Cascade**: High-level reasoning and synthesis default to `gemini-2.0-flash` with automatic failover to `gpt-4o` and `claude-3-5-sonnet`.

### 5. Human-in-the-Loop Governance

Human operators retain sovereign authority over the multi-agent execution lifecycle:
- **Tier 3 (HumanintheLoop Operator Takeover):** When autopilot hits iteration limits or encounters circular merge conflicts, suspend autonomous actions and transfer interactive terminal control to the human console.
- **Session Telemetry Auditing**: Operators inspect execution logs, routing traces, and token usage to maintain oversight.

---

## The Data It Uses

AgentsMesh operates under strict principles of data minimization, environment isolation, and privacy protection.

### 1. Ingested Input Data

The framework processes only operational data necessary to perform its functions:
- **Workload Instructions**: High-level engineering tasks, user prompts, and autopilot instruction templates.
- **Repository Metadata**: Git repository URLs, branch names, commit hashes, and file diffs.
- **Runner Telemetry**: CPU load, memory utilization, disk availability, active pod count, and network ping.

### 2. Configuration & Reference Data

- **Runner Inventory**: Registration table of authorized self-hosted runners, advertised capacities, and tag affinities.
- **Credential Vault**: Encrypted repository deployment keys and fine-grained agent service tokens.
- **Pod State Store**: Immutable history of pod commands, output logs, autopilot turns, and terminal history.

### 3. Base Model & Inference Lineage

- **Model Agnostic**: Compatible with any CLI or API-driven coding agent (Claude Code, Aider, Codex, OpenHands, SWE-agent).
- **Weight Integrity**: Operates directly on agent binaries and API models configured by the user with zero internal model tampering.

### 4. Data Privacy, Storage, and Retention

- **OWASP LLM & MITRE ATLAS Hardened**: Defended against indirect prompt injection, credential leakage, and unauthorized external API dispatch.
- **Local Environment Isolation**: Agent execution workspaces, intermediate scratchpads, and vector stores reside strictly within designated local project directories.
- **Automated Secret Scrubbing**: API keys, database credentials, and personal credentials are automatically redacted prior to embedding or logging.
- **Zero Commercial Monetization**: Prompts, intermediate reasoning trajectories, and task deliverables are never commercialized or shared with third parties.

---

## Limitations

Understanding the operational boundaries and technical constraints of AgentsMesh is essential for effective deployment.

### 1. High-density agent runs on a single
- **Limitation**: High-density agent runs on a single runner machine can lead to disk space exhaustion from parallel Git worktrees.
- **Mitigation**: The workspace manager uses Git shared object stores (`--reference`) and enforces automated sandbox disk quotas.

### 2. Autopilot control agents can generate circular
- **Limitation**: Autopilot control agents can generate circular instructions if the underlying agent repeatedly reports partial completion.
- **Mitigation**: Semantic similarity deduplication on consecutive autopilot prompts halts execution when instruction drift is negligible.

### 3. Distributed runners behind restrictive corporate firewalls
- **Limitation**: Distributed runners behind restrictive corporate firewalls can experience WebSocket streaming disconnections.
- **Mitigation**: Built-in TCP reconnection heartbeats and local terminal buffer replays prevent dropped keystrokes or output loss.

### 4. Concurrent Git pushes from multiple agent
- **Limitation**: Concurrent Git pushes from multiple agent pods to the same branch cause remote rejection conflicts.
- **Mitigation**: AgentsMesh assigns each pod a dedicated ephemeral topic branch (`mesh/{pod-id}`) with automated PR generation.

### 5. Self-hosted runner process crashes can leave
- **Limitation**: Self-hosted runner process crashes can leave orphaned worktree directories on the host operating system.
- **Mitigation**: On runner startup, the lifecycle reconciliation manager scans and cleans up all unassociated sandbox worktrees.

---

## Summary & Compliance Checklist

| Checkpoint 2 Requirement | Corresponding Section | Status |
| :--- | :--- | :---: |
| **How the agent decides** | [How the Agent Decides](#how-the-agent-decides) | **Covered** |
| - Decision architecture & 5-stage pipeline | Section 1 | Verified |
| - Decision logic & routing formulations | Section 2 | Verified |
| - Thresholding & refusal decision criteria | Section 3 | Verified |
| - Fallback decision mechanism | Section 4 | Verified |
| - Human-in-the-loop governance & oversight | Section 5 | Verified |
| **The data it uses** | [The Data It Uses](#the-data-it-uses) | **Covered** |
| - Ingested input data & query streams | Section 1 | Verified |
| - Configuration & reference schemas | Section 2 | Verified |
| - Base model lineage & deterministic engines | Section 3 | Verified |
| - Data privacy, retention lifecycle & MITRE/OWASP | Section 4 | Verified |
| **Its limitations** | [Limitations](#limitations) | **Covered** |
| - High-density agent runs on a single | Section 1 | Verified |
| - Autopilot control agents can generate circular | Section 2 | Verified |
| - Distributed runners behind restrictive corporate firewalls | Section 3 | Verified |
| - Concurrent Git pushes from multiple agent | Section 4 | Verified |
| - Self-hosted runner process crashes can leave | Section 5 | Verified |
