# #83058: Post Navigation Link: Add shadow support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Post Navigation Link`
- **Merged:** [`043c9a4`](https://github.com/WordPress/gutenberg/commit/043c9a4afd9b2cecca93bb68f7a36cd68ce0729e)
- **Discussion:** [#83058](https://github.com/WordPress/gutenberg/pull/83058) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Shadow support is added to the `core/post-navigation-link` block. Because the block always renders its wrapper `<div>` even when no adjacent post exists, a naïve shadow would paint as a coloured bar on the zero-height empty wrapper. The fix skips normal style-engine serialization for shadow and instead applies the shadow inline only when a link was actually rendered, while Global Styles are scoped with a `:not(:empty)` selector.

## Impact

- **Theme & block developers:** The block now accepts a `shadow` attribute and responds to Global Styles shadow settings. No code changes are required to use it; set the shadow in the block inspector or under Blocks → Post Navigation Link in Global Styles.
- **Block developers (pattern reference):** This PR introduces a new use of `__experimentalSkipSerialization` on a block support to keep styles on the same element while conditionally withholding them. It also uses the `selectors` property in `block.json` to scope a Global Styles rule. Both patterns may be relevant if you build blocks that conditionally render empty wrappers.
- **Site owners / hosting:** No action required. Existing sites with a Post Navigation Link block will simply gain the ability to apply a shadow; nothing changes unless a shadow is explicitly set.
- **Editor vs. front-end discrepancy:** The editor always renders placeholder text ("Next" / "Previous"), so the shadow is always visible in the editor. On the front end, the first post (no previous) or last post (no next) will show no shadow because the wrapper is empty. This is intentional and documented in the PR.

## Technical details

**`block.json`** — two additions:

```json
"supports": {
  "shadow": {
    "__experimentalSkipSerialization": true
  }
},
"selectors": {
  "shadow": ".wp-block-post-navigation-link:not(:empty)"
}
```

`__experimentalSkipSerialization` tells the style engine not to emit the shadow CSS into the wrapper's `style` attribute during normal serialization. The `selectors` entry scopes the Global Styles shadow rule so it only matches non-empty wrappers.

**`index.php`** (`render_block_core_post_navigation_link`) — the `get_block_wrapper_attributes()` call is moved from the top of the function to after `$content` is resolved. Shadow styles are then applied manually and only when content exists:

```php
$styles = '';
if ( '' !== $content && ! empty( $attributes['style']['shadow'] ) ) {
    $shadow_styles = wp_style_engine_get_styles( array( 'shadow' => $attributes['style']['shadow'] ) );
    $styles        = $shadow_styles['css'] ?? '';
}

$wrapper_attributes = get_block_wrapper_attributes(
    array(
        'class' => $classes,
        'style' => $styles,
    )
);
```

**`edit.jsx`** — imports `__experimentalGetShadowClassesAndStyles` from `@wordpress/block-editor` and passes its `style` output into `useBlockProps`:

```js
const shadowProps = getShadowClassesAndStyles( attributes );
const blockProps = useBlockProps( { style: shadowProps.style } );
```

The editor always shows the shadow because it always renders placeholder text, so the wrapper is never empty in the editor context.

**Docs** — `block.json` support list and the block's `README.md` are updated to list `shadow` and document the new `selectors` entry.

## Contribution

Opened by @aaronrobertshaw as a companion to #83122 (border and spacing for the same block) and part of the broader #43241 shadow-support effort. The author noted in review that they "had to rework the approach" to skip serialization after discovering the empty-wrapper shadow-bar problem; the earlier iteration presumably applied shadow through the standard style-engine path. Removing the wrapper entirely was considered and rejected in #64258, which is why the conditional-withholding approach was taken instead. @shail-mehta provided a full local test pass covering Global Styles, block-level override, empty wrappers, and responsive "Unset" shadow. Co-authored with @im3dabasia and @shail-mehta. The PR notes it was "implemented, built and screenshotted by a Claude Code agent."

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
