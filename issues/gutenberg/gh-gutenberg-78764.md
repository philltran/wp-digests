# #78764: feat: Migrate the browserlintrc file to `packages/postcss-plugins-preset`

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @im3dabasia
- **Labels:** `[Type] Enhancement`, `[Tool] Browserslist config`
- **Merged:** [`cd08f65`](https://github.com/WordPress/gutenberg/commit/cd08f6569f6dad0016546cc050965d3e7cb2290b)
- **Discussion:** [#78764](https://github.com/WordPress/gutenberg/pull/78764) · 2 comments · 0 reactions

## Summary

The root `.browserslistrc` file has been removed from the Gutenberg monorepo, and its configuration has been migrated directly into `packages/postcss-plugins-preset`. This consolidates build tooling dependencies and eliminates a redundant config file as part of the broader Phase 7 cleanup tracking issue #75041.

## Impact

['**Plugin & theme developers / Site owners**: No action required. This is an internal build toolchain adjustment with no impact on public APIs, block editor features, or runtime behavior.']

## Technical details

The diff deletes the root `.browserslistrc` file (which previously contained `extends @wordpress/browserslist-config`) and wires the config explicitly into the PostCSS preset.

In `packages/postcss-plugins-preset/package.json`, `@wordpress/browserslist-config` is added as a direct dependency:
```json
"@wordpress/browserslist-config": "file:../browserslist-config"
```

In `packages/postcss-plugins-preset/lib/index.js`, Autoprefixer configuration shifts from implicit file discovery to explicit programmatic injection. The `grid: true` option is now paired with `overrideBrowserslist`:
```diff
-	require( 'autoprefixer' )( { grid: true } ),
+	require( 'autoprefixer' )( {
+		grid: true,
+		overrideBrowserslist: require( '@wordpress/browserslist-config' ),
+	} ),
```
This change ensures browser target resolution for CSS prefixing is tightly scoped to the PostCSS preset package rather than relying on root-level browserslist file traversal.

## Contribution

Opened by @im3dabasia and merged as commit `cd08f65` with co-authorship from @manzoorwanijk. The PR was a targeted, low-complexity step in the Phase 7 cleanup workflow for issue #75041. Review discussion was minimal, primarily limited to correcting the linked issue relationship in the PR description (clarifying it as related rather than closing). No alternative implementations or design debates were documented.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
