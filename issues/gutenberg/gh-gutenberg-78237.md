# #78237: Performance Tests: log timestamps, optimize build overhead

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jsnajdr
- **Labels:** `[Type] Performance`, `[Package] E2E Tests`
- **Merged:** [`2f4eb56`](https://github.com/WordPress/gutenberg/commit/2f4eb56e2f1be1ba94cc611cf57bd07ed2ba5b39)
- **Discussion:** [#78237](https://github.com/WordPress/gutenberg/pull/78237) · 6 comments · 1 reactions

## Summary

This PR speeds up the Gutenberg performance-testing pipeline, which was taking ~36 minutes in CI despite the actual `npm run test:performance` suite running in only ~6 minutes per branch — meaning ~24 of those minutes were pure overhead (checkout, build, Docker setup). It first adds timestamps to every logged step in `bin/plugin/commands/performance.js` to expose where time is spent, then applies three build/runtime optimizations that cut the end-to-end run from ~36 to ~29 minutes. The changes are confined to the perf test runner and the `e2e-test-utils-playwright` package; there is no change to shipped plugin code (reported size change: 0 B).

## Impact

This is a developer-tooling/CI change only. No runtime, REST, block, or theme behavior is affected.

- **Site owners / headless & REST consumers:** No action required — nothing in the shipped build changes.
- **Gutenberg contributors / core developers:** Performance test runs (locally and in CI) are faster. The perf log output now includes per-step timestamps.
- **Consumers of `@wordpress/e2e-test-utils-playwright`:** Note that this package no longer ships a built CJS version in `build/` — it now exports its original `.ts` source files directly, relying on Playwright's internal esbuild to strip types/transpile JSX at runtime. Anything importing this package outside a Playwright/esbuild context (i.e. expecting compiled CJS) would need to handle TS sources itself.

## Technical details

In `bin/plugin/commands/performance.js`, a `perfStartTime = performance.now()` is captured at module load and the logging helper is updated to prefix each logged step/substep with an elapsed timestamp.

The three optimizations:

1. **No build in the test-runner folder.** The runner now only runs `npm ci`; the Playwright run executes directly from raw sources (Playwright uses esbuild to strip types from `.ts` and transpile JSX). To enable this, the build step was removed from `e2e-test-utils-playwright`, which now exports `.ts` files instead of a compiled `build/` CJS output.
2. **`--skip-types` when building each branch.** Building the actual Gutenberg plugin for the `trunk` and PR branches now passes `--skip-types` to `npm run build`, since only the JS artifacts are needed — consistent with existing usage elsewhere.
3. **Boot the server container once per branch.** Instead of calling `wp-env start`/`wp-env stop` around each suite (~10s each), the Docker container is booted once per branch, all suites run against it, then it shuts down. Only multiple test "rounds" remain isolated.

Net effect: end-to-end perf CI run drops from ~36 min to ~29 min.

## Contribution

Authored and merged by **@jsnajdr**, with props to **@ellatrix** and **@Mamaduka**; merged as commit `2f4eb56`. The PR began as an investigation (adding timestamps to find the overhead) and grew into the optimizations once the breakdown was clear. In discussion, **@ellatrix** suggested also summarizing per-section elapsed time; @jsnajdr proposed a possible "delta" format (e.g. `[33:40.0 +13.4]`) but noted that pasting two logs into an LLM already yields a good summary. **@tyxla** approved. Flaky tests in `classic.spec.js` and `publish-panel.spec.js` were flagged by CI but unrelated to the change.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
