# #82439: Upgrade React 19 to 19.2.8, read version from package.json

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jsnajdr
- **Labels:** `[Type] Bug`
- **Merged:** [`28d799a`](https://github.com/WordPress/gutenberg/commit/28d799a42a664e4811a2df222b2eecd8c8a35dc5)
- **Discussion:** [#82439](https://github.com/WordPress/gutenberg/pull/82439) · 6 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Gutenberg's vendored React 19 bundle is now officially at 19.2.8, and the vendor build reads the React version from the installed packages' `package.json` instead of hardcoding it. Gutenberg 23.9 had accidentally shipped 19.2.8 (via an `npm dedupe` side effect of the `^19.2.7` range) while the script URLs still carried a hardcoded `v=19.2.7`. That let browsers pair a cached `react` with an uncached `react-dom` from different versions, producing React error 527 (version mismatch).

## Impact

- **Site owners:** Sites running the React 19 experiment on Gutenberg 23.9 could hit `react.dev/errors/527` (`19.2.8` vs `19.2.7`) when only one of the `react` / `react-dom` scripts was cached. The fix is cherry-picked to `release/23.9` for a proposed 23.9.1 point release; updating resolves it.
- **Plugin & theme developers:** No API change. Asset `version` strings for `react`, `react-dom`, `react-jsx-runtime` and their `-19` handles now track the real bundled React version (e.g. `18.3.1`, `19.2.8`), so cache-busting query strings change when the bundled React changes.
- **Gutenberg contributors/build maintainers:** The React versions in `build-vendors.mjs` no longer need manual edits. The React 19 range in `tools/react-19/package.json` is bumped to `^19.2.8`.
- **Action required:** None beyond updating the plugin.

## Technical details

Changes in `tools/build-scripts/packages/build-vendors.mjs`:

- A new `getReactVersion( pkg )` helper resolves `${pkg}/package.json` from the workspace via `createRequire`, then resolves `react/package.json` from that vendor package's location and returns its `version`.
- Top-level `await` computes `REACT_18_VERSION` (from `@wordpress/react-18`) and `REACT_19_VERSION` (from `@wordpress/react-19`), which replace the hardcoded `'18.3.1'` / `'19.2.7'` literals in every `VENDOR_SCRIPTS` entry.
- The JSDoc for `generateAssetFile`'s `config.version` is corrected to "Bundled React version (e.g. `18.3.1`)".

In `tools/react-18/package.json` and `tools/react-19/package.json`, `exports` gains `"./package.json": "./package.json"`, which is needed for the `resolve` call to succeed under package exports. React 19's `react` and `react-dom` ranges move from `^19.2.7` to `^19.2.8`, with `package-lock.json` updated to match.

```js
// before
version: '19.2.7',
// after
version: REACT_19_VERSION, // read from installed react/package.json
```

The ranges remain caret semver, so a future `npm dedupe` could still move the installed version. The version string now follows the installed package, so URLs stay accurate.

## Contribution

Opened by @jsnajdr as a fix for a regression introduced in #81870 and shipped in Gutenberg 23.9; he argued it warranted a 23.9.1 point release and later cherry-picked it to `release/23.9`. @tyxla asked how to prevent recurrence. @jsnajdr answered that reading the version from `package.json` removes the root cause, and that pinning exact versions would prevent accidental upgrades but the project relies on the lockfile instead, per an earlier discussion with @manzoorwanijk.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
