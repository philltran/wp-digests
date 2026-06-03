# #70615: Fix: properly merged schema during `mergeSchemas` part in `getBlockContentSchemaFromTransforms` 

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @USERSATOSHI
- **Labels:** `[Type] Bug`, `[Package] Blocks`, `Good First Review`, `[Package] Block editor`
- **Merged:** [`5d21736`](https://github.com/WordPress/gutenberg/commit/5d217364fe271c9274c256e4c943451631c48bee)
- **Discussion:** [#70615](https://github.com/WordPress/gutenberg/pull/70615) · 15 comments · 1 reactions

## Summary

Fixes a regression in `getBlockContentSchemaFromTransforms` where merging block content schemas incorrectly converted array properties like `classes` into objects keyed by numeric indices. This type coercion caused downstream parsing routines, specifically `cleanNodeList`, to fail when mapping expected array values, breaking block transforms that rely on schema-based class validation during raw content handling.

## Impact

- **Plugin & theme developers**: No action required. The fix restores the expected array structure for `classes` during schema merging, preventing runtime parser errors in block transform pipelines.
- **Headless/Block Editor consumers**: Restores correct behavior for raw HTML output containing class attributes; no configuration changes needed.
- **Hosting & platform teams**: No action required. This is an internal update to the `@wordpress/blocks` package's raw-handling logic.

## Technical details

The change modifies `packages/blocks/src/api/raw-handling/utils.ts`, specifically updating the `mergeSchemas` and `mergeTagNameSchemas` helper functions used by `getBlockContentSchemaFromTransforms`. Previously, merging a schema property with `{ ...b[ key ] }` coerced arrays into objects (e.g., `[ 'a' ]` became `{ 0: 'a' }`). The updated logic now explicitly checks `Array.isArray( b[ key ] )`, assigning a shallow copy via `.slice()` instead of object spread. A dedicated `case 'classes':` branch was added to handle wildcard tokens (`'*'`) by short-circuiting to `['*']` if either side contains it, otherwise concatenating both arrays and preserving them as arrays. Test coverage was expanded in `packages/blocks/src/api/raw-handling/test/utils.js` to verify array preservation across single/missing transform combinations and duplicate class merging.

**Before:**
```typescript
// In mergeTagNameSchemas / mergeSchemas
for ( const key in b ) {
	a[ key ] = a[ key ] ? mergeTagNameSchemaProperties( a[ key ], b[ key ], key ) : { ...b[ key ] };
}
```
**After:**
```typescript
for ( const key in b ) {
	if ( a[ key ] ) {
		a[ key ] = mergeTagNameSchemaProperties( a[ key ], b[ key ], key );
	} else if ( Array.isArray( b[ key ] ) ) {
		a[ key ] = b[ key ].slice();
	} else {
		a[ key ] = { ...b[ key ] };
	}
}
```

## Contribution

Opened and merged as PR #70615 by @USERSATOSHI, with co-authors @sirreal, @Adi-ty, @aaronjorbin, @desrosj, @getdave, and @SirLouen. The author initially encountered a branch rebase conflict but quickly resolved it; lead reviewers requested explicit unit test coverage for the regression, which was added to `utils.js`. Merged into Gutenberg trunk at commit `5d21736`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
