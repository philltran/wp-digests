# #83094: Enable setting position within viewport states

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @tellthemachines
- **Labels:** `[Type] Enhancement`, `[Package] Block editor`, `[Feature] Style States`
- **Merged:** [`22631ce`](https://github.com/WordPress/gutenberg/commit/22631cea16c255855594abdad6688b4cdf1f51ea)
- **Discussion:** [#83094](https://github.com/WordPress/gutenberg/pull/83094) · 3 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `position` block support (used by the Group block for sticky/fixed positioning) now accepts per-viewport values, so a block can be sticky on desktop but static on mobile, or vice versa. The PHP renderer in `lib/block-supports/position.php` was refactored to iterate over responsive media queries from `theme.json` viewport settings, merge each viewport's position config with the base (inheritance), and emit rules wrapped in the appropriate `@media` query. The editor-side `PositionPanelPure` and `PositionControlsPanel` components now read and write position values scoped to the currently selected viewport style state.

## Impact

- **Site owners / content editors:** Group blocks (the only core block with `position` support) can now be configured per viewport breakpoint in the block inspector. No code changes required; the existing Position panel simply gains viewport awareness when responsive styles are enabled.
- **Plugin & theme developers:** If you register a custom block with `"position": true` in `block.json`, the PHP renderer will now also look for `@{breakpoint}` keys inside the block's `style` attribute and emit responsive CSS. No API change, but the shape of the `style` attribute a block can carry is broader. The new internal helper `gutenberg_get_position_support_styles()` is not a public API.
- **No breaking changes.** Existing single-viewport position styles render identically. The `is-position-{type}` wrapper class is now added for every viewport that sets a position type, not just the default one, so CSS targeting `.is-position-sticky` will match blocks that are sticky at *any* breakpoint.
- **No action required** for existing sites or plugins.

## Technical details

**PHP (`lib/block-supports/position.php`):**

- New function `gutenberg_get_position_support_styles( $selector, $position, $allowed_position_types )` extracts the per-side CSS generation (including the `calc()` admin-bar offset for `top`) into a reusable unit.
- `gutenberg_render_position_support()` now:
  1. Detects position in either `$style_attribute['position']` (default) or any `@`-prefixed key (viewport state).
  2. Calls `WP_Theme_JSON_Gutenberg::get_viewport_media_queries( $viewport_settings )` to get the breakpoint → media-query map.
  3. For each breakpoint, merges the viewport position over the base via `array_replace( $base_position, $viewport_position_style )`, generates rules, and tags each rule with `'rules_group' => $media_query` so the global styles engine wraps it in the correct `@media`.
  4. If a viewport state explicitly clears the inherited position type, it emits `position: static` in that media query.
  5. Always adds the unique `wp-container-{id}` class to the wrapper; `is-position-{type}` classes are deduplicated via `array_unique`.

**JS (`packages/block-editor/src/hooks/position.jsx`):**

- New exported `getResponsivePositionCSS( { selector, style, viewportSettings } )` iterates `getResponsiveMediaQueries( viewportSettings )` (unlocked from `@wordpress/global-styles-engine`), merges each viewport's position over the base, and returns a CSS string with each rule wrapped in its media query. Emits `position: static` when a viewport clears the inherited type.
- New internal `getPositionTypes( style, viewportSettings )` collects the set of position types across all states (used for class generation).
- `PositionPanelPure` now calls `getSelectedBlockStyleState( clientId )` (unlocked) to determine the active viewport. When in a viewport state, it reads the effective type as `stateStyle?.position?.type ?? style?.position?.type` (inheritance) and writes via `setStyleForState( style, selectedState, newStateStyle )`.

**JS (`packages/block-editor/src/components/inspector-controls-tabs/position-controls-panel.jsx`):**

- `hasAnyPositionValue( style )` now checks both `style.position.type` and any `@`-prefixed key for a position type.
- The "clear" handler writes `position: { type: '', top: undefined, … }` into the specific viewport key rather than the top-level `style.position`.

**JS (`packages/block-editor/src/components/block-inspector/index.jsx`):**

- `<PositionControls />` is rendered inside the `isViewportStyleState` branch of `StyleStateInspectorSlots`, making the panel visible when a viewport state is selected.

**Attribute shape (before → after):**

```jsonc
// Before – single viewport only
{ "style": { "position": { "type": "sticky", "top": "0px" } } }

// After – per-viewport
{
  "style": {
    "position": { "type": "sticky", "top": "0px" },
    "@mobile": { "position": { "type": "" } }
  }
}
```

The `@mobile` entry inherits `top: "0px"` from the base but overrides `type` to empty, which the renderer interprets as `position: static` at that breakpoint.

## Contribution

Opened by @tellthemachines as part of the broader style-states effort (#80388). @ramonjd is credited as co-author. The PR used AI tooling (Copilot, Kimi 3.0, Sonnet 5) with human review, per the author's disclosure. A backport to `wordpress-develop` was tracked as PR #13626 (changelog entry added under `backport-changelog/7.2/`). The discussion is minimal—3 comments, 0 reactions—with no notable design debate visible in the record.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
