# #82278: Global styles engine: return a copy from getResolvedValue instead of mutating

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Bug`, `Global Styles`
- **Merged:** [`1a134f1`](https://github.com/WordPress/gutenberg/commit/1a134f12130a92ad2bd6bb687b723c6af4829b01)
- **Discussion:** [#82278](https://github.com/WordPress/gutenberg/pull/82278) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

`getResolvedValue` in `@wordpress/global-styles-engine` no longer overwrites `url` on the object it resolved when converting a theme-relative `file:./…` URL to an absolute one. It now returns a shallow copy with the resolved `url`. This prevents a portable theme-relative pointer from being replaced by a hardcoded site URL in shared config data that could later be saved.

## Impact

- **Plugin & theme developers using `@wordpress/global-styles-engine` directly:** If you call `getResolvedValue` and relied on the input object being mutated, switch to using the return value. The PR states all in-repo callers already use the return value.
- **Site owners / theme authors:** Fixes a path where a `file:./…` `backgroundImage` URL could be rewritten to an absolute URL in user or theme config objects that are later persisted. Themes keep portable references.
- **Everyone else:** No action required.

## Technical details

In `packages/global-styles-engine/src/utils/common.ts`, the branch of `getResolvedValue` that handles an object with a truthy `url` previously assigned `resolvedValue.url = getResolvedThemeFilePath(resolvedValue.url, tree?._links?.['wp:theme-file'])` and then returned `resolvedValue`. It now returns a new object instead:

```ts
// before
resolvedValue.url = getResolvedThemeFilePath( resolvedValue.url, tree?._links?.[ 'wp:theme-file' ] );
return resolvedValue;

// after
return {
	...resolvedValue,
	url: getResolvedThemeFilePath( resolvedValue.url, tree?._links?.[ 'wp:theme-file' ] ),
};
```

The aliasing arises because `resolvedValue` is either the caller's own value or, after a `ref` is resolved, the tree's object. `mergeGlobalStyles` keeps `backgroundImage` leaves atomic (`userConfig ?? baseConfig`), so the merged tree can point at the user or theme config object. Two unit tests were added in `src/test/utils.test.ts`: a plain `{ url }` input remains `file:./assets/image.jpg` after the call, and resolving a `ref` to `styles.background.backgroundImage` leaves the tree's value unchanged while returning the absolute URL. The PR notes the existing `getResolvedRefValue` tests only passed in their prior order because the mutation altered a shared fixture. A `CHANGELOG.md` entry was added under Unreleased / Bug Fixes. No hooks, REST schema, or DB changes.

## Contribution

Opened by @ramonjd and reviewed quickly by @Mamaduka, who is credited in the co-author trailer. The record shows no design debate or alternative approaches. The only CI noise was an unrelated flaky e2e test (`pages.spec.js`).

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
