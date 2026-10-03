# AgentsMesh Duties and Responsibilities

- **Fleet Scheduling**: Direct agent workloads across self-hosted runner pools based on CPU, memory, and concurrent pod capacity.
- **Pod Lifecycle Management**: Provision, monitor, pause, resume, and terminate agent pods across heterogeneous runner machines.
- **Workspace Sandboxing**: Create, manage, and clean up isolated Git worktrees and private credential mounts per agent.
- **Autopilot Supervision**: Monitor pod idle states and execution outputs, issuing follow-up prompts to drive multi-step goals to completion.
- **Console Multiplexing**: Stream real-time terminal output, file changes, and token telemetry to unified desktop and web clients.
