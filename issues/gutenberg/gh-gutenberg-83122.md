# #83122: Post Navigation Link: Add border and spacing support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Post Navigation Link`
- **Merged:** [`6e5cabb`](https://github.com/WordPress/gutenberg/commit/6e5cabb8dd131e35b14a8de78d3c3cb805535026)
- **Discussion:** [#83122](https://github.com/WordPress/gutenberg/pull/83122) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Adds border (color, radius, style, width) and spacing (margin, padding) support to the `core/post-navigation-link` block. Because the block always renders its wrapper `<div>` even when no adjacent post exists, both supports use `__experimentalSkipSerialization` and the styles are applied only when a link actually renders, avoiding a visible box around an empty element. This is a companion to PR #83058 (shadow support) and supersedes the earlier #64258 attempt.

## Impact

- **Theme & block developers:** `core/post-navigation-link` now accepts `border` and `spacing` (margin, padding) in its `style` attribute. All sub-controls are hidden by default (`__experimentalDefaultControls` set to `false`), so existing blocks are unaffected.
- **Global Styles:** New border and spacing controls appear under Blocks → Post Navigation Link. The generated CSS is scoped to `.wp-block-post-navigation-link:not(:empty)`, so empty wrappers (first/last post) receive no styles.
- **No action required** for existing sites. No breaking changes, no removed APIs, no migration needed.
- **Editor vs. front-end divergence:** The editor always renders placeholder text, so border/spacing/shadow styles are always visible in the editor even on the first or last post where the front end shows nothing.

## Technical details

**`block.json`** — Two new support entries, both with `__experimentalSkipSerialization: true`:

```json
"spacing": {
  "__experimentalSkipSerialization": true,
  "margin": true,
  "padding": true,
  "__experimentalDefaultControls": { "margin": false, "padding": false }
},
"__experimentalBorder": {
  "__experimentalSkipSerialization": true,
  "color": true, "radius": true, "style": true, "width": true,
  "__experimentalDefaultControls": { "color": false, "radius": false, "style": false, "width": false }
}
```

The `selectors` map gains `border` and `spacing` entries alongside the existing `shadow`, all pointing to `.wp-block-post-navigation-link:not(:empty)`.

**`index.php`** — A new function `block_core_post_navigation_link_get_support_styles( $attributes )` (marked `@since 7.2.0`) builds border, shadow, and spacing styles in one call to `wp_style_engine_get_styles()`. It handles legacy unitless border radius/width values by appending `px`, resolves preset border colors from the `borderColor` attribute into `var:preset|color|…` tokens, and iterates individual sides (`top`, `right`, `bottom`, `left`). The render callback now casts `$content` to string (to handle `{next,previous}_post_link` filters returning `null`) and calls the helper only when the cast content is non-empty:

```php
$content = (string) $content;

$support_styles = '' === $content
    ? array( 'class' => '', 'style' => '' )
    : block_core_post_navigation_link_get_support_styles( $attributes );
```

The previously inline shadow-only logic is replaced by this unified helper.

**`edit.jsx`** — Imports `__experimentalUseBorderProps as useBorderProps` and `__experimentalGetSpacingClassesAndStyles as getSpacingClassesAndStyles`. Merges `borderProps.className`, `borderProps.style`, `shadowProps.style`, and `spacingProps.style` into `useBlockProps`, so preset border colors retain their `has-*-border-color` class in the editor.

**`style.scss`** — Adds `box-sizing: border-box` to `.wp-block-post-navigation-link` so that padding and border do not expand the element beyond its content width.

**Tests** — New file `phpunit/blocks/render-block-post-navigation-link-test.php` (`Tests_Blocks_Render_Post_Navigation_Link`) with six methods covering: empty wrapper omits styles, empty wrapper keeps normally-serialized supports (text-align, background), rendered link includes all three supports, individual border sides match core declaration order, unitless border values get `px` appended, and a `previous_post_link` filter returning `null` omits styles.

## Contribution

Opened by @aaronrobertshaw with co-authorship from @ramonjd. The PR supersedes #64258, which was closed after removing the wrapper was rejected as a backward-compatibility risk. It is a companion to #83058 (shadow support) and part of the broader design-tools tracking issues #43241, #43243, and #43247. The author noted a rebase after #83058 merged, which folded the shadow handling into the shared helper and reordered the individual border sides to match core block-supports order. The PR was implemented, tested, and screenshotted by a Claude Code agent. Three comments, no reactions.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
