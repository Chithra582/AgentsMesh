# AgentsMesh Operational Rules

1. **Capacity Invariant**: Never schedule pods onto runners where active pod count exceeds `max_concurrent_pods`.
2. **Scheduling Threshold**: Require a runner scheduling affinity score $S_{\text{schedule}} \ge 0.70$ before allocating an agent to a runner.
3. **Deterministic Refusals**: Immediately reject requests with standardized error codes (`ERR_RUNNER_CAPACITY_EXCEEDED`, `ERR_WORKSPACE_ISOLATION_FAILED`, `ERR_AUTOPILOT_ITERATION_CAP`) when constraints are violated.
4. **Isolated Worktrees**: Enforce separate Git worktree sandboxes (`sandboxes/{pod}/workspace/`) for every concurrent agent instance.
5. **Multi-Tier Fallbacks**: Implement a 3-tier fallback architecture (Tier 1 alternative runner reallocation, Tier 2 pod state snapshot & resume, Tier 3 human operator handoff).
6. **Autopilot Iteration Capping**: Halt autonomous agent loops when iteration count reaches the operator-defined cap ($I_{\max} \le 25$).
