# #79014: Framework: Remove invalid stale nested npm package references

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aduth
- **Labels:** `[Type] Code Quality`
- **Merged:** [`275320e`](https://github.com/WordPress/gutenberg/commit/275320e0555a8ba04531e464d87af7f46920abd4)
- **Discussion:** [#79014](https://github.com/WordPress/gutenberg/pull/79014) · 8 comments · 0 reactions

## Summary

This pull request removes stale and invalid nested package references from `package-lock.json`, resolving `npm error: invalid` warnings reported when running `npm ls --depth=0`. The cleanup eliminates dangling copies of `@types/node` and `@wordpress/scripts` that were left over from earlier tooling updates, aligning the lockfile with the actual resolved dependency graph. Reducing these unused transitive dependencies improves npm resolution performance and narrows the supply-chain attack surface for the development environment.

## Impact

- **Plugin & Theme Developers / Core Contributors:** No action required. The changes are strictly confined to `package-lock.json` and do not alter any public APIs, build outputs, or package manifests.
- **Build/CI Tooling:** A cleaner lockfile ensures faster npm resolutions and prevents validation failures when running strict npm versions (e.g., v12+). No migration steps are needed; a fresh `npm install` will automatically reify the corrected state.

## Technical details

The unified diff in `package-lock.json` removes dozens of orphaned `node_modules/...` entries that no longer satisfy any active package's dependency tree. Specific stale entries removed include dev dependencies like `@humanwhocodes/config-array`, `@types/glob`, `clean-webpack-plugin`, `eslint-config-prettier`, and `babel-runtime` (along with their sub-dependencies), as well as two explicitly flagged dangling packages: `core-data/node_modules/@types/node` and `tools/validation/node_modules/@wordpress/scripts`. These entries were manually scrubbed from the JSON and the file was regenerated via `npm install` to reify a consistent lockfile. No hooks, filters, REST routes, or block.json fields are modified.

## Contribution

Opened and merged by @aduth (commit `275320e`) alongside co-authorship from @manzoorwanijk. The issue was surfaced while testing npm v12's stricter remote fetching behavior in the Gutenberg monorepo. After investigating root causes with AI-assisted tooling, stale entries were manually pruned from the lockfile and committed to align the dependency graph with current workspace resolutions.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
