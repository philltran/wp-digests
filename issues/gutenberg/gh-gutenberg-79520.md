# #79520: Fix unsetting values in viewport states for grid and constrained layouts

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @tellthemachines
- **Labels:** `[Type] Bug`, `[Package] Block editor`, `[Feature] Layout`, `[Feature] Style States`
- **Merged:** [`5023ce3`](https://github.com/WordPress/gutenberg/commit/5023ce37c1f209f3627d3e8a8db510c12bea83ec)
- **Discussion:** [#79520](https://github.com/WordPress/gutenberg/pull/79520) · 7 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Fixes a bug where clearing layout properties (e.g., grid column count or constrained content width) in specific viewport states was ignored, causing default state values to bleed into smaller viewports. The change introduces explicit `null` overrides for cleared fields and generates corresponding CSS resets in both the PHP rendering layer and JS editor logic.

## Impact

["- **Block & theme developers**: Responsive layout controls in the block editor now correctly respect cleared/unset values per viewport. Previously, omitting a property in a child viewport fallback to the default state's value.", '- **Site owners/editors**: Grid and constrained layout settings can now be properly reset across viewport states without unexpected styling bleeding through from the desktop/default configuration.', '- **Action required**: No migration or code changes needed. Developers using theme.json or custom block patterns should verify that clearing viewport-specific overrides now outputs the expected CSS resets in both editor and frontend.']

## Technical details

The fix operates across two layers to properly serialize explicit unsets:
- **PHP** (`lib/block-supports/layout.php`): Updates `gutenberg_get_layout_style()` to treat `null` values inside `$options['viewport_overrides']` as intentional clears via `array_key_exists()`. It now determines whether to output constrained size resets and injects CSS fallbacks like `var(--wp--style--global--content-size, none)` when a width is cleared.
- **JS** (`packages/block-editor/src/hooks/layout.js`): Exports `getLayoutStateOverrides()` and modifies its logic so that cleared properties are stored as `null` rather than being stripped from the override object. Inspector controls in `constrained.js` and `grid.js` are updated to render `null` values as empty inputs and correctly pass them through `onChange`.
- **Behavior shift**: In viewport overrides, `{ columnCount: undefined }` previously disappeared, falling back to default. It now resolves to `{ columnCount: null }`, triggering the CSS reset path in both editor serialization and frontend `gutenberg_get_layout_style()` output.

## Contribution

Opened by `@tellthemachines` as a follow-up to #78543, the PR addresses how the block editor serializes cleared layout properties across viewport states. Co-authored with `@talldan` and `@ramonjd`. Review focused on verifying test coverage for grid column resets and constrained width overrides, with reviewer notes clarifying inspector control locations and editor/frontend rendering differences. Merged as part of the 7.1 backport track.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
