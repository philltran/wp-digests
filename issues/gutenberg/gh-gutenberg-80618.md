# #80618: createInterpolateElement: handle unmatched closing tags gracefully

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Kgupta62
- **Labels:** `[Type] Bug`, `[Package] Element`
- **Merged:** [`65951dd`](https://github.com/WordPress/gutenberg/commit/65951ddf6afd2d41ae679f2c440e05d6ac156f90)
- **Discussion:** [#80618](https://github.com/WordPress/gutenberg/pull/80618) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

`createInterpolateElement()` in `@wordpress/element` now handles strings containing unmatched closing tags without throwing a fatal `TypeError`. Previously, encountering a closing tag with no matching opener on the internal stack caused a crash in the editor. The function now gracefully stops interpolation at the first unmatched closer, returns the remaining text as a plain string, and logs a development warning.

## Impact

- **Plugin & theme developers**: No breaking changes. Code that previously crashed on malformed interpolation strings will now render safely with a `console.warn` in `SCRIPT_DEBUG` mode.
- **Editor users**: Prevents critical editor crashes when blocks or widgets contain mismatched closing tags in dynamic content.
- **Action required**: None. Existing valid usage is unaffected. Developers should audit dynamic content sources for mismatched tags to eliminate the new warning.

## Technical details

The fix targets `packages/element/src/create-interpolate-element.ts`. In the `'closer'` parsing branch, a guard checks `if ( 0 === stackDepth )`. When triggered, it conditionally logs `console.warn( \`Unmatched closing tag '</${ name }>' in createInterpolateElement. The rest of the string was not interpolated.\` )` (only when `globalThis.SCRIPT_DEBUG` is true), calls `addText()` to flush any previously interpolated content, and returns `false` to halt further parsing. This ensures tokens matched before the invalid closer are still rendered, while the remainder is returned as unprocessed text. A regression test in `packages/element/src/test/create-interpolate-element.tsx` verifies both the warning output and the partial interpolation behavior.

**Before** (throws `TypeError`):
```js
createInterpolateElement( 'a </item> b', { item: <em /> } )
```

**After** (returns plain text + warning):
```js
// Returns <>{ 'a </item> b' }</> and logs a console.warn in SCRIPT_DEBUG
createInterpolateElement( 'a </item> b', { item: <em /> } )
```

## Contribution

Opened by @Kgupta62 to resolve #60843, the PR initially proposed silently ignoring the invalid tag. Reviewer @Mamaduka requested a developer-facing warning instead, which was added in a follow-up commit. @manzoorwanijk and @Mamaduka refined the guard placement and test coverage before merge. The author noted that two prior PRs on the same issue (#65294, #71413) were superseded due to targeting outdated JS files or including unrelated repository changes.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
