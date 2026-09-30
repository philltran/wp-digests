# #82335: Fix: Restore layout styles for block style variations

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Jiwoon-Kim
- **Labels:** `[Type] Bug`, `First-time Contributor`, `[Feature] Layout`, `[Feature] Block Style Variations`, `Backported to WP Core`
- **Merged:** [`8477719`](https://github.com/WordPress/gutenberg/commit/8477719f8ede78311bb983ea9d406f991e6f4249)
- **Discussion:** [#82335](https://github.com/WordPress/gutenberg/pull/82335) · 9 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Block style variations that declare `spacing.blockGap` stopped emitting their gap CSS in WordPress 7.1 because a `name` field added to variation nodes by #78276 was misread by `get_layout_styles()` as a block name, causing the layout-support check to fail silently. This PR restores the 7.0 behaviour by substituting the owning block's name into the metadata copy passed to `get_layout_styles()` at both the base and responsive call sites in `get_styles_for_block()`. The fix is backported to the 7.1.1 minor release.

## Impact

- **Theme & plugin developers using block style variations with `spacing.blockGap`:** Gap styles (including responsive variants inside viewport breakpoints) will render again once the 7.1.1 backport lands. No code changes required — the feature simply works as it did in 7.0.
- **Site owners:** No action required. Front-end gap spacing on styled block variations will be restored automatically with the update.
- **No breaking changes, deprecations, or new APIs.** The fix is internal to `WP_Theme_JSON_Gutenberg::get_styles_for_block()` and does not alter the public block style variation schema or the `block_has_support()` contract.

## Technical details

In `lib/class-wp-theme-json-gutenberg.php`, `get_styles_for_block()` builds a metadata array for each style variation node and passes it to `get_layout_styles()`. Since #78276, `get_block_nodes()` attaches `'name' => $variation_slug` to each variation node. `get_layout_styles()` reads `name` as a block name and calls `block_has_support( $block_name, 'layout' )`; a variation slug is never a registered block, so the check returns `false` and the function returns an empty string, discarding all layout rules for that variation.

The fix overrides `name` on a **copy** of the variation metadata before each `get_layout_styles()` call, using `$block_name` (already resolved above for the pseudo-selector logic, with a fallback that derives it from the metadata path). Two call sites are patched:

1. **Base `spacing.blockGap`** (~line 3882): after building `$variation_metadata_with_selector`, the new line `$variation_metadata_with_selector['name'] = $block_name;` is added before the metadata is stored in `$style_variation_layout_metadata`.
2. **Responsive breakpoint `spacing.blockGap`** (~line 3949): the same override is applied to `$variation_layout_metadata` before the `get_layout_styles()` call inside the `@mobile` / viewport loop.

The original variation node retains its slug, so the feature-selector processing introduced by #78276 is unaffected. `get_layout_styles()` itself is unchanged — its guard is correct for real block nodes; the fix simply ensures it receives a real block name.

Three new tests in `phpunit/class-wp-theme-json-test.php`:

- `test_block_style_variation_with_block_gap_emits_layout_styles` — registers a `core/group` variation with `blockGap: '3em'`, asserts the stylesheet contains `:root :where(.wp-block-group.is-style-custom-group.wp-block-group-is-layout-flex){gap: 3em;}`.
- `test_block_style_variation_with_responsive_block_gap_emits_layout_styles` — adds a `@mobile` breakpoint with `blockGap: '1em'`, asserts both the `@media (width <= 599px)` wrapper and the gap rule inside it.
- `test_block_style_variation_block_gap_respects_layout_support` — registers a `core/paragraph` variation (a block without layout support) and asserts the stylesheet does **not** contain the variation class, confirming the support check is answered rather than skipped.

The first two tests fail on trunk without the fix.

## Contribution

Opened by first-time contributor @Jiwoon-Kim. @t-hamano reviewed and cc'd @tellthemachines, @ramonjd, and @talldan. @ramonjd created Trac ticket #66044 for the Core backport; @Jiwoon-Kim then opened the corresponding `wordpress-develop` sync PR (#13398) and added the `backport-changelog/7.1/13398.md` entry. @adamsilverstein flagged the 7.1.1 RC deadline (September 10) to expedite review. The author explicitly scoped the PR as a minimal restoration of 7.0 behaviour, noting in review that whether a variation node's `name` should be passed through the general layout path at all is a broader design question deliberately left out of this regression fix.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
