# Dembrandt MCP tools

Use this reference when Dembrandt is exposed through an MCP-compatible client. The documented server is started with `npx -y --package dembrandt dembrandt-mcp` over stdio.

## Configure the server

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

Extraction tools accept `url`, optional `slow`, `sync`, `mobile`, and `cookie`. `get_design_tokens` additionally accepts `darkMode` and `wcag`; the focused tools accept the options relevant to their category. A URL without a scheme is normalized to HTTPS by the server.

## Extraction tools

| Tool | Use for | Returned focus |
|---|---|---|
| `get_design_tokens` | Complete design-system extraction | Full extraction object |
| `get_color_palette` | Brand and UI color work | Semantic colors, palette, CSS variables, state colors, optional WCAG |
| `get_typography` | Type-system work | Families, fallbacks, sizes, weights, line heights, tracking, sources |
| `get_component_styles` | UI component reconstruction | Buttons, inputs, links, badges, state styling |
| `get_surfaces` | Surface and elevation analysis | Radii, borders, shadows with element context |
| `get_spacing` | Layout scale analysis | Margin, padding, frequency, pixel/rem values, grid detection |
| `get_brand_identity` | Identity and implementation context | Site name, logo, favicons, frameworks, icons, breakpoints |

By default, each extraction returns `{job_id, status:"queued"}`. Poll the job rather than launching repeated duplicate extractions. Set `sync: true` only when the user needs the result directly and the wait is acceptable.

## Pure analysis and export tools

These tools do not launch a browser and operate on an extraction object, normally the `result` from `get_job_status`.

| Tool | Inputs | Purpose |
|---|---|---|
| `compute_drift` | `baseline`, `candidate`, optional `failThreshold` | Returns a 0–100 drift score, verdict, category scores, and changed/added/removed tokens |
| `get_findings` | `result` | Lints for contrast failures, near-duplicate colors, off-scale spacing, radius sprawl, and duplication |
| `export_dtcg` | `result` | Converts colors, typography, spacing, radius, border, and shadow data to DTCG with provenance |
| `generate_design_md` | `result` | Produces a human-readable `DESIGN.md` design reference |
| `render_report` | `result`, optional `drift` | Produces self-contained HTML suitable for an offline file or CI artifact |

Recommended sequence: extract once, poll once to completion, run `get_findings`, then choose drift or an export. Keep the raw extraction so later exports do not require another page load.

## Job-control tools

| Tool | Use |
|---|---|
| `get_job_status` | Poll a job ID; returns queued, running, completed, failed, or cancelled, and includes the result on completion |
| `list_jobs` | Inspect session jobs and timestamps; completed jobs are retained for about one hour |
| `cancel_job` | Cancel a queued job; it does not stop a job already running |

The server permits two active extraction jobs. If a job fails, surface the error and adjust the context before retrying: use `slow` for timeouts, verify the URL for resolution failures, and install the matching browser when launch fails.

## Safety and privacy

Use `cookie` only for an authorized authenticated page and never place the cookie value in a report, prompt transcript, log, or commit. Do not use the server to bypass access controls. Treat returned logos, favicons, font URLs, and brand assets as third-party material that may have usage restrictions.

## Source

[1]: https://github.com/dembrandt/dembrandt/blob/main/mcp-server.ts "Dembrandt MCP server source"
[2]: https://github.com/dembrandt/dembrandt#ai-agent-integration-mcp "Dembrandt MCP setup in the project README"
