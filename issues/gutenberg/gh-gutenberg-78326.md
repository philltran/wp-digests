# #78326: Fix inconsistencies in feature selector processing part 2: pseudo block instances

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @tellthemachines
- **Labels:** `[Type] Bug`, `[Package] Block editor`
- **Merged:** [`6fee93b`](https://github.com/WordPress/gutenberg/commit/6fee93b8e4381051cc66e492f7e94b03596ec481)
- **Discussion:** [#78326](https://github.com/WordPress/gutenberg/pull/78326) · 6 comments · 0 reactions

## Summary

Resolves a bug in the block editor where CSS applied to pseudo-states of the Button block was incorrectly targeting the inner element instead of the wrapper. This ensures that layout and typography styles intended for pseudo-state variants are correctly scoped, aligning editor rendering with expected front-end behavior.

## Impact

- **Theme & Block Plugin Developers**: Styles applied to Button block pseudo-states will now correctly target the wrapper element rather than the inner content element, preventing specificity or layout mismatches.
- **Editors**: Applying `width` or `text-orientation` styles to button pseudo-states in the inspector will produce accurate visual feedback in both editor and front-end contexts.
- **No action required** for existing sites; this is a corrective bug fix with no breaking changes or migration steps.

## Technical details

The change adjusts the feature selector processing logic responsible for generating CSS rules applied to block instance pseudo states. Previously, style declarations for properties like `width` and `text-orientation` targeting a button's pseudo-state were erroneously nested inside the inner element's selector context. The updated pipeline ensures these declarations are correctly attached to the Button wrapper element when a pseudo-state variant is active, fixing the scoping mismatch in the generated stylesheet.

## Contribution

Opened by @tellthemachines as a follow-up to #76491 and complementary to #78276, this PR addresses an inconsistency in how feature selectors were processed for pseudo-state styling. During review, @talldan provided direct corrections via commit, leading to a co-authored merge at `6fee93b`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
