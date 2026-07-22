# #79210: Style states: Use symbols for property keys to avoid clashes

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @talldan
- **Labels:** `[Type] Enhancement`, `[Package] Blocks`, `[Package] Editor`, `[Package] Block library`, `[Package] Block editor`, `[Feature] Style States`
- **Merged:** [`8ace4f7`](https://github.com/WordPress/gutenberg/commit/8ace4f7ef243632eb358d3ae6508cbed1b9e7acd)
- **Discussion:** [#79210](https://github.com/WordPress/gutenberg/pull/79210) · 12 comments · 5 reactions
- **Usefulness:** 4/5

## Summary

The Style States feature now prefixes responsive breakpoint keys with `@` (e.g., `@mobile`, `@tablet`) and custom state keys with `-` (e.g., `-current`) to prevent key collisions with future arbitrary state definitions. This change updates the canonical representation of style states across PHP and JavaScript, while introducing a backward-compatibility resolver that transparently maps legacy unprefixed keys to their new forms.

## Impact

- **Theme & plugin developers:** No immediate action required. The `WP_Theme_JSON_Gutenberg` class and block editor hooks now expect prefixed keys. Legacy `mobile`/`tablet` keys in `theme.json` or block attributes are automatically resolved via `gutenberg_resolve_style_state_aliases()`.
- **Block developers:** If you directly read or write `style['mobile']` or `style['tablet']` in block attributes, update references to `style['@mobile']` and `style['@tablet']`. The editor UI and `getStyleForState()` helper now use the prefixed keys.
- **Hosting & platform teams:** No configuration changes needed. The backward-compatibility shim ensures existing serialized block content and global styles continue to render correctly without migration scripts.

## Technical details

The diff replaces hardcoded `mobile`/`tablet` strings with `@mobile`/`@tablet` across `WP_Theme_JSON_Gutenberg::RESPONSIVE_BREAKPOINTS`, `lib/block-supports/layout.php`, `lib/block-supports/states.php`, and multiple JS hooks (`packages/block-editor/src/hooks/layout.js`, `style.js`, `states.js`). Custom states for `core/navigation-link` shift from `@current` to `-current` in `WP_Theme_JSON_Gutenberg::VALID_BLOCK_CUSTOM_STATES`.

A new PHP function `gutenberg_resolve_style_state_aliases()` (in `lib/compat/plugin/style-state-aliases.php`) recursively normalizes legacy keys to their prefixed equivalents. It is invoked during `WP_Theme_JSON_Gutenberg` construction, `remove_insecure_properties()`, and block rendering in `layout.php` and `states.php`.

In JS, the editor now uses a `getStyleForState()` helper instead of direct bracket notation, and `RESPONSIVE_STATE_LABELS` maps `@mobile`/`@tablet` to UI strings. Tests in `packages/block-editor/src/hooks/test/` are updated to reflect the new key structure.

**Before/After:**
```js
// Before
const viewportStyle = attributes?.style?.[ viewport ]; // viewport = 'mobile'

// After
const viewportStyle = getStyleForState( attributes?.style, {
    viewport: '@mobile',
    pseudo: DEFAULT_BLOCK_STYLE_STATE.pseudo,
} );
```

## Contribution

Opened by @talldan to address a design risk where unprefixed responsive keys could clash with future arbitrary state definitions. The PR considered nesting state data under a `states` property but rejected it due to conflicts with existing pseudo-state implementations. The team settled on CSS-inspired prefixes (`@` for responsive, `-` for custom) to maintain path consistency. @talldan implemented a recursive backward-compatibility resolver to handle legacy serialized data, and the change was merged with co-authors including @MaggieCabrera, @tellthemachines, @ramonjd, @andrewserong, and @youknowriad.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
