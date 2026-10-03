---
name: "pod-workspace-isolation"
description: "Create and manage dedicated Git worktree sandboxes ensuring clean state isolation."
---

# Pod Workspace Isolation Skill

Provides hermetic filesystem and credential sandboxing for agents.

## Core Capabilities
- Allocates independent Git worktrees per pod without full repo duplication.
- Mounts scoped environment variables and authentication keys.
- Reconciles and reaps dangling worktree directories on completion.
