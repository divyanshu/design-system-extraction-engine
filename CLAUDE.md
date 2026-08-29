# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Design system extractor engine: takes **shadcn/ui** as a baseline and transforms it to reproduce any target design system at 80-90% fidelity for prototyping. Agent-portable (Claude and Gemini). Skills are target-agnostic; target-specific work lives in `targets/<name>/`.

## Key Conventions

- Components use **shadcn patterns**: `cva` for variants, `cn` utility, Tailwind CSS
- Theming via **CSS custom properties** mapped to shadcn's token layer
- Output structured as a **shadcn registry** (`shadcn build` / `npx shadcn add`)
- Bolt-on libraries themed through CSS custom properties, not library-specific APIs
- Every extracted artifact carries a **confidence score** (High / Medium / Low)

## Reference Documents

- `.planning/design-system-replication-workflow.md` — Full workflow spec: phases, skill definitions, manifest schema, component mapping taxonomy, repo structure. **Read this first when starting work.**
- `.planning/project-status.md` — Quick-glance current status, decisions made/deferred.

## Standing Instructions

- **Keep planning files in sync** after meaningful changes: `.planning/project-status.md` and `.planning/design-system-replication-workflow.md`. Only update CLAUDE.md if conventions or repo structure fundamentally change.
- **Agent-driven workflow:** The agent does the work; the human verifies at checkpoints. Minimize manual work for the human.

<!-- knowledge-graph-start (managed: ~/Projects/workshop/knowledge-graph) -->
## Knowledge graph — reuse, appraise, enrich

A curated **reuse knowledge graph** lives at `~/Projects/workshop/knowledge-graph` (Obsidian vault). Treat it
as a *starting point to enrich*, not a closed answer — a thin or wrong-shaped hit still needs fresh
research.

**Before** researching a domain, evaluating a stack, designing tokens/UI, or running a
competitive/dataset analysis: run the **`graph-forager`** skill (full pass in
`~/Projects/workshop/knowledge-graph/REUSE.md`). Its non-negotiable step is the **appraisal gate** — after
consulting the graph, state an explicit verdict on whether it answers *the actual question asked*:

- **sufficient** → reuse the artifact named in `Reuse this →` and stop.
- **partial / miss** → research the *named delta* broadly, cross-check against the graph, then **fold
  it back** (extend the node + its `covers:`/`thin_on:` + `Reuse this →`, or drop a
  `~/Projects/workshop/knowledge-graph/inbox/` note per `CONTRIBUTING.md`).

Judge **coverage**, **shape** (is it the right *kind* of answer?), and **depth/recency**; default to
**partial** when unsure. A hit is a starting point, not a conclusion. Apply this to subagents too.
<!-- knowledge-graph-end -->
