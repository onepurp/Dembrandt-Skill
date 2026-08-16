# Dembrandt Agent Skill

> **Extract, analyze, and operationalize website design systems with an AI agent.**

Dembrandt Skill equips agents with a structured workflow for inspecting live websites and turning observed design-system evidence into useful implementation artifacts. It covers colors, typography, spacing, borders, radii, shadows, motion, components, responsive behavior, brand identity, accessibility, design tokens, and visual drift.

The skill is designed to work with the [Dembrandt CLI and MCP server](https://github.com/dembrandt/dembrandt). It teaches an agent when to use live extraction, how to select the appropriate browser and viewport context, how to interpret confidence and limitations, and how to export results for design and development workflows.


## What this skill supports

The skill helps an agent:

- Extract a complete design system from a live website.

- Inspect focused categories such as color palettes, typography, spacing, surfaces, components, or brand identity.

- Compare desktop and mobile rendering contexts.

- Analyze light and dark themes.

- Run WCAG 2.1 contrast analysis using real DOM text and background pairs.

- Produce W3C Design Tokens Community Group JSON, `DESIGN.md`, Tailwind v4 `@theme` CSS, HTML reports, and brand-guide PDFs.

- Detect design-system findings such as contrast failures, near-duplicate colors, off-scale spacing, radius sprawl, and duplication.

- Compare extraction baselines and candidates to identify design drift in CI.

- Use Dembrandt through MCP with asynchronous job polling and focused analysis tools.

## Installation

Install this skill through your agent by adding the generated skill package or attaching the included `SKILL.md` file.

The skill itself provides guidance for using Dembrandt. To run Dembrandt locally through the CLI, install the package and its matching browser binary:

```bash
npm install -g dembrandt
dembrandt install-browser
dembrandt https://example.com --json-only
```

For an ephemeral installation:

```bash
npx dembrandt install-browser
npx dembrandt https://example.com --json-only
```

Dembrandt requires Node.js 18 or later. Its browser-dependent workflows require a compatible Playwright browser installation.

## MCP configuration

When an MCP-compatible client is available, configure the Dembrandt server with:

```json
{
  "mcpServers": {
    "dembrandt": {
      "command": "npx",
      "args": ["-y", "--package", "dembrandt", "dembrandt-mcp"]
    }
  }
}
```

The skill explains how to use extraction tools such as `get_design_tokens`, `get_color_palette`, `get_typography`, `get_component_styles`, `get_surfaces`, `get_spacing`, and `get_brand_identity`. It also covers pure analysis and export tools including `compute_drift`, `get_findings`, `export_dtcg`, `generate_design_md`, and `render_report`.

## Typical workflow

1. Define the design question and confirm the target URL and authorization.

1. Select the observation context, such as desktop, mobile, dark mode, WCAG, or multi-page crawling.

1. Prepare the matching browser environment.

1. Extract machine-readable JSON and preserve the raw evidence.

1. Interpret measured values conservatively, distinguishing observations from recommendations.

1. Export the result for the intended destination, such as a token pipeline, AI-readable design guide, Tailwind project, or review artifact.

1. Verify the output and document the URL, timestamp, viewport, browser, flags, and limitations.

## Design drift in CI

Dembrandt can compare a candidate extraction against a committed baseline:

```bash
dembrandt https://app.example.com \
  --compare baseline.json \
  --json-only > drift.json
```

The skill teaches agents to distinguish design drift from extraction failure. In a compare run, exit code `1` indicates drift, while exit codes `2` and `67` indicate extraction/browser failure or a retryable navigation timeout respectively. Baselines should be captured with the same Dembrandt version, browser, routes, viewport, theme, and authentication context as the candidate.

For GitHub Actions, the official action can wrap extraction, comparison, browser setup, and pull-request annotations:

```yaml
- uses: dembrandt/dembrandt@v0.28.0
  with:
    url: https://preview.example.com
    baseline: .dembrandt/baseline.json
```

## Responsible use

Use this skill only for websites that the user owns or is authorized to inspect, and only where automated access is permitted. Respect Terms of Service, `robots.txt`, rate limits, copyright, and third-party brand rights. Do not use extracted material to reproduce a third-party brand identity without permission. Treat cookies, API keys, logos, font URLs, and extracted brand assets as sensitive.

Dembrandt observes rendered DOM and CSS evidence. Canvas/WebGL interfaces may not be analyzable, dynamically loaded content may be missed, dark mode must be requested explicitly, and hover or focus behavior is not equivalent to a complete interactive usability test.

## Included files

| File | Purpose |
| --- | --- |
| `SKILL.md` | Core skill metadata, workflow, routing rules, limitations, and output discipline |
| `references/mcp-tools.md` | MCP configuration, tool catalog, job polling, and privacy guidance |
| `references/cli-and-exports.md` | CLI setup, flags, browser options, and export-format selection |
| `references/ci-drift.md` | Baselines, drift comparison, exit codes, and CI reporting |

## License and upstream project

This agent skill is an adaptation of the workflow and public documentation of [Dembrandt](https://github.com/dembrandt/dembrandt). Consult the upstream project for the current CLI implementation, release information, and license terms.

## Links

- [Dembrandt GitHub repository](https://github.com/dembrandt/dembrandt)

- [Dembrandt documentation](https://github.com/dembrandt/dembrandt/tree/main/docs)

- [Dembrandt MCP server](https://github.com/dembrandt/dembrandt/blob/main/mcp-server.ts)

- [Dembrandt CI documentation](https://github.com/dembrandt/dembrandt/blob/main/docs/ci.md)
