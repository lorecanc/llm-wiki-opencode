---
title: "Agents"
description: "The five agents that power the wiki generation system"
category: "modules"
source_files:
  - ".opencode/agents/wiki-analyzer.md"
  - ".opencode/agents/wiki-indexer.md"
  - ".opencode/agents/wiki-orchestrator.md"
  - ".opencode/agents/wiki-updater.md"
  - ".opencode/agents/wiki-writer.md"
created: "2026-04-27"
last_updated: "2026-04-27"
---

# Agents

The wiki generation system is powered by five specialized agents. Each agent has a distinct role, with permissions scoped to their specific responsibilities.

## Overview

The agents follow the architecture patterns described in [Architecture](./architecture.md).

| Agent | Mode | Role |
|-------|------|------|
| [wiki-orchestrator](#wiki-orchestrator) | primary | Orchestrates generation and updates, delegates to subagents |
| [wiki-analyzer](#wiki-analyzer) | subagent | Analyzes codebase structure for wiki generation |
| [wiki-indexer](#wiki-indexer) | subagent | Maintains index.md, log.md, and glossary.md |
| [wiki-writer](#wiki-writer) | subagent | Writes and updates individual wiki pages |
| [wiki-updater](#wiki-updater) | subagent | Analyzes git diff to determine wiki impact |

## wiki-orchestrator

**Mode**: primary

**Role**: Orchestrates wiki generation and updates. Decides the flow and delegates to specialized subagents. This agent NEVER writes wiki pages directly — it only decides and delegates.

The orchestrator uses a two-pass writing approach documented in [Two-Pass Writing](../concepts/two-pass-writing.md).

### Permissions

- **edit**: allow
- **bash**: allow (all commands)
- **task**: allow
- **skill**: allow

### Key Features

- Auto-detects depth level based on file count (< 20 → deep, 20-200 → medium, > 200 → light)
- Two execution paths: CREATION (no wiki exists) and UPDATE (wiki exists)
- Coordinates Pass 1 (content without cross-references) and Pass 2 (cross-reference linking)
- Enforces maximum 5 parallel writer tasks
- Sets the git commit watermark after completion

### Responsibilities

1. Invoke wiki-analyzer for codebase analysis
2. Create wiki directory structure
3. Orchestrate wiki-writer tasks in two passes
4. Invoke wiki-indexer to create/update index, log, and glossary
5. Set `.last-updated-commit` watermark

## wiki-analyzer

**Mode**: subagent

**Role**: Analyzes a codebase structure for wiki generation. Read-only — never modifies files.

### Permissions

- **edit**: deny
- **bash**: limited to `find`, `wc`, `head`, `cat`, `ls`, `tree`
- **skill**: allow

### Key Features

- Identifies project language, framework, and architecture pattern
- Maps top-level directory structure to architectural concepts
- Plans wiki pages at light/medium/deep depth levels
- Collects glossary terms (class names, acronyms, domain-specific terminology)
- Produces structured analysis report with page plan

### Responsibilities

1. Read config files (package.json, Cargo.toml, go.mod, etc.) for project identity
2. Identify architecture pattern (monorepo, MVC, feature-based, layer-based)
3. Analyze modules, entry points, components, API surface, and config patterns
4. Return structured analysis report with page plan

## wiki-indexer

**Mode**: subagent

**Role**: Maintains wiki/index.md, wiki/log.md, and wiki/glossary.md. Sole owner of these files.

### Permissions

- **edit**: limited to wiki/index.md, wiki/log.md, wiki/glossary.md only
- **bash**: limited to `find wiki/`, `cat wiki/`, `ls wiki/`, `date`
- **skill**: allow

### Key Features

- Sole owner of index.md, log.md, and glossary.md (no other agent may modify these)
- Maintains complete catalog of all wiki pages organized by category
- Append-only log of wiki operations
- Alphabetically sorted glossary of project-specific terms
- Supports both initial generation and incremental update modes

### Responsibilities

1. Create and maintain index.md catalog
2. Append operation entries to log.md
3. Manage glossary.md with project terminology
4. Update catalog when pages are added, removed, or deprecated

## wiki-writer

**Mode**: subagent

**Role**: Writes and updates individual wiki pages based on source code analysis.

The wiki-writer relies on [Skills](../modules/skills.md) for conventions and templates.

### Permissions

- **edit**: limited to wiki/* except index.md, log.md, glossary.md
- **bash**: deny
- **skill**: allow

### Key Features

- Reads source files and produces structured markdown documentation
- Selects appropriate template based on page type
- Two-pass writing: Pass 1 (no cross-references), Pass 2 (with cross-references)
- Updates only affected sections when processing changes
- Preserves unaffected content exactly as-is

### Responsibilities

1. Create new wiki pages following templates and conventions
2. Update existing pages when source files change
3. Add cross-references between wiki pages (Pass 2 only)
4. Ensure all content is traceable to source files

### CRITICAL Restrictions

- NEVER modify index.md, log.md, or glossary.md (those belong to wiki-indexer)
- NEVER create files outside wiki/
- NEVER run bash commands

## wiki-updater

**Mode**: subagent

**Role**: Analyzes git diff to determine which wiki pages need updating. Maps source file changes to wiki page impact.

### Permissions

- **edit**: deny
- **bash**: limited to `git diff`, `git log`, `git show`, `cat`
- **skill**: allow

### Key Features

- Parses git diff output to classify file changes (Added, Modified, Deleted, Renamed)
- Maps changed source files to existing wiki pages
- Assesses impact scope (high/medium/low)
- Identifies new glossary terms from changed files
- Produces structured update plan

### Responsibilities

1. Analyze git diff to classify changes
2. Read wiki/index.md to understand current structure
3. Map source file changes to wiki page impact
4. Identify pages needing updates, new pages needed, and pages to deprecate
5. Note new glossary terms

### Impact Classification

- **High impact**: new files, deleted files, API changes, config changes, new dependencies, architectural changes
- **Medium impact**: significant refactors, new exports, changed interfaces
- **Low impact**: formatting changes, comment updates, test-only changes, minor bug fixes
