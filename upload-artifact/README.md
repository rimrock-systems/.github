# `rimrock-systems/.github/upload-artifact`

Composite action wrapping `actions/upload-artifact@v4` with Rimrock org defaults, so
CI artifacts stop filling the **org-shared** Actions storage quota (WiFiMaster#255).

Bakes in **`retention-days: 14`** (vs GitHub's 90-day default) and
**`if-no-files-found: ignore`**. Use it everywhere instead of calling
`actions/upload-artifact` directly.

## Usage

```yaml
- name: Upload CI logs
  if: always()                 # still capture on failure
  continue-on-error: true      # REQUIRED — an upload must never fail the job / hide a real failure
  uses: rimrock-systems/.github/upload-artifact@main
  with:
    name: ci-logs
    path: logs/
    # retention-days: 14       # default; override per artifact if you must
```

### Heavy artifacts (coverage, build archives) → labeled builds only

Keep the coverage **test + threshold gate** running on every PR; upload the **report**
only on labeled / TestFlight builds so it isn't stored on every run:

```yaml
- name: Upload coverage report
  if: startsWith(github.ref, 'refs/tags/')   # or a PR label — match your TestFlight scheme
  continue-on-error: true
  uses: rimrock-systems/.github/upload-artifact@main
  with:
    name: coverage-report
    path: coverage/
    retention-days: 30
```

## Why `continue-on-error` is on the caller, not baked in

GitHub composite-action steps don't support `continue-on-error`, so it must live on the
calling step. The two-line pattern — `continue-on-error: true` + this action — is the org
standard: **storage stays bounded, and an upload hiccup never reddens CI.**

## Pinning

Examples use `@main`. Once this lands, cut a **`v1`** tag and pin callers to
`@v1` for stability.

## Companion org/repo settings (not per-step)

- **Org default retention:** Settings → Actions → "Artifact and log retention period" → **14 days**.
- **Don't upload giant logs.** NerdFontInstaller produced a single **~384 MB** `ci-logs`
  artifact — trim what goes in (don't tar DerivedData / full `xcodebuild` logs; prefer
  `if: failure()` for logs).

Tracking issue: **WiFiMaster#255**. Authored by infrabot (ops).
