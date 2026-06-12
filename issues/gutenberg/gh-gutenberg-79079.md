# #79079: Style Engine: Export public TypeScript types

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ralucaStan
- **Labels:** `[Type] Enhancement`, `[Package] Style Engine`
- **Merged:** [`2c4461a`](https://github.com/WordPress/gutenberg/commit/2c4461a1b6f98bbaba8af3f148f5a052ec58c41d)
- **Discussion:** [#79079](https://github.com/WordPress/gutenberg/pull/79079) · 1 comments · 0 reactions

## Summary

The `@wordpress/style-engine` package now explicitly exports the TypeScript signatures `Style`, `StyleOptions`, and `GeneratedCSSRule` from its main entry point. Previously, consumers had to rely on fragile deep imports from `src/types` to type against `compileCSS()` and `getCSSRules()`. Making these types public aligns the package with standard WordPress publishing patterns, improves developer experience for TypeScript projects, and eliminates non-standard deep-path imports.

## Impact

- **Plugin & theme developers**: No code changes or configuration updates required. Existing implementations continue to work unchanged.
- **Package consumers**: Deep imports from `@wordpress/style-engine/src/types` are now redundant and effectively deprecated in favor of importing directly from `@wordpress/style-engine`. Type resolution will succeed natively at the package root level.
- **No breaking changes**: The diff is strictly additive. Existing runtime behavior, REST schema, and CLI/tooling expectations remain untouched.

## Technical details

- **`packages/style-engine/src/index.ts`**:
  - Added three documented `export type` statements that re-export `Style`, `StyleOptions`, and `GeneratedCSSRule` from `./types`.
  - Added JSDoc blocks with usage examples to the existing `compileCSS()` and `getCSSRules()` function signatures.
- **`packages/style-engine/src/types.ts`**:
  - Enriched the existing `Style`, `StyleOptions`, and `GeneratedCSSRule` interfaces with descriptive JSDoc comments. No structural changes to the interfaces themselves.
- **`packages/style-engine/README.md`**:
  - Documented the newly exported types and added concrete usage examples for both functions.
- **Before/After Usage Pattern**:
  ```ts
  // Before (fragile deep import)
  import type { Style } from '@wordpress/style-engine/src/types';

  // After (official public API)
  import type { Style, StyleOptions, GeneratedCSSRule } from '@wordpress/style-engine';
  ```
- The TypeScript build process (`packages/style-engine/build-types/index.d.ts`) will now expose these types at the root level, matching the pattern used by `@wordpress/rich-text` and `@wordpress/components`.

## Contribution

Opened and merged as an `[Enhancement]` for the `[Package] Style Engine` by @ralucaStan (co-authored by @senadir). The discussion was brief, focusing on API consistency rather than feature design. The change directly addresses a reported gap where the package shipped a `types` field but only exported runtime functions from `src/index.ts`, forcing consumers to bypass the published entry point. No alternative implementations or rejected approaches were recorded in the review thread.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
