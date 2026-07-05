# Automated Presentation Builder

A Claude Code skill that generates branded `.pptx` presentations from scratch. Give it a topic, point it at a brand source, and it builds a full consulting-style deck — programmatically.

## What it does

1. **Extracts brand** from a Figma file, website URL, existing `.pptx` template, or PDF
2. **Researches** the topic independently (+ any sources you provide)
3. **Proposes slide structure** using a composition library (stat grids, card grids, bar charts, doughnut charts, timelines, etc.)
4. **Locks content** before building (action titles, stats, card text — all approved)
5. **Builds the deck** with `python-pptx` — no templates, fully generated from code
6. **Runs visual QA** and fixes overlaps, contrast issues, text overflow

## How to use

Install as a Claude Code skill (copy `presentation-builder.md` to your `.claude/skills/` directory), then ask Claude to build a presentation. It will walk you through:

1. Topic
2. Audience and tone
3. Design source (where to pull the brand from)
4. Content sources
5. Photos/images (optional)

## Sample output

See [`samples/`](./samples/) for a generated deck:

- **Shruti_Revolut_WhyHireMe_Portfolio.pptx** — A "Why hire me" application deck using Revolut's brand (extracted from their 2025 Annual Report PDF). Features: dark mode, bar chart (JD vs evidence scoring), doughnut chart (experience breakdown), connected timeline, 2x2 card grids.

## Tech

- Python + `python-pptx` for generation
- Claude AI for research, content writing, and orchestration
- Pillow for asset generation (arcs, icons)
- Brand tokens stored as JSON for reuse

## Composition library

| Pattern | Use case |
|---------|----------|
| `cover` | Opening slide (dark bg, logo, title) |
| `hero_stats` | Key metrics with stat boxes |
| `card_grid_2x2` / `3x2` | Features, services, requirements |
| `bar_chart` | Comparisons, scoring |
| `doughnut_chart` | Breakdowns, concentration |
| `timeline` | Career, milestones, history |
| `two_column_story` | Narrative + structured data |
| `investment_thesis` | Why invest / why hire |
| `full_table` | Financial data, comparisons |
| `disclaimer` | Legal close |

## Requirements

```
python-pptx
Pillow
```
