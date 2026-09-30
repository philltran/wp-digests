# #81086: URL: Stop normalizePath throwing on a malformed percent sequence

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @konnen916
- **Labels:** `[Type] Bug`, `[Package] Url`, `First-time Contributor`
- **Merged:** [`51d339f`](https://github.com/WordPress/gutenberg/commit/51d339f0d0bf3cb8050dad1e2db86a02148c0c12)
- **Discussion:** [#81086](https://github.com/WordPress/gutenberg/pull/81086) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

`normalizePath()` in `@wordpress/url` no longer throws a `URIError` when a query parameter contains a malformed percent sequence such as a lone `%` (e.g. `/wp/v2/posts?search=50%off`). It now decodes each parameter with the package's existing `safeDecodeURIComponent`, which returns the component unchanged if decoding fails. Well-formed input behaves as before, and ordering stability is preserved.

## Impact

- **Plugin & theme developers / block authors:** Calls to `wp.url.normalizePath()` with an unencoded `%` in a query value now return a normalized path instead of throwing. No code changes required.
- **`@wordpress/api-fetch` consumers:** The preloading middleware calls `normalizePath` on every preloaded data key and on `options.path` for each request. Previously, one malformed `%` in a hand-written path or a server-generated preload key would throw inside the middleware. It now normalizes instead.
- **Site owners / hosting:** No action required.
- **Behavior note:** Malformed sequences are re-encoded in the output (`50%off` becomes `50%25off`), so the returned path differs from the input for such values.

## Technical details

The change is one line in `packages/url/src/normalize-path.ts`: the per-pair mapping switches from bare `decodeURIComponent` to `safeDecodeURIComponent`, imported from `./safe-decode-uri-component`.

```diff
-	.map( ( pair ) => pair.map( decodeURIComponent ) )
+	.map( ( pair ) => pair.map( safeDecodeURIComponent ) )
```

`safeDecodeURIComponent` is the helper introduced in #45561 for `getQueryArgs`; `normalizePath` was missed at the time. Because the failed decode leaves the raw string, the later re-encoding step yields `%25` for the stray `%`.

Two tests were added to `packages/url/src/test/index.js`:
- `/foo/bar?baz=%E0%A4%A` does not throw and normalizes to `/foo/bar?baz=%25E0%25A4%25A`.
- `/foo/bar?a=5&b=50%off` and `/foo/bar?b=50%off&a=5` normalize to the same string.

A `packages/url/CHANGELOG.md` entry was also added. No hooks, REST schema, or public signatures change.

## Contribution

Opened by first-time contributor @konnen916, closing issue #81085. Review feedback led to removing a dependency-group comment block above the import (an ESLint failure, since sibling files such as `get-query-args.ts` don't use one), adopting suggested CHANGELOG wording, and a rebase after the 4.53.0 changelog section was cut. The approach mirrors #45561 and no alternatives were discussed.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
