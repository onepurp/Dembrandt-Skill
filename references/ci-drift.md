# Dembrandt CI and design drift

Use this reference when enforcing a design baseline in CI, comparing preview and production, or deciding whether a failed run indicates drift versus an extraction problem.

## Baseline discipline

Capture the baseline and candidate with the same Dembrandt version, routes, viewport, color scheme, browser, wait behavior, authentication context, and relevant flags. Store the raw baseline JSON in the repository or another controlled artifact location.

```bash
# Capture once, from the intended reference environment
dembrandt https://app.example.com --json-only > baseline.json

# Compare in CI and produce machine-readable drift
dembrandt https://app.example.com \
  --compare baseline.json \
  --json-only > drift.json
```

A compare result includes a score, status, summary, and token-level `changes[]` entries under the drift data. Each change identifies a category and whether a token was changed, added, or removed, together with before/after values and a delta where applicable.

When a change is intentional, approve it explicitly rather than silently replacing the baseline:

```bash
dembrandt https://app.example.com --compare baseline.json --approve
```

Do not approve after an extraction failure, timeout, browser mismatch, or context change. Re-establish a valid candidate first.

## Exit-code handling

| Code | Meaning | Agent response |
|---|---|---|
| `0` | Extraction succeeded, or compare is stable | Continue or publish the stable report |
| `1` | Compare found design drift | Read the drift JSON and surface the changed tokens; fail the gate unless intentional |
| `2` | Extraction or browser failure | Fix URL, browser, dependencies, or auth; do not call it design drift |
| `67` | Navigation or connection timeout | Retry with `--slow` or investigate availability |

With `--json-only`, failures also emit machine-readable error information containing a code and message. Preserve stdout and stderr as CI artifacts when diagnosing a runner.

## GitHub Actions path

The official action wraps extraction, comparison, browser installation, and PR annotations:

```yaml
- uses: dembrandt/dembrandt@v0.28.0
  with:
    url: https://preview.example.com
    baseline: .dembrandt/baseline.json
```

Common inputs are `url`, `baseline`, optional `key` for cloud snapshot sync, and optional `args` such as `--wcag` or `--crawl 3`. The action exposes a report path and drift score. Pin a release tag to avoid silently changing the gate’s behavior.

For other CI systems, use the same extract → compare → branch-on-exit-code core, then parse the drift JSON to create a merge-request note, build annotation, chat notification, or issue.

## Reporting guidance

A useful drift report names the source URL, baseline and candidate contexts, score, verdict, affected categories, and the exact changed tokens. Separate intentional design changes from regressions, and distinguish missing evidence from a measured removal. Attach the HTML report when human review benefits from visual context.

## Source

[1]: https://github.com/dembrandt/dembrandt/blob/main/docs/ci.md "Dembrandt CI documentation"
[2]: https://github.com/dembrandt/dembrandt/blob/main/examples/drift-gate.yml "Dembrandt drift-gate workflow example"
