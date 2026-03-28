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
