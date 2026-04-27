---
title: "Skills"
description: "Specialized skill definitions for wiki operations"
category: "modules"
source_files:
  - ".opencode/skills/wiki-conventions/SKILL.md"
  - ".opencode/skills/wiki-templates/SKILL.md"
created: "2026-04-27"
last_updated: "2026-04-27"
---

# Skills

## Purpose

The project defines two specialized skills that provide domain-specific instructions for wiki operations. These skills are loaded by the wiki-writer to ensure consistent documentation structure, naming, and formatting across the project.

## Key Files

| File | Role |
|------|------|
| `.opencode/skills/wiki-conventions/SKILL.md` | Defines wiki structure, naming rules, frontmatter format, and ownership boundaries |
| `.opencode/skills/wiki-templates/SKILL.md` | Provides reusable page templates for different documentation types |

## Skills Reference

### wiki-conventions

**Purpose**: Conventions and rules for the project wiki structure, naming, and formatting.

**Coverage**:
- **Wiki Structure**: Defines the directory layout under `wiki/`, including reserved files (`index.md`, `log.md`, `glossary.md`) and content directories (`modules/`, `components/`, `api/`, `concepts/`, `config/`, `dependencies/`)
- **Naming Rules**: Enforces kebab-case, lowercase filenames with `.md` extension
- **Page Frontmatter**: Specifies required YAML frontmatter fields (`title`, `description`, `category`, `source_files`, `created`, `last_updated`)
- **Language**: All content in English, technical terminology consistency, glossary term marking on first use
- **Cross-References**: Relative markdown links, first-mention-only linking, no circular chains
- **Ownership Rules**: `index.md`, `log.md`, `glossary.md` are owned exclusively by wiki-indexer; all other pages are owned by wiki-writer

### wiki-templates

**Purpose**: Templates for different types of wiki pages — use these as starting structures.

**Available Templates**:
- **Overview Page** (`wiki/overview.md`): Project-level summary with key features, tech stack, and structure
- **Architecture Page** (`wiki/architecture.md`): System design with component diagrams and data flow
- **Module Page** (`wiki/modules/*.md`): Module documentation with purpose, key files, public API, and usage examples
- **Component Page** (`wiki/components/*.md`): Component documentation with props/interface and usage
- **API Page** (`wiki/api/*.md`): Endpoint documentation with request/response shapes
- **Getting Started Page** (`wiki/getting-started.md`): Setup instructions and project conventions

## Skill Loading

Skills are loaded by the wiki-writer when a task matches their description. The wiki-conventions skill must be loaded before writing any page, and wiki-templates must be loaded before applying templates.

```python
skill(name="wiki-conventions")  # Load conventions first
skill(name="wiki-templates")    # Then load templates
```
