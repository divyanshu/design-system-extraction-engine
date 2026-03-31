# Design System Extractor

An agent-portable toolkit for replicating any design system on top of **shadcn/ui** at 80-90% fidelity — sufficient for high-quality prototyping and proof-of-concept work.

Give it a target design system (Storybook, Figma, npm tokens), and the agent-readable **skills** guide an AI agent (Claude, Gemini) through surveying, extracting, replicating, and validating components systematically.

## How It Works

1. **Survey** — Audit the target system and produce a structured manifest of every component, its variants, and its mapping against shadcn/ui
2. **Extract** — Pull component anatomy (props, slots, states) and design tokens from available sources
3. **Replicate** — Build themed components using shadcn patterns (`cva`, `cn`, Tailwind, CSS custom properties)
4. **Validate** — Human reviews each component for visual fidelity; agent iterates on feedback

The workflow is iterative — each phase feeds back into earlier phases as confidence improves.

## Quick Start

```bash
# 1. Clone this repo
git clone https://github.com/divyanshu/design-system-extraction-engine.git
cd design-system-extraction-engine

# 2. Create a new target
cp -r templates/ targets/{your-target}/

# 3. Fill in sources.md with your available sources (Storybook URL, npm packages, Figma, etc.)

# 4. Run the survey skill to produce a manifest
#    Point your AI agent at skills/01-survey-target-system.md

# 5. Extract anatomy, replicate components, validate with human review
#    Follow skills 03 → 05/06/07/08 based on each component's mapping type
```

See `targets/_example/` for a synthetic reference showing the expected file structure.

## Directory Structure

```
design-system-extraction-engine/
├── skills/                    # Agent-readable workflow guides (target-agnostic)
│   ├── 01-survey-target-system.md
│   ├── 03-extract-anatomy.md
│   ├── 05-replicate-theme.md      # direct_match components
│   ├── 06-replicate-extend.md     # partial_match components
│   ├── 07-replicate-build.md      # target_only components
│   └── 08-replicate-integrate.md  # bolt_on components
│
├── templates/                 # Starter files for new targets
│   ├── manifest.yaml
│   ├── sources.md
│   └── registry.json
│
├── targets/                   # Per-target replication work
│   └── _example/              # Synthetic reference (Acme Corp DS)
│
├── shared/                    # Cross-target utilities and scripts
│   ├── lib/
│   ├── scripts/
│   └── storybook-config/
│
└── .planning/                 # Workflow methodology and project status
    ├── design-system-replication-workflow.md
    └── project-status.md
```

## Skills

Skills are structured markdown instructions that any capable AI agent can follow. They are target-agnostic — target-specific context comes from the manifest, sources, and specs within each `targets/{name}/` directory.

| Skill | Purpose |
|-------|---------|
| `01-survey-target-system` | Audit target system, produce component manifest with mapping types and confidence scores |
| `03-extract-anatomy` | Extract props, variants, slots, and states from Storybook/npm/Figma |
| `05-replicate-theme` | Build `direct_match` components — token overrides + variant extension |
| `06-replicate-extend` | Build `partial_match` components — fork and modify structure/API |
| `07-replicate-build` | Build `target_only` components from scratch using primitives |
| `08-replicate-integrate` | Integrate and theme bolt-on libraries (charts, tables, drag-and-drop) |

## Key Conventions

- Components use **shadcn patterns**: `cva` for variants, `cn` utility, Tailwind CSS
- Theming via **CSS custom properties** mapped to shadcn's token layer
- Every extracted artifact carries a **confidence score** (High / Medium / Low)
- Agent-driven workflow: the agent does the work, the human verifies at checkpoints
- Output structured as a **shadcn registry** or plain `.tsx` files + `theme.css`

## Full Methodology

See [`.planning/design-system-replication-workflow.md`](.planning/design-system-replication-workflow.md) for the complete workflow specification — phases, skill definitions, manifest schema, component mapping taxonomy, repo structure, output strategy, and risk mitigations.

## License

MIT
