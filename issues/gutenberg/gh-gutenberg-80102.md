# #80102: Fix: Critical error related to simple-git

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @im3dabasia
- **Labels:** `npm Packages`, `[Type] Security`
- **Merged:** [`39ff578`](https://github.com/WordPress/gutenberg/commit/39ff578f357eb4f6ff1ba2eebe13933ed1d6c185)
- **Discussion:** [#80102](https://github.com/WordPress/gutenberg/pull/80102) · 8 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Upgrades the `simple-git` npm package from `3.24.0` to `3.32.3` across three internal tooling packages to resolve a critical Dependabot security alert. The change patches a known vulnerability in a build-time dependency used by the Gutenberg plugin and WordPress development tooling, preventing potential security risks during local development and release workflows.

## Impact

- **Gutenberg contributors & plugin/theme developers:** No runtime impact. Requires running `npm install` or `npm update` in the workspace to pull the patched `package-lock.json`.
- **Hosting & platform teams:** No action required for production sites, as `simple-git` is strictly a development and release utility.
- **CI/CD pipelines:** Ensure build environments use the updated lockfile to avoid triggering the patched vulnerability during dependency resolution.

## Technical details

The diff updates the `simple-git` version constraint from `^3.24.0` to `^3.32.3` in `packages/env/package.json`, `tools/release/package.json`, and `tools/validation/package.json`. The `package-lock.json` reflects this upgrade, introducing two new transitive dependencies—`@simple-git/args-pathspec` (v1.0.3) and `@simple-git/argv-parser` (v1.1.1)—under the `simple-git` package entries in `packages/env/node_modules`, `tools/release/node_modules`, and `tools/validation/node_modules`. The change resolves Dependabot alert `dependabot/523` without modifying public APIs, hooks, or runtime behavior, as the package is exclusively used for Git operations during build and release processes.

## Contribution

Opened by @im3dabasia to address critical Dependabot alert `dependabot/523`. During review, @im3dabasia asked whether to batch other outstanding Dependabot alerts; @desrosj advised handling them separately to maintain atomic commits, noting that several existing Dependabot PRs had stalled due to configuration changes. The PR was merged with co-authors @im3dabasia, @manzoorwanijk, @t-hamano, and @desrosj.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
