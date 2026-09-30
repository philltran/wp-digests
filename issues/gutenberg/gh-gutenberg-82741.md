# #82741: Public API for supporting viewport states in custom controls

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @tellthemachines
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Package] Block editor`, `Needs Dev Note`, `[Feature] Style States`
- **Merged:** [`387ecd0`](https://github.com/WordPress/gutenberg/commit/387ecd066adfafafeef26ad365d960ef1ffc341c)
- **Discussion:** [#82741](https://github.com/WordPress/gutenberg/pull/82741) · 3 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

This PR promotes four previously private (locked) block-editor APIs to the public surface of `@wordpress/block-editor`: the `getStyleForState` and `setStyleForState` hooks, and the `getSelectedBlockStyleState` and `hasSelectedBlockStyleState` store selectors (the latter renamed from `hasSelectedStyleState`). It also adds a new PHP function `gutenberg_get_viewport_media_queries()` as a thin wrapper around `WP_Theme_JSON_Gutenberg::get_viewport_media_queries`. The change gives plugin and theme authors a stable, documented way to read and write per-viewport and per-pseudo-state style values in custom block controls, rather than reaching into `unlock()`-gated internals.

## Impact

- **Plugin & theme developers (custom block controls):** You can now import `getStyleForState` and `setStyleForState` directly from `@wordpress/block-editor` and call `getSelectedBlockStyleState` / `hasSelectedBlockStyleState` via `useSelect` without `unlock()`. If your code currently accesses these through `unlock( select( blockEditorStore ) )` or `unlock( blockEditorPrivateApis )`, update to the public imports.
- **Breaking (private API removal):** The old private selector `hasSelectedStyleState` is removed from `private-selectors.js`. The private exports of `getStyleForState` and `setStyleForState` are removed from `private-apis.js`. Any code that called these via `unlock()` will break at runtime.
- **PHP / server-side:** New function `gutenberg_get_viewport_media_queries( $viewport_settings, $options )` is available in `lib/global-styles-and-settings.php`. No existing function is renamed or removed.
- **No action required** if you do not build custom block controls that interact with viewport/pseudo style states.

## Technical details

**JS – `@wordpress/block-editor` public exports** (`packages/block-editor/src/index.js`):

```js
// Before (private, required unlock)
import { unlock } from '@wordpress/block-editor';
const { getStyleForState, setStyleForState } = unlock( blockEditorPrivateApis );
const { hasSelectedStyleState } = unlock( select( blockEditorStore ) );

// After (public)
import { getStyleForState, setStyleForState } from '@wordpress/block-editor';
const { getSelectedBlockStyleState, hasSelectedBlockStyleState } = select( blockEditorStore );
```

- `getStyleForState( style, selectedState )` and `setStyleForState( style, selectedState, newStyle )` live in `packages/block-editor/src/hooks/block-style-state.js`. They were already exported from that module but were only reachable through `private-apis.js`; the diff adds a direct re-export in `src/index.js` and removes them from `private-apis.js`.
- `getSelectedBlockStyleState` and `hasSelectedBlockStyleState` are moved verbatim from `packages/block-editor/src/store/private-selectors.js` to `packages/block-editor/src/store/selectors.js`. The old `hasSelectedStyleState` name is gone; the new name is `hasSelectedBlockStyleState`. `DEFAULT_BLOCK_STYLE_STATE` is now exported from `private-selectors.js` (still private) so the public selectors can reference it.
- `packages/block-editor/src/hooks/fit-text.jsx` is updated to call the public `hasSelectedBlockStyleState` selector instead of the removed private `hasSelectedStyleState`.
- Block-library custom controls (Cover, Image, Post featured image, Gallery, Navigation) are updated to use the public selectors where applicable.

**PHP – `lib/global-styles-and-settings.php`:**

```php
function gutenberg_get_viewport_media_queries( $viewport_settings = null, $options = array() ) {
    return WP_Theme_JSON_Gutenberg::get_viewport_media_queries(
        $viewport_settings,
        $options
    );
}
```

This mirrors the existing `gutenberg_get_global_settings()` pattern and provides a function-level entry point for the viewport-only subset of `getResponsiveMediaQueries`.

**Docs:** New entries added to `docs/reference-guides/data/data-core-block-editor.md` for both selectors, and to `packages/block-editor/README.md` for both hooks with usage examples.

## Contribution

Opened by @tellthemachines as part of the broader Style States feature track (#80388), closing issue #82082. Co-authored with @ramonjd and @talldan. The PR body discloses use of AI tooling (Copilot, GPT 5.6, Kimi 3.0) with human review. In a follow-up comment, @tellthemachines noted that feedback was addressed by narrowing the public surface to exactly four exports (`getStyleForState`, `setStyleForState`, `getSelectedBlockStyleState`, `hasSelectedBlockStyleState`) and deferring formal deprecation of the old private hooks to a later PR, since they are all private and can simply be removed. Merged as `387ecd0`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
