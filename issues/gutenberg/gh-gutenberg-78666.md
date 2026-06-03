# #78666: @wordpress/theme: deduplicate addFallbackToVar helper

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Code Quality`, `[Package] Theme`
- **Merged:** [`689bc1a`](https://github.com/WordPress/gutenberg/commit/689bc1acf95ba4736d51858276e64c5bea50095f)
- **Discussion:** [#78666](https://github.com/WordPress/gutenberg/pull/78666) · 6 comments · 0 reactions

## Summary

The `addFallbackToVar` CSS variable fallback helper in the `@wordpress/theme` package existed as copy-pasted code in two separate files — `postcss-plugins/add-fallback-to-var.ts` and `postcss-plugins/ds-token-fallbacks.mjs` — each carrying comments warning maintainers to keep them in sync. This PR collapses both into a single canonical implementation at `packages/theme/src/postcss-plugins/add-fallback-to-var.mjs`, making it a generic function parameterized by the token fallback map. `ds-token-fallbacks.mjs` is reduced to a thin pre-binding wrapper. No published API or bundle output changes.

## Impact

**Plugin & theme developers / headless & REST consumers**
- No action required. The three published entry points — `./postcss-plugins/postcss-ds-token-fallbacks`, `./esbuild-plugins/esbuild-ds-token-fallbacks`, and `./vite-plugins/vite-ds-token-fallbacks` — are unchanged on disk and in behavior.
- Bundle size delta: 0 B.

**`@wordpress/theme` package contributors**
- The old `packages/theme/src/postcss-plugins/add-fallback-to-var.ts` is deleted; any internal import pointing at it will need updating. The replacement is `add-fallback-to-var.mjs` in the same directory.

## Technical details

**Before:** Identical regex-and-replacement logic lived in two places:
- `packages/theme/src/postcss-plugins/add-fallback-to-var.ts` — TypeScript source with inline type annotations.
- `packages/theme/src/postcss-plugins/ds-token-fallbacks.mjs` — a second inline copy, with a comment flagging the need to stay in sync.

**After:** A single generic implementation:
```js
// packages/theme/src/postcss-plugins/add-fallback-to-var.mjs
/**
 * @param {Record<string, string>} tokenFallbackMap
 * @param {{ escapeQuotes?: boolean }} [options]
 */
export function addFallbackToVar( tokenFallbackMap, { escapeQuotes = false } = {} ) {
  // single regex+replacement implementation
}
```
`ds-token-fallbacks.mjs` now pre-binds the package's generated fallback map and re-exports:
```js
import { addFallbackToVar as _addFallbackToVar } from './add-fallback-to-var.mjs';
import tokenFallbacks from '../generated/token-fallbacks.mjs';
export const addFallbackToVar = ( opts ) => _addFallbackToVar( tokenFallbacks, opts );
```
The companion unit test file (14-case `add-fallback-to-var` suite + 5-case prebound `ds-token-fallbacks` suite, 38 total) is updated to import from the new `.mjs` source. JSDoc replaces the TypeScript type annotations from the removed `.ts` file. No changes to `block.json`, REST schema, hooks, or DB.

## Contribution

Opened and authored by @ciampo as item **B4** of the `@wordpress/theme` stabilization effort tracked in issue #77462. During review, @aduth raised a pointed question about whether `add-fallback-to-var.ts` had any active callers at the time, implying a simple deletion might have been sufficient rather than a migration. The PR was merged as commit `689bc1a` with @ciampo and @aduth as credited contributors.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
