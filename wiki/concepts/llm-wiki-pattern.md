---
title: "LLM Wiki Pattern"
description: "A pattern for building personal knowledge bases using LLMs through a persistent, compounding wiki that sits between users and raw sources"
category: "concepts"
source_files:
  - "docs/theory.md"
created: "2026-04-27"
last_updated: "2026-04-27"
---

# LLM Wiki Pattern

## Overview

The LLM Wiki pattern is implemented by the [LLM Wiki OpenCode project](../overview.md). It inverts the typical RAG approach to knowledge management. Instead of retrieving raw document chunks at query time and rediscovering knowledge from scratch on every question, an LLM incrementally builds and maintains a persistent wiki — a structured, interlinked collection of markdown files that accumulates understanding over time.

The key difference: **the wiki is a persistent, compounding artifact.** Cross-references are already established. Contradictions are flagged. Synthesis already reflects everything you've read. The knowledge is compiled once and kept current, not re-derived on every query.

## Three Layers

### Raw Sources

Your curated collection of source documents — articles, papers, images, data files. These are immutable; the LLM reads from them but never modifies them. Raw sources serve as the definitive source of truth.

### The Wiki

A directory of LLM-generated markdown files containing summaries, entity pages, concept pages, comparisons, overviews, and synthesis documents. The LLM owns this layer entirely: creating pages, updating them when new sources arrive, maintaining cross-references, and keeping everything consistent. You read it; the LLM writes it.

### The Schema

A configuration file (e.g., `CLAUDE.md` or `AGENTS.md`) that instructs the LLM on wiki structure, conventions, and workflows for ingesting sources, answering questions, and maintaining the wiki. This is the key configuration that makes the LLM a disciplined wiki maintainer rather than a generic chatbot. The schema co-evolves as you discover what works for your domain.

## Operations

### Ingest

When you add a new source to the raw collection, the LLM processes it by:

- Reading the source and discussing key takeaways with you
- Writing a summary page in the wiki
- Updating the index
- Updating relevant entity and concept pages across the wiki
- Appending an entry to the log

A single source might touch 10-15 wiki pages. You can ingest sources one at a time with active involvement, or batch-ingest multiple sources with less supervision.

### Query

When you ask questions against the wiki, the LLM:

- Searches for relevant pages using the index
- Reads relevant pages
- Synthesizes an answer with citations

Answers can take different forms: a markdown page, a comparison table, a slide deck (Marp), a chart (matplotlib), a canvas. Answers that represent valuable synthesis — comparisons, analyses, discovered connections — should be filed back into the wiki as new pages. This way explorations compound in the knowledge base just like ingested sources do.

### Lint

Periodically, ask the LLM to health-check the wiki. Look for:

- Contradictions between pages
- Stale claims superseded by newer sources
- Orphan pages with no inbound links
- Important concepts mentioned but lacking their own page
- Missing cross-references
- Data gaps that could be filled with a web search

The LLM suggests new questions to investigate and new sources to look for, keeping the wiki healthy as it grows.

## Indexing and Logging

### index.md

A content-oriented catalog of everything in the wiki. Each page is listed with a link, a one-line summary, and optional metadata (date, source count). Organized by category (entities, concepts, sources, etc.). The LLM updates it on every ingest. When answering a query, the LLM reads the index first to find relevant pages, then drills into them.

At moderate scale (~100 sources, hundreds of pages), this approach works well and avoids embedding-based RAG infrastructure entirely.

### log.md

A chronological, append-only record of what happened and when — ingests, queries, and lint passes. Each entry starts with a consistent prefix (e.g., `## [2026-04-02] ingest | Article Title`), making it parseable with simple unix tools:

```bash
grep "^## \[" log.md | tail -5
```

The log provides a timeline of the wiki's evolution and helps the LLM understand recent activity.

## Why It Works

The tedious part of maintaining a knowledge base is not reading or thinking — it's bookkeeping. Updating cross-references, keeping summaries current, noting when new data contradicts old claims, maintaining consistency across dozens of pages. Humans abandon wikis because the maintenance burden grows faster than the value. LLMs don't get bored, don't forget to update a cross-reference, and can touch 15 files in one pass. The wiki stays maintained because the cost of maintenance is near zero.

The pattern uses a [Two-Pass Writing](./two-pass-writing.md) approach to ensure consistent cross-referencing across the wiki.

The human's job is to curate sources, direct the analysis, ask good questions, and think about what it all means. The LLM's job is everything else.