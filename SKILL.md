---
name: dembrandt
description: Extract and analyze live website design systems into design tokens, brand identity, typography, colors, spacing, surfaces, components, WCAG reports, DESIGN.md, Tailwind v4 themes, DTCG JSON, drift reports, and HTML or brand-guide artifacts. Use when asked to reverse-engineer, document, compare, audit, or enforce a website’s visual system from a URL, or when Dembrandt CLI/MCP workflows are needed.
---

# Dembrandt

Use Dembrandt to measure a website’s **observed design system** rather than inventing one. It renders the target with a real browser, reads computed DOM styles, and returns evidence for colors, typography, spacing, borders, radii, shadows, motion, components, responsive behavior, branding, frameworks, and icons.

> **Authorization rule:** Analyze only sites the user owns or is authorized to inspect, and only where automated access is permitted. Respect the site’s Terms of Service, `robots.txt`, rate limits, copyright, and third-party brand rights. Treat cookies, API keys, and extracted brand assets as sensitive.

## Choose the execution route

1. **Prefer the Dembrandt MCP tools** when they are available in the current agent environment. Use the extraction tools for live pages, then poll the returned job with `get_job_status` unless the user explicitly needs a blocking result.
2. **Use the CLI through the shell** when MCP is unavailable, when files must be written locally, or when a reproducible CI command is required. Use `npx dembrandt ...` for an ephemeral install or the globally installed `dembrandt` command for a project that already depends on it.
3. **Use pure analysis tools** such as `compute_drift`, `get_findings`, `export_dtcg`, or `generate_design_md` on an existing extraction instead of reloading the website.
4. Read [references/mcp-tools.md](references/mcp-tools.md) for the MCP tool contract, [references/cli-and-exports.md](references/cli-and-exports.md) for CLI options and output selection, and [references/ci-drift.md](references/ci-drift.md) for baseline and CI work.

## Standard extraction workflow

1. **Define the question and scope.** Decide whether the user needs a full design system, a focused category, a responsive comparison, an accessibility audit, a brand guide, or a drift check. Confirm the URL and authorization when it is not obvious.
2. **Select the observation context.** Use desktop by default. Add `mobile` when responsive behavior matters, `dark-mode` when the dark theme is in scope, `wcag` for contrast evidence, and `crawl` or explicit paths when one page is insufficient. Use `sitemap` when navigation links are incomplete.
3. **Prepare the browser.** Dembrandt requires Node.js 18 or later and a browser binary matching its bundled `playwright-core`. Run `dembrandt install-browser` once before the first CLI extraction. In Linux CI, install the matching browser system dependencies or use a compatible Playwright container. If a browser already exists, use `BROWSER_CDP_ENDPOINT` with Chromium.
4. **Extract machine-readable evidence.** Prefer `--json-only` for agent processing and `--save-output` when a durable artifact is needed. Use `--slow` for heavy single-page applications. Preserve the raw extraction before transforming it.
5. **Interpret conservatively.** Distinguish observed values from recommendations. Treat confidence, frequency, semantic context, viewport, color scheme, and page coverage as evidence. Do not fill omitted sections with guessed defaults. Account for the limitations below before making design claims.
6. **Export for the destination.** Choose DTCG for token pipelines, `DESIGN.md` for AI-readable project guidance, Tailwind v4 `@theme` CSS for a Tailwind project, `--brand-guide` for a printable summary, and `--html` or `render_report` for a self-contained review artifact.
7. **Verify the result.** Check that the output exists, is parseable, and matches the requested scope. For a comparison, use the same browser, viewport, routes, wait behavior, and Dembrandt version for baseline and candidate whenever possible.

## Quick command selection

| Need | Preferred command or operation | Result |
|---|---|---|
| Full extraction | `dembrandt URL --json-only` | Raw extraction JSON on stdout |
| Persist JSON | `dembrandt URL --save-output` | Timestamped JSON under `output/` |
| W3C token pipeline | `dembrandt URL --dtcg` | `.tokens.json` DTCG export |
| AI-readable design guidance | `dembrandt URL --design-md` | `DESIGN.md` |
| Tailwind v4 project | `dembrandt URL --tailwind` | Observed-value `@theme` CSS |
| Accessibility review | `dembrandt URL --wcag` | DOM-based contrast results and grades |
| Responsive or theme comparison | Add `--mobile` or `--dark-mode` | Context-specific extraction |
| Multi-page system | `dembrandt URL --crawl N` or explicit paths | Merged extraction with cross-page evidence |
| Printable summary | `dembrandt URL --brand-guide` | Brand-guide PDF |
| Drift gate | `dembrandt URL --compare baseline.json --json-only` | Drift JSON and exit status |

## Decision rules and failure handling

Use `--slow` before concluding that a JavaScript-heavy site is inaccessible. If Chromium is blocked or unreliable, install Firefox and retry with `--browser=firefox`; use `--stealth` only when the user is authorized to access the site and the site’s rules permit it. Use `--no-sandbox` only in an appropriately isolated container or CI runner. For authenticated pages, pass a cookie only when the user has supplied an authorized session value; never expose it in logs or final output.

Treat extraction failure as different from design drift. In CLI drift gates, exit code `1` means drift, `2` means extraction or browser failure, and `67` means a navigation timeout that is generally retryable with `--slow`. Do not approve or overwrite a baseline merely because extraction failed or because the candidate used a different observation context.

Dembrandt does not fully analyze Canvas/WebGL-rendered interfaces, may miss dynamically loaded content, does not automatically infer dark mode, and reads hover/focus states primarily from CSS rather than complete interactive behavior. The default desktop viewport is 1920×1080; the mobile profile is 390×844. Report these constraints whenever they could affect the user’s conclusion.

## MCP operating pattern

Extraction MCP tools return a `job_id` by default because browser work can take 15–40 seconds. Poll `get_job_status` until the status is `completed`, then pass the returned extraction to the pure analysis or export tools. Set `sync: true` only when blocking is acceptable and the result is small enough to use directly. Keep concurrent live extractions modest; the server limits active jobs to two.

Use focused tools when the user asks for one category, and use `get_design_tokens` when the complete context is needed. After extraction, prefer this sequence for quality work: `get_findings` to identify problems, `compute_drift` to compare against a baseline, then `export_dtcg`, `generate_design_md`, or `render_report` for delivery. See [references/mcp-tools.md](references/mcp-tools.md) for the exact catalog.

## Output discipline

Label every deliverable with its source URL, extraction context, timestamp, and whether it is raw evidence or an interpretation. Keep raw JSON alongside derived files. When summarizing colors or typography, retain the original notation and semantic context even if the user requests a transformed format. Tailwind output contains observed values only: do not present absent shade ramps, hover variants, or on-color values as if Dembrandt measured them.

## References

[1]: https://github.com/dembrandt/dembrandt "Dembrandt source repository"
[2]: https://github.com/dembrandt/dembrandt/blob/main/docs/usage.md "Dembrandt usage reference"
[3]: https://github.com/dembrandt/dembrandt/blob/main/docs/ci.md "Dembrandt CI and drift reference"
[4]: https://github.com/dembrandt/dembrandt/blob/main/mcp-server.ts "Dembrandt MCP server implementation"
