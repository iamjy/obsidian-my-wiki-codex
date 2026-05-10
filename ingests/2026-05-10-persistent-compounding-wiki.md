---
title: "Persistent Compounding Wiki"
date: 2026-05-10
tags:
  - llm-wiki
  - second-brain
  - knowledge-management
type: ingest
status: stable
source: "../llm-wiki.md"
related:
  - "[[index]]"
---

# Persistent Compounding Wiki

## Source Excerpt

`llm-wiki.md` describes the core idea as a shift away from retrieving raw document chunks at query time and toward an LLM-maintained wiki that is incrementally updated. The source states that the wiki is "a persistent, compounding artifact" whose cross-references, contradictions, and synthesis are maintained over time.

## Processed Summary

The second brain should not behave like a temporary retrieval layer. It should become a durable Markdown wiki that accumulates knowledge with every ingest and query. When new source material arrives, the LLM reads it once, extracts the important claims, files a structured note, updates existing pages, strengthens or challenges prior synthesis, and records the work in the log.

## Key Claims

- Raw sources remain immutable and serve as source-of-truth material.
- The LLM owns the generated wiki layer: summaries, concept pages, links, indexes, and maintenance.
- The schema gives the LLM disciplined operating rules for note structure and workflows.
- The index is content-oriented and should be updated on every ingest.
- The log is chronological and should record ingests, queries, lint passes, and other maintenance.

## Links

- [[index]]
