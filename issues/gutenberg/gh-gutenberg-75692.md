# #75692: Navigation Overlay Close: Set Close as default text, rather than using a placeholder

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @scruffian
- **Labels:** `[Type] Bug`, `[Package] Block library`, `Backported to WP Core`
- **Merged:** [`e90f98c`](https://github.com/WordPress/gutenberg/commit/e90f98c993abf73ada30c5d8fdc975da3d2044be)
- **Discussion:** [#75692](https://github.com/WordPress/gutenberg/pull/75692) · 11 comments · 0 reactions

## Summary

The Navigation Overlay Close button block (`core/navigation-overlay-close`) previously rendered its "Close" label as a `RichText` placeholder rather than as real content. This PR removes the placeholder and instead seeds the block's text with a default value of "Close", and drops the `withoutInteractiveFormatting` flag on the `RichText`. The result is that the close button's text now renders identically in the editor and on the frontend (it was previously shown in grey placeholder styling in the editor), and inline formatting controls now apply to the label as expected.

## Impact

- **Site owners / editors:** The overlay close button label now appears in normal text color in the editor (matching the frontend) and can be formatted with the standard inline rich-text controls. No action required for existing navigation blocks.
- **Theme developers:** If you relied on the close button having empty/placeholder text by default, note the block now ships an actual default value of "Close" as content. Behavior is otherwise unchanged.
- **No breaking API changes.** This is a bug fix to the block's editor behavior; the build size impact is +5 B.

## Technical details

The change targets the `navigation-overlay-close` block in `packages/block-library`. Three coordinated edits:

- The `placeholder` prop is removed from the block's `RichText` (the editor no longer shows greyed-out placeholder text).
- A default value of "Close" is provided for the block's text. Per the discussion, the default string was placed in `edit.js` (and the corresponding PHP render path) rather than hardcoded in `block.json`, specifically so it remains translatable via the i18n functions.
- The `withoutInteractiveFormatting` flag is removed from the `RichText`, which re-enables the interactive inline formatting controls for the label.

Conceptually:

```jsx
// Before
<RichText
  withoutInteractiveFormatting
  placeholder={ __( 'Close' ) }
  value={ label }
  …
/>

// After
<RichText
  value={ label } // defaults to __( 'Close' )
  …
/>
```

Because the label is now stored as real block content (defaulted in `edit.js`/PHP) instead of a placeholder, the editor and frontend share the same rendered markup and color, and formatting is preserved.

## Contribution

Opened and merged by **@scruffian** (merge commit `e90f98c`), with **@mikachan** as co-author. The notable design point in the discussion was localization: an initial approach risked putting the "Close" string in `block.json` (mirroring the existing pattern used by the `form-input` block), which would not be translatable. @mikachan flagged this, and @scruffian moved the default text into `edit.js` so it could be run through the i18n functions. The PR is labeled `[Type] Bug`, `[Package] Block library`, and has been backported to WordPress Core.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
