# #80058: Scripts: Make 'test-e2e' run Playwright and remove Puppeteer

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Enhancement`, `[Tool] WP Scripts`
- **Merged:** [`e7c7d6d`](https://github.com/WordPress/gutenberg/commit/e7c7d6d91a430f05a9e166456b7a7a5cef749617)
- **Discussion:** [#80058](https://github.com/WordPress/gutenberg/pull/80058) · 4 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `wp-scripts` package has replaced Puppeteer with Playwright as the default runtime for the `test-e2e` command. All Puppeteer-related dependencies and infrastructure have been removed from `@wordpress/scripts`, aligning with Gutenberg's core migration to Playwright for improved debugging and test authoring.

## Impact

- Plugin & theme developers: The `test-e2e` command now executes Playwright tests. Existing Puppeteer-based E2E test suites will fail and require migration to Playwright syntax and APIs.
- Developers using only Jest unit tests or not running E2E tests: No action required.
- The `test-playwright` alias remains available as a direct replacement.

## Technical details

The change removes ~29 packages from `package-lock.json` (including `expect-puppeteer`, `jest-dev-server`, `cwd`, `find-process`, `expand-tilde`, `homedir-polyfill`, and their transitive dependencies) and reclassifies several previously production dependencies as `"dev": true`. The `test-e2e` CLI entry point now delegates to Playwright's runner. Documentation in `docs/getting-started/devenv/get-started-with-wp-scripts.md` is updated to reflect Jest for unit testing and Playwright for E2E. A follow-up deprecation for `jest-puppeteer-axe` was noted in review but deferred to a subsequent PR.

## Contribution

Opened and merged by @Mamaduka, with co-authors @manzoorwanijk and @youknowriad. The PR closed #60357 and streamlined the testing toolchain by removing legacy Puppeteer infrastructure. Reviewers noted that the related `jest-puppeteer-axe` package should be deprecated in a subsequent PR, which was acknowledged but deferred.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
