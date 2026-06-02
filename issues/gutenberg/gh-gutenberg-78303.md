# #78303: wp-build: Replace getter-based exports with data properties

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ellatrix
- **Labels:** `[Type] Build Tooling`
- **Merged:** [`2c4ff33`](https://github.com/WordPress/gutenberg/commit/2c4ff33115c92fcdc90618fb9d16c3bf90e59cea)
- **Discussion:** [#78303](https://github.com/WordPress/gutenberg/pull/78303) · 6 comments · 0 reactions

## Summary

The Gutenberg build pipeline now routes every `wpScript` package through a synthetic CJS entrypoint — `module.exports = { ...require('<entry>') }` — so that the IIFE result assigned to `wp.<pkg>` is a plain object with data properties instead of esbuild's getter-based namespace object. Previously, esbuild's IIFE output wrapped each export in a non-configurable `Object.defineProperty` getter to preserve ESM live-binding semantics; those getters remained active at runtime even though the bindings are stable after the IIFE completes, adding per-access overhead across every hook lookup and selector read. Packages marked as `wpScriptDefaultExport` now use `module.exports = require('<entry>').default`, unwrapping the namespace at build time and removing the previous post-hoc unwrap step. The change is a build-tooling optimisation with no API surface changes.

## Impact

**Plugin & theme developers**
- No action required. The public API of every `wp.*` global (`wp.element`, `wp.data`, `wp.compose`, `wp.blocks`, etc.) is unchanged; properties are still enumerable and accessible by the same names.
- Code that uses `Object.getOwnPropertyDescriptor(wp.data, 'someExport')` and branches on whether the descriptor is a getter vs. a data property will now see a data property — this is an edge case but technically observable.

**Hosting & platform teams**
- Editor performance improves measurably out of the box once the bundled assets ship with WordPress core. Measured regressions against trunk: `firstBlock` −4.96% (post-editor) / −2.79% (site-editor), `type` −16.21%, `domContentLoaded` −5.07%.
- No server-side or infrastructure changes required.

**Headless & REST consumers**
- No impact; this change affects only the browser-side IIFE bundles.

## Technical details

The change is isolated to the Gutenberg build tooling, specifically the script responsible for building `wpScript` packages as IIFE globals.

**Root cause of the old behaviour:** esbuild's IIFE format emits exports as non-configurable getters:
```js
// esbuild-generated output (before)
var wp_data = (() => {
  // ...
  Object.defineProperty(exports, 'dispatch', { get: () => dispatch, enumerable: true });
  // ...
  return exports;
})();
```
Each read of `wp.data.dispatch` (and every other export) incurred a getter invocation — roughly 2× slower than a direct property read in V8. With hundreds of thousands of such reads during editor mount and each keystroke (hooks, selectors, component renders), the cumulative cost was measurable.

**Fix:** The `entryPoints` array in the esbuild config is replaced with a `stdin` synthetic CJS entry for each package:
```js
// new synthetic entrypoint fed via stdin
module.exports = { ...require('@wordpress/data') };
```
Esbuild evaluates the spread inside the IIFE, so the global assigned to `wp.data` is a plain object with data properties from the start — no getters survive into the runtime output.

For packages flagged as `wpScriptDefaultExport`, the entrypoint becomes:
```js
module.exports = require('@wordpress/package').default;
```
This replaces the previous post-processing step that unwrapped `.default` after the fact.

The pattern is already used in `bin/packages/build-vendors.mjs` for the `react-dom` global, making this an extension of an established convention rather than a novel approach.

All `build/scripts/*/index.min.js` artifacts gained ~26–48 B (the spread wrapper), totalling +2.35 kB across the entire build — a negligible size trade-off for the runtime gain.

## Contribution

Opened and merged by @ellatrix. The PR notes an iterative process: a footer-based approach in the first commit (`291a030950a`) preceded the final stdin/spread approach; a revert commit (3rd commit) appears in the history, suggesting at least one intermediate approach was tried and rolled back before the spread-based solution was settled on. @jsnajdr is credited as a co-author in the merge commit attribution. The PR carried the `[Type] Build Tooling` label and received 6 comments, primarily automated (size report, props bot). The author notes use of Claude Code assistance with personal review and verification.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
