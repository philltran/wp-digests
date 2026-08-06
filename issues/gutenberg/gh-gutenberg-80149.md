# #80149: Widget Primitives: Ship as a script module

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @retrofox
- **Labels:** `[Type] Enhancement`, `[Package] Widget primitives`
- **Merged:** [`e7690ec`](https://github.com/WordPress/gutenberg/commit/e7690ecd9fe292d76b56248ad811f1383f96f28e)
- **Discussion:** [#80149](https://github.com/WordPress/gutenberg/pull/80149) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `@wordpress/widget-primitives` package is now shipped as a WordPress script module instead of a standard bundled package. This change ensures the package's module-scope state (the field type registry `Map`) is shared across all consumers via the import map, preventing duplicate instances and isolated state when multiple bundles import it.

## Impact

- **Plugin & theme developers / external package consumers:** No action required. The change is handled by the build pipeline and import map resolution.
- **Gutenberg package consumers & internal tooling:** Consumers that import `@wordpress/widget-primitives` will now externalize it to the import map rather than bundling a local copy. If you are manually building or vendoring Gutenberg packages, ensure your build configuration respects the new `wpScriptModuleExports` field to maintain the shared instance guarantee.

## Technical details

The diff adds `"wpScriptModuleExports": "./build-module/index.mjs"` to `packages/widget-primitives/package.json`. During the `wp-build` process, this triggers the generation of `build/modules/widget-primitives/index.min.js` and registers the package in `build/modules.php`. Consumers that previously inlined the package into their bundle will now resolve it as a shared script module via the import map. Because the field type registry is a module-level `Map`, evaluating the package exactly once per page load guarantees that all consumers reference the same registry instance instead of isolated copies.

## Contribution

Opened and merged by @retrofox with co-authorship from @chihsuan. During review, the author initially explored lazy-loading the dashboard's field type registration to reduce init module weight, but found that `@wordpress/ui` (which provides the autocomplete stack) was not yet a script module, causing it to inline into the critical path. The approach was adjusted to keep registration in the route bundle and focus solely on externalizing `@wordpress/widget-primitives` as a script module to solve the state duplication issue.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
