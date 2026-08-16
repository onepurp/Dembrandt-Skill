# Dembrandt CLI and exports

Use this reference when the task requires exact CLI flags, browser setup, output locations, or an export-format decision.

## Installation and browser setup

Dembrandt requires Node.js 18 or later. The package uses `playwright-core`, so installing the npm package does not download a browser binary.

```bash
npm install -g dembrandt
dembrandt install-browser
dembrandt https://example.com
```

For an ephemeral install:

```bash
npx dembrandt install-browser
npx dembrandt https://example.com --json-only
```

Install Firefox when Chromium is blocked or unreliable:

```bash
dembrandt install-browser firefox
dembrandt https://example.com --browser=firefox
```

In Linux CI, install system dependencies for the exact bundled Playwright version, or use a matching Playwright container. To drive an existing Chromium over DevTools Protocol, set `BROWSER_CDP_ENDPOINT=http://localhost:9222`; CDP is supported only with Chromium.

## Core commands

```bash
# Terminal overview
dembrandt https://example.com

# Agent-friendly JSON
dembrandt https://example.com --json-only

# Persist a timestamped JSON extraction
dembrandt https://example.com --save-output

# Explicit routes merged into one result
dembrandt https://example.com /pricing /docs --json-only

# Discover and merge several pages
dembrandt https://example.com --crawl 5 --json-only

# Use sitemap discovery
dembrandt https://example.com --crawl 10 --sitemap --json-only
```

Multi-page extraction boosts confidence for tokens repeated across pages. Failed pages are skipped rather than aborting the whole run; report the actual pages covered.

## Context and resilience flags

| Flag | Use when | Important behavior |
|---|---|---|
| `--mobile` | Responsive mobile behavior matters | Uses a 390×844 mobile viewport |
| `--dark-mode` | Dark theme is in scope | Explicitly emulates dark color scheme |
| `--slow` | The site is a heavy SPA or hydration is slow | Increases timeouts; retry before declaring failure |
| `--browser=firefox` | Chromium faces bot detection or timeout issues | Install Firefox first |
| `--no-sandbox` | Isolated Docker or CI runner requires it | Do not use casually on an ordinary desktop |
| `--wcag` | Contrast audit is requested | Uses real DOM text/background pairs and grades |
| `--ai` | Experimental ML brand-color detection is requested | Requires optional `onnxruntime-node` |
| `--stealth` | Authorized access needs opt-in anti-detection | Use only where permitted |
| `--locale`, `--timezone` | Locale-sensitive rendering matters | Changes browser fingerprint context |
| `--user-agent`, `--accept-language` | Site behavior depends on request headers | Record the custom context in the report |
| `--screen-size` | Physical screen resolution matters | Does not replace the mobile viewport profile |

## Export formats

| Flag | Output | When to choose |
|---|---|---|
| `--dtcg` | W3C Design Tokens JSON, usually `.tokens.json` | Style Dictionary, Tokens Studio, or token pipelines |
| `--design-md` | `DESIGN.md` | AI-readable project design guidance |
| `--tailwind` | Tailwind v4 `@theme` CSS | Starting theme for a Tailwind project |
| `--brand-guide` | Printable brand-guide PDF | Human review or handoff |
| `--html` | Self-contained HTML report | Offline review or CI artifact |

Tailwind output contains **observed values only**. It intentionally does not create shade ramps, interpolated scale steps, derived hover states, or on-color variants. Treat the emitted CSS as a measured starting point that developers may extend manually.

`--color-format` changes terminal presentation only (`hex`, `rgb`, `lch`, `oklch`, or `source`). It does not change the JSON payload, DTCG, DESIGN.md, HTML, brand guide, or drift comparison.

## Output interpretation

The extraction contains more information than the default terminal display. Preserve the raw JSON when a later task may need complete palette entries, low-confidence colors, CSS custom properties, motion, responsive data, or WCAG details. Do not infer that a missing section means a zero value; it usually means Dembrandt found no sufficient evidence in the observed DOM.

## Source

[1]: https://github.com/dembrandt/dembrandt/blob/main/docs/usage.md "Dembrandt usage documentation"
[2]: https://www.designtokens.org/ "W3C Design Tokens Community Group"
