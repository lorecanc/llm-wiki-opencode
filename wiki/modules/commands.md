---
title: "Wiki Command"
description: "Command definition for automated wiki generation and updates"
category: "modules"
source_files:
  - ".opencode/commands/wiki.md"
created: "2026-04-27"
last_updated: "2026-04-27"
---

# Wiki Command

## Overview

The `/wiki` command is the entry point for invoking automated wiki generation in the llm-wiki-opencode project. It is handled by the `wiki-orchestrator` agent and triggers either a creation or update flow depending on the current state of the wiki directory.

## Command Definition

**Agent**: `wiki-orchestrator`

**Purpose**: Generate or update the project wiki automatically

## Arguments

| Argument | Type | Description |
|----------|------|-------------|
| `depth` | integer (optional) | Depth level for wiki generation. If not provided, depth is auto-detected based on project size. |

## State Detection

When invoked, the command performs several state checks:

### Wiki Directory Status

Checks whether the `wiki/` directory exists. If it does not exist, the command runs the **creation flow**. If it exists, the command runs the **update flow**.

### Watermark Check

Reads `.last-updated-commit` to determine the last processed commit. If no watermark exists, a full scan is needed.

### Change Detection

If a watermark exists, the command runs `git diff --name-status` to identify files changed since the last wiki update. This enables incremental updates.

### Project Analysis

The command gathers:

- Project file tree (source files only, limited to first 100 entries)
- Total project file count
- Project configuration files (package.json, Cargo.toml, go.mod, pyproject.toml, etc.)

## Execution Flow

```
Start
  ├─> Wiki directory missing? ──> YES ──> CREATION FLOW
  └─> Wiki directory exists? ──> YES ──> UPDATE FLOW
```

### Creation Flow

Triggered when no wiki directory exists. Performs a full wiki generation from scratch.

### Update Flow

Triggered when wiki directory exists. Performs incremental updates based on changes since the last processed commit.

## Source

`.opencode/commands/wiki.md`