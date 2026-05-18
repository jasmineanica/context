---
layout: page
title: Adding a project
permalink: /docs/adding-a-project/
---

The hub at [/projects/]({{ "/projects/" | relative_url }}) is driven by `_data/projects.yml`. Each entry is one project. Adding a new one is a six-step loop.

## 1. Pick a CS concept

Something I've actually studied recently — a data structure, an algorithm, a system-design primitive. Trie, sliding window, consistent hashing, topological sort, CRDT, etc.

## 2. Find a real-world problem that needs it

Not a toy demo. A problem where the concept is the *right* answer, not a stretch. Some seed pairings:

| Concept | Real problem |
|---|---|
| Sliding window / token bucket | Rate limiter |
| Trie | Autocomplete, URL routing |
| Min-heap | Job queue with priorities |
| Hashing + Base62 | URL shortener |
| Bloom filter | Negative-cache layer |
| Topological sort | Build system, task scheduler |

## 3. Scaffold from the template

Copy the layout from [`sm2-flashcards`](https://github.com/jasmineanica/sm2-flashcards): `Dockerfile`, `.github/workflows/ci.yml`, `Makefile`, README template, `pyproject.toml`.

## 4. Build the core from scratch

The data structure or algorithm at the heart of the project gets its own module, full typing, and a test file. Plus a benchmark against the naive approach — that benchmark is what the README highlights.

## 5. Deploy

Fly.io for anything web-shaped. Skip this step for pure-library projects.

## 6. Add a YAML entry

```yaml
- name: <Project Name>
  slug: <repo-slug>
  tagline: <one sentence>
  concepts: [<Concept>, <Concept>]
  repo: https://github.com/jasmineanica/<repo-slug>
  demo: https://<repo-slug>.fly.dev
  status: live   # or wip
```

Commit, push, done. The project shows up on [/projects/]({{ "/projects/" | relative_url }}).
