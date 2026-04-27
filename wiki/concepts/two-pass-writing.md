---
title: "Two-Pass Writing"
description: "The two-pass approach for wiki page generation: content-first, then cross-reference linking"
category: "concepts"
source_files:
  - ".opencode/agents/wiki-writer.md"
  - ".opencode/agents/wiki-orchestrator.md"
created: "2026-04-27"
last_updated: "2026-04-27"
---

# Two-Pass Writing

The wiki generation process uses a two-pass approach to ensure consistency, avoid circular references, and maintain link integrity across all pages. This approach is a key part of the overall [Architecture](../architecture.md).

## Why Two Passes Are Needed

## Why Two Passes Are Needed

Creating cross-references between wiki pages presents a fundamental ordering problem: a page cannot link to another page that does not yet exist. In a single-pass approach, writers must either omit links (leaving the wiki disconnected) or create placeholder links that may become incorrect once all content is known.

The two-pass approach solves this by separating content creation from link generation:

1. **Content completeness**: All pages exist with their full content before any cross-references are added
2. **Accurate linking**: Writers can read other pages to understand their actual content before linking
3. **No circular references**: The orchestrator can detect and prevent reference cycles
4. **Consistent link format**: All cross-references use the same pattern and relative paths

## Pass 1: Content Creation

In Pass 1, the wiki writer creates each page with full content but **without any cross-references to other wiki pages**.

### What the Writer Does

- Reads all source files specified for the assigned page
- Applies the appropriate template from the wiki-templates skill
- Writes complete, self-contained content
- Adds YAML frontmatter with proper metadata
- Omits all `[Page Title](./path.md)` style links

### Why Cross-References Are Excluded

During Pass 1, other pages may not yet exist or may still be in draft form. Adding links at this stage would result in:

- Broken links if the target page has not yet been written
- Incorrect links if the target page content changes during Pass 1
- Difficulty detecting circular reference chains

### Constraints

- No relative links to other wiki pages
- No links to pages that do not yet exist
- Content must be understandable without cross-references

## Pass 2: Cross-Reference Linking

Pass 2 begins only after all Pass 1 tasks complete. The wiki writer reads all created pages and adds cross-references between them.

### What the Writer Does

- Reads all wiki pages created in Pass 1
- Identifies natural link opportunities based on actual content
- Adds relative markdown links using the format: `[Page Title](./relative/path.md)`
- Marks glossary terms on first use with `*[term]*`
- Ensures no circular reference chains form

### Cross-Reference Linking Strategy

The writer follows these rules when adding links:

| Rule | Example |
|------|---------|
| Use relative paths | `[Architecture](./architecture.md)` |
| Link on first mention per section only | First mention gets the link, subsequent mentions do not |
| Avoid circular chains | Page A links to Page B, but Page B does not link back to Page A just because A linked to B |
| Mark glossary terms | `*[term]*` on first use in each page |

### Link Placement

Links are added only where the connection is semantically meaningful:

- **Related concepts**: When one page explains something that complements another
- **Prerequisites**: When understanding Page A requires reading Page B first
- **Usage references**: When code in Page A demonstrates use of a component from Page B
- **Architecture context**: When a component page links to the architecture overview

## Pass Execution Flow

```
Pass 1 (Parallel, up to 5 writers)
├── Writer 1 → Page A (content only)
├── Writer 2 → Page B (content only)
├── Writer 3 → Page C (content only)
└── ...

Pass 2 (Parallel, up to 5 writers)
├── Writer 1 → Read all pages, add cross-refs to A
├── Writer 2 → Read all pages, add cross-refs to B
└── ...
```

The orchestrator coordinates this flow (see [Agents](../modules/agents.md) for details on the orchestrator and writer roles):

1. Invokes wiki-analyzer to create a page plan
2. Spawns Pass 1 writer subtasks (max 5 parallel)
3. Waits for all Pass 1 tasks to complete
4. Spawns Pass 2 writer subtasks (max 5 parallel)
5. Invokes wiki-indexer to create index, log, and glossary

## Source Files

- [wiki-writer.md](../../.opencode/agents/wiki-writer.md) — Defines wiki-writer behavior for both passes
- [wiki-orchestrator.md](../../.opencode/agents/wiki-orchestrator.md) — Orchestrates the two-pass flow
