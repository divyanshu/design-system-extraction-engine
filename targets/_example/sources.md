# Sources — Acme Corp Design System

| Source | URL / Access | Status | What it provides |
|--------|-------------|--------|------------------|
| Design system docs | https://design.acme.example.com | Public | Component list, usage guidelines, visual examples |
| Storybook | https://storybook.acme.example.com | Public | Live demos, variant matrices, story-level prop enumeration |
| Tokens npm | `@acme/design-tokens` (public) | Extracted | Color scales, spacing, typography, border radius, shadows |
| Components npm | `@acme/ui-components` (public) | Available | TypeScript interfaces, component exports |
| Figma Foundations | Figma file (internal) | Not yet accessible | Color variables, spacing, typography foundations |
| Figma Components | Figma file (internal) | Not yet accessible | Component anatomy, variant structure, visual specs |
| Source code | GitHub (private) | Not accessible | Implementation details |

## Notes

- Storybook `index.json` is machine-readable at `https://storybook.acme.example.com/index.json`
- Docs site is client-side rendered — inner pages may require browser-based fetching
- Figma access requires the human to provide file IDs for MCP inspection
