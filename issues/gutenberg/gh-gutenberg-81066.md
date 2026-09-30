# #81066: Fix: getQueryArgs discards everything after the second = in a query argument value

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @hbhalodia
- **Labels:** `[Type] Bug`, `[Package] Url`
- **Merged:** [`c384568`](https://github.com/WordPress/gutenberg/commit/c384568f215d0c0bac8226094745e6f679d082b6)
- **Discussion:** [#81066](https://github.com/WordPress/gutenberg/pull/81066) · 9 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

`getQueryArgs()` in `@wordpress/url` now splits each `key=value` pair on the first `=` only. Previously it split on every `=` and kept just the first two segments, so a value like `x=y` was silently truncated to `x`. The fix also stops a keyless pair such as `?=orphan` from being parsed as `{ orphan: '' }`; it is now skipped.

## Impact

- **Plugin & theme developers:**
  - Any code that parses URLs with `getQueryArgs()`, `getQueryArg()`, `hasQueryArg()` or `removeQueryArgs()` now gets full values when they contain unencoded `=` (base64 padding, JWTs, nested URLs such as `?redirect=/watch?v=abc`).
  - `addQueryArgs()` merges existing args through `getQueryArgs()`, so appending a param to a URL used to rewrite the truncated value back out. That silent data loss is fixed.
  - `@wordpress/api-fetch`'s default `user-locale` middleware calls `addQueryArgs()`, so requests whose URLs carried `=` in a value were affected and now are not.
- **Behavior change to check:** `?=orphan` previously returned `{ orphan: '' }` and now returns `{}`. Code that relied on that (unlikely) will see a difference.
- **Unchanged:** no signature, type or `QueryArgs` change. `?foo` and `?foo=` still give `{ foo: '' }`, bracket syntax (`foo[]=`, `user[name]=`, and their percent-encoded forms) still works, and malformed percent escapes still return the raw string rather than throwing.
- **Action required:** none, beyond a look at any workarounds you added for the truncation.

## Technical details

The change is in `packages/url/src/get-query-args.ts`. The old reducer body was:

```js
const [ key, value = '' ] = keyValue
	.split( '=' )
	.filter( Boolean )
	.map( safeDecodeURIComponent );
```

`'b=x=y'.split('=')` gives `['b','x','y']`, and the destructuring read only the first two entries. `.filter( Boolean )` also removed the empty key from `['', 'orphan']`, which promoted the value to a key.

The diff replaces this with an `indexOf( '=' )` lookup:

```js
const separatorIndex = keyValue.indexOf( '=' );
const hasValue = separatorIndex !== -1;
const key = safeDecodeURIComponent(
	hasValue ? keyValue.slice( 0, separatorIndex ) : keyValue
);

if ( key ) {
	const value = hasValue
		? safeDecodeURIComponent( keyValue.slice( separatorIndex + 1 ) )
		: '';
	const segments = key.replace( /\]/g, '' ).split( '[' );
	setPath( accumulator, segments, value );
}
```

- Dropping `.filter( Boolean )` lets an empty key reach the existing `if ( key )` guard, so keyless pairs are skipped.
- `value` is now computed inside the guard, and only the key and value substrings are decoded.
- The key is still decoded before being split on `[`, which keeps the encoded bracket forms working.
- Tests added in `packages/url/src/test/index.js`: four `getQueryArgs` cases (multiple `=`, base64 padding, unencoded nested URL, keyless pair) and two `addQueryArgs` round-trip regression tests.
- A `Bug Fixes` entry was added to `packages/url/CHANGELOG.md` under Unreleased.

## Contribution

Reviewers agreed on the `indexOf`/`slice` approach over aduth's alternative of keeping the split and rejoining the remainder (`const [ key, ...valueParts ] = keyValue.split( '=' )`), noting it does less work. He also observed that JavaScript's `split` limit argument discards the remainder rather than keeping it intact, so it could not be used. @konnen916 asked for `addQueryArgs` round-trip tests, since that path is how the bug reaches `api-fetch` requests, and @hbhalodia added them. The PR was approved and merged by @aduth after a merge conflict was resolved. The author disclosed using Claude Code for an implementation draft, which they reviewed.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
