---
title: "Architecture"
description: "System architecture and design decisions for the LLM Wiki pattern"
category: "root"
source_files:
  - ".opencode/agents/wiki-orchestrator.md"
  - ".opencode/agents/wiki-analyzer.md"
  - ".opencode/agents/wiki-writer.md"
  - ".opencode/agents/wiki-updater.md"
  - ".opencode/agents/wiki-indexer.md"
created: "2026-04-27"
last_updated: "2026-04-27"
---

# Architecture

## Overview

The llm-wiki-opencode project implements the LLM Wiki pattern — an automated system that generates and maintains project documentation by leveraging LLM agents with specialized roles. The architecture uses an Orchestrator + Subagent pattern with five coordinated agents, each with distinct permissions and responsibilities.

## Agent Roles

The five agents are documented in detail in [Agents](./modules/agents.md).

| Agent | Mode | Role |
|-------|------|------|
| `wiki-orchestrator` | Primary | Decides flow, coordinates subagents, delegates work |
| `wiki-analyzer` | Subagent | Analyzes codebase structure, produces structured report |
| `wiki-writer` | Subagent | Writes individual wiki pages from source analysis |
| `wiki-updater` | Subagent | Analyzes git diffs to determine wiki impact |
| `wiki-indexer` | Subagent | Maintains index.md, log.md, glossary.md |

## Orchestration Flow

### Path A — Wiki Creation (No Existing Wiki)

When no wiki exists, the orchestrator executes a six-step creation pipeline:

**Step 1: Analysis**
The orchestrator invokes `wiki-analyzer` to examine the codebase. The analyzer reads config files (package.json, Cargo.toml, go.mod, etc.) to identify language, framework, and project identity. It traverses the directory structure to identify architecture patterns (monorepo, MVC, feature-based, layer-based), then catalogs modules, entry points, components, API surfaces, and configuration patterns. Glossary terms are collected from class names, acronyms, and domain-specific terminology.

**Step 2: Directory Setup**
The orchestrator creates the wiki directory structure:
```
mkdir -p wiki/modules wiki/components wiki/api wiki/concepts wiki/config wiki/dependencies
```
Only subdirectories with identified content are created.

**Step 3: Pass 1 — Content Writing**
The orchestrator spawns up to 5 parallel `wiki-writer` tasks. Each writer receives a page path, analysis context, and source files to document. Writers create content following the wiki-templates without cross-references in this pass. This isolation ensures pages are independently verifiable before linking.

**Step 4: Pass 2 — Cross-Reference Injection**
After all Pass 1 writers complete, the orchestrator invokes `wiki-writer` again to read all created pages and add cross-references between them. Links use relative paths (`[Page Title](./relative/path.md)`) and are added on first mention per section only.

**Step 5: Index, Log, and Glossary**
The orchestrator invokes `wiki-indexer` with the complete page list and glossary terms. The indexer creates three files:
- `wiki/index.md` — Complete catalog organized by category
- `wiki/log.md` — Append-only operation log with entry type "initial-generation"
- `wiki/glossary.md` — Alphabetically sorted term definitions

**Step 6: Watermark**
The orchestrator writes the current git SHA to `wiki/.last-updated-commit`:
```
git rev-parse HEAD > wiki/.last-updated-commit
```

### Path B — Wiki Update (Existing Wiki)

When a wiki already exists, the orchestrator executes an incremental update pipeline:

**Step 1: Change Analysis**
The orchestrator invokes `wiki-updater` with the git diff output. The updater parses changed files (Added, Modified, Deleted, Renamed), reads the current `wiki/index.md` to map files to pages, and classifies impact:
- **High impact**: New files, deleted files, API changes, config changes, new dependencies, architectural changes
- **Medium impact**: Significant refactors, new exports, changed interfaces
- **Low impact**: Formatting changes, comment updates, test-only changes

**Step 2: Page Updates**
For each impacted page, the orchestrator spawns `wiki-writer` tasks (max 5 parallel). Writers read the current wiki page, re-read affected source files, update only changed sections, preserve unaffected content, and refresh cross-references.

**Step 3: Index, Log, Glossary Update**
The orchestrator invokes `wiki-indexer` with the list of changed pages. The indexer updates `index.md`, appends a new `log.md` entry (type: "incremental_update"), and refreshes `glossary.md`.

**Step 4: Watermark Update**
The orchestrator updates the SHA watermark.

## Two-Pass Wiki Writing

The two-pass approach is documented in detail in [Two-Pass Writing](./concepts/two-pass-writing.md). It separates content creation from linking, providing several benefits:

**Pass 1 — Independent Content**
Writers create each page without cross-references. This ensures:
- Each page can be verified independently
- No circular dependency chains form during creation
- Writers can execute in parallel without coordination overhead
- Content is stable before inter-page links are established

**Pass 2 — Cross-Reference Injection**
After all content exists, a second writer pass reads every page and adds links. This pass:
- Has full knowledge of all page titles and paths
- Can avoid circular reference chains
- Ensures links are added on first mention per section only
- Maintains consistent linking style across the wiki

## Permission-Based Isolation

Each agent operates under strict permission boundaries enforced at the agent definition level:

| Agent | Edit | Bash | Task |
|-------|------|------|------|
| `wiki-orchestrator` | Allow all | Allow all | Allow all |
| `wiki-analyzer` | Deny all | Limited (find, wc, head, cat, ls, tree) | Allow |
| `wiki-writer` | Deny all | Deny all | Allow |
| `wiki-updater` | Deny all | Limited (git diff, git log, git show, cat) | Allow |
| `wiki-indexer` | Deny all | Limited (find wiki/, cat wiki/, ls wiki/, date) | Allow |

### wiki-writer Isolation

The `wiki-writer` agent is explicitly denied write access to three files:
- `wiki/index.md` — Owned exclusively by `wiki-indexer`
- `wiki/log.md` — Owned exclusively by `wiki-indexer`
- `wiki/glossary.md` — Owned exclusively by `wiki-indexer`

This isolation prevents accidental corruption of the wiki catalog and ensures `wiki-indexer` remains the authoritative source for page inventory.

### wiki-analyzer and wiki-updater Read-Only Posture

Both `wiki-analyzer` and `wiki-updater` have `edit: deny` on all files. They produce analysis reports but never modify wiki content. This ensures change analysis is always a planning step, never an execution step.

## Depth Levels

The orchestrator auto-detects project complexity to determine documentation depth:

| File Count | Depth | Pages Generated |
|------------|-------|-----------------|
| < 20 | deep | ~50+ pages (detailed per-file docs, config reference, dependency map) |
| 20–200 | medium | ~20-30 pages (module, component, concept, API pages) |
| > 200 | light | ~10 pages (overview, architecture, getting-started, one per module) |

Users can override this auto-detection by passing an explicit depth argument.

## Key Design Decisions

### Orchestrator as Primary Coordinator

The orchestrator runs in `primary` mode with full edit, bash, and task permissions. It never writes wiki pages directly — instead it decides and delegates. This keeps the orchestrator focused on flow control while specialized agents handle execution.

### Subagent Task Spawning

Wiki-writer tasks spawn with a maximum parallelism of 5. This cap:
- Prevents overwhelming the LLM context window
- Avoids file system contention on wiki directory writes
- Allows efficient pipeline completion while maintaining quality

### SHA Watermark

The `.last-updated-commit` file stores the git SHA, enabling the system to correlate wiki state with specific codebase commits. This supports incremental updates by providing a reference point for diff generation.

### Permission Deny-by-Default

All agents use deny-by-default permission models. Even the orchestrator (which has broad allow permissions) and wiki-writer (which edits wiki content) deny by default and only allow specific paths. This limits blast radius if an agent behaves unexpectedly.
