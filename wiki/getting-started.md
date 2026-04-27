---
title: "Getting Started"
description: "How to invoke the wiki command and understand its operation modes"
category: "root"
source_files: []
created: "2026-04-27"
last_updated: "2026-04-27"
---

# Getting Started

## Invoking the Wiki Command

The wiki command is invoked through the OpenCode agent system using the `/wiki` command. See [Wiki Command](./modules/commands.md) for detailed documentation.

```
/wiki [depth-level]
```

The optional `depth-level` argument controls how thoroughly the wiki is generated or updated.

## Depth Levels

The wiki command supports three explicit depth levels plus auto-detection:

| Level | Description |
|-------|-------------|
| `light` | Minimal documentation coverage — key files and high-level overview only |
| `medium` | Standard coverage — main modules, components, and APIs documented |
| `deep` | Comprehensive coverage — all files, full details, and extensive cross-referencing |
| *(empty)* | Auto-detect based on project size |

### Auto-Detection

When no depth level is specified, the system automatically determines the appropriate depth based on project size (file count and complexity). Smaller projects default to `light` or `medium`, while larger projects may automatically select `medium` or `deep`.

## Operation Flows

The wiki command operates in two distinct flows depending on the current state of the wiki directory.

### Creation Flow

Triggered when no wiki exists (`NO_WIKI` state). The system:

1. Scans the entire project file tree
2. Builds a complete set of wiki pages from source files
3. Creates the `wiki/` directory structure
4. Sets the initial `.last-updated-commit` watermark

The creation flow performs a full scan of all source files and generates documentation for every significant component.

### Update Flow

Triggered when a wiki already exists (`WIKI_EXISTS` state). The system:

1. Reads the existing `.last-updated-commit` watermark
2. Compares it against the current `HEAD` commit
3. Computes the diff of changed files since the last update
4. Updates only the pages affected by those changes

The update flow is incremental — it only processes files that have changed, preserving all unaffected content exactly as-is.

## Wiki State Indicators

| Indicator | Meaning |
|-----------|---------|
| `NO_WIKI` | No `wiki/` directory exists — creation flow will run |
| `WIKI_EXISTS` | Wiki directory present — update flow will run |
| `NO_WATERMARK` | Wiki exists but has no commit reference — full rescan needed |
| `FULL_SCAN_NEEDED` | No watermark found — treated as new wiki |
