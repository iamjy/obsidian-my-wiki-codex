# Repository Guidelines

## Purpose

This repository contains an LLM-maintained second brain based on `llm-wiki.md`. The wiki is a persistent, compounding artifact: the LLM reads sources, writes structured Markdown notes, maintains links, updates indexes, and appends chronological log entries.

## Note Types

- `concept`: synthesized topic, pattern, entity, or durable idea.
- `ingest`: processed external source or source excerpt.
- `log`: append-only operational history.
- `reference`: stable supporting material, tool note, or cited resource.
- `project`: active work area, plan, or initiative.

## Required Frontmatter

Every note except `wiki/logs/log.md` must start with YAML frontmatter:

```yaml
---
title: "Human-readable title"
date: YYYY-MM-DD
tags: []
type: concept | ingest | log | reference | project
status: draft | active | stable | archived
source: ""
related: []
---
```

Use `source` for the raw source path, URL, or source title. Use `related` for `[[wikilink]]` targets.

## Link Style

Use Obsidian wikilinks: `[[note-slug]]` or `[[note-slug|Display Text]]`. Prefer links to existing notes. When a key concept has no note yet, create or propose one rather than leaving important ideas unlinked.

## Naming Rules

Use kebab-case filenames. Ingest notes must be date-prefixed: `YYYY-MM-DD-source-slug.md`. Examples: `persistent-compounding-wiki.md`, `2026-05-10-persistent-wiki-pattern.md`.

## Folder Routing

- `wiki/ingests/`: `ingest` notes.
- `wiki/concepts/`: `concept` notes.
- `wiki/references/`: `reference` notes.
- `wiki/projects/`: `project` notes.
- `wiki/logs/log.md`: chronological `log`.
- `wiki/index.md`: master catalog, updated whenever a note is added.
- `AGENTS.md`: schema and operating rules for the vault.

## Ingest Pipeline

1. Read the raw source without modifying it.
2. Extract key claims, concepts, contradictions, and useful references.
3. Create one `ingest` note with complete frontmatter.
4. Add wikilinks to related notes or concepts.
5. Update relevant concept, reference, or project notes when needed.
6. Update `wiki/index.md` with the new note and a one-line summary.
7. Append a timestamped entry to `wiki/logs/log.md`.

## Persistent Operating Rules

- All notes must conform to the schema in `AGENTS.md`.
- After any vault write, append a timestamped entry to `wiki/logs/log.md`.
- After any new note, update `wiki/index.md`.
- If `llm-wiki.md` is ambiguous on a decision, ask before acting.
- Raw sources are source-of-truth material; do not modify them during ingest.
