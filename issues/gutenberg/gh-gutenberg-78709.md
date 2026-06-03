# #78709: Unset grid span defaults with viewport states enabled

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @tellthemachines
- **Labels:** `[Type] Bug`, `[Package] Block editor`, `[Feature] Layout`
- **Merged:** [`56dc564`](https://github.com/WordPress/gutenberg/commit/56dc5644daa47d99773bde4e38d3b49c54a39e80)
- **Discussion:** [#78709](https://github.com/WordPress/gutenberg/pull/78709) · 4 comments · 0 reactions

## Summary

When a grid child block had a column or row span set, switching to a non-default viewport state (e.g., tablet or mobile) caused the span controls in `child-layout-control` to display `1` — the numeric input's fallback default — even though no span value had been saved for that viewport. Because the control showed `1`, the editor treated the field as already set to `1`, making it impossible to explicitly save a span of `1` without first entering a different value. The fix removes those phantom defaults from the span inputs when viewport states are enabled, so the controls render empty (no saved value) rather than misleadingly populated.

## Impact

**Plugin & theme developers**
- No breaking changes or API removals.
- Teams using the Grid block with viewport-responsive span values in the block editor will see corrected behavior: span controls are now blank when no viewport-specific value exists, rather than showing a false `1`.
- No migration or code changes required — this is a pure editor UX bug fix.

**Site owners / content editors**
- Existing saved span values are unaffected on the front end.
- In the editor, span controls will appear empty when switching to a non-default viewport state with no span set, allowing a value of `1` to be entered and saved directly.

**Hosting & platform / headless & REST consumers**
- No action required. No REST schema, serialization format, or block attribute structure changed.

## Technical details

The change is isolated to `packages/block-editor/src/components/child-layout-control/index.js`, reflected in the compiled `build/scripts/block-editor/index.min.js` (+34 B).

The `ChildLayoutControl` component renders `NumberControl` (or equivalent) inputs for `columnSpan` and `rowSpan`. Previously, these inputs supplied a hard-coded default of `1` regardless of whether a value existed for the active viewport state. When viewport states are enabled and the active state has no saved span, the component now passes `undefined` (or omits the default) instead of `1`, so the control renders empty.

The condition gates on whether a non-default viewport state is active — the default state retains normal behavior. The PR is a follow-up to #78543, which introduced viewport state support for grid span controls.

## Contribution

Opened and authored by @tellthemachines as a follow-up to #78543. @andrewserong was credited as co-author in the merge commit. The PR attracted no substantive design debate in the four-comment thread. @cbravobernal cherry-picked the fix to the `release/23.3` branch (commit `def6279b848`) for inclusion in the next release.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
