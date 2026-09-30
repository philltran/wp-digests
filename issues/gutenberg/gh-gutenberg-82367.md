# #82367: Icons: Support keyword-based search in the icons registry

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @n8finch
- **Labels:** `[Type] Enhancement`, `[Package] Core data`, `[Package] Block library`, `[Package] Icons`, `[Feature] Icons`
- **Merged:** [`0ed283b`](https://github.com/WordPress/gutenberg/commit/0ed283bea5c7980743007400e055b04242910c02)
- **Discussion:** [#82367](https://github.com/WordPress/gutenberg/pull/82367) · 15 comments · 1 reactions
- **Usefulness:** 4/5

## Summary

The icons registry now supports a `keywords` property on registered icons, and both the REST endpoint (`GET /wp/v2/icons?search=<term>`) and the Icon block's client-side library filter match those keywords in addition to the icon's name and label. All 88 public icons in the manifest receive search terms (e.g. "hamburger" → Menu, "gear" → Settings, "poetry" → Verse), and the Storybook icon library now reads keywords from `manifest.json` instead of a hardcoded local list. The change also fixes a test assertion in `Tests_Icons_WpGetIcon` that matched `width=` inside `stroke-width=` and only passed on trunk due to leftover state from earlier tests.

## Impact

- **Plugin & theme developers:** `wp_register_icon()` (via the Gutenberg subclass) now accepts an optional `keywords` key (array of strings). Passing a non-array or non-string value triggers `_doing_it_wrong()`. No existing registration code breaks — the property is optional and defaults to absent.
- **Headless & REST consumers:** `GET /wp/v2/icons` responses now include a `keywords` field (readonly, `string[]`, always present — empty array when the icon has no keywords). The field is available in `view`, `edit`, and `embed` contexts. No migration needed; the field is additive.
- **Icon block users / site owners:** Searching the icon library by concept ("hamburger", "gear", "shopping") now returns matching icons. No configuration change required.
- **No breaking changes.** The `keywords` property is purely additive. The `lib/compat/wordpress-7.0/` file is deliberately untouched.

## Technical details

**Registry (`lib/class-wp-icons-registry-gutenberg.php`):**
- `register()` adds `'keywords'` to `$allowed_keys` and validates it as an array of strings, calling `_doing_it_wrong()` with version `'7.2.0'` on failure.
- A new `protected function icon_matches_search( $icon, $search )` replaces the inline `stripos` checks in `get_registered_icons()`. It tests name, label, then each keyword via `stripos`.
- `get_instance()` now copies `keywords` from the base registry's icons when replaying them into the Gutenberg subclass, preventing silent loss of third-party icon keywords.

**REST controller (`lib/class-wp-rest-icons-controller-gutenberg.php`):**
- `prepare_item_for_response()` adds `$data['keywords'] = isset( $item['keywords'] ) ? array_values( $item['keywords'] ) : array();` when the field is requested.
- `get_item_schema()` adds a `keywords` property: `type: array`, `items: { type: string }`, `readonly: true`, contexts `view/edit/embed`.

**Manifest generation (`packages/icons/lib/generate-manifest-php.cjs`):**
- Emits a conditional `'keywords' => array( _x( '…', 'icon keyword', 'gutenberg' ), … )` line between `label` and `filePath` in the generated `manifest.php`. Each term is wrapped in `_x()` for translation.

**Default registration (`lib/icons.php`):**
- `gutenberg_register_default_icons()` now calls `WP_Icons_Registry_Gutenberg::get_instance()` before registering (ensuring the subclass is active, since the base `WP_Icons_Registry` rejects `keywords` and would drop the icon entirely). It also passes `$icon_data['keywords']` into `$icon_args` when present.

**Client-side (`packages/block-library/src/icon/components/custom-inserter/index.jsx`):**

```js
// Before
iconName.includes( input ) || iconLabel.includes( input )

// After
iconName.includes( input ) ||
iconLabel.includes( input ) ||
( icon.keywords ?? [] ).some( ( keyword ) =>
    normalizeSearchInput( keyword ).includes( input )
)
```

**TypeScript (`packages/core-data/src/entity-types/icon.ts`):** adds `keywords?: string[]` to the icon entity record type.

**Validation (`packages/icons/lib/validate-collection.cjs`):** build-time check that `keywords`, if present, is an array of strings.

**Storybook (`storybook/stories/icons/library.story.tsx`):** removes the hardcoded keyword map and reads from `manifest.json`.

## Contribution

The idea originated with @manhar-addweb in #76481, which proposed keyword search but targeted `lib/compat/wordpress-7.0/class-wp-icons-registry.php` — a file the live `WP_Icons_Registry_Gutenberg` subclass overrides, so the change was unreachable at runtime. That mismatch was the source of disagreement on the original thread. @n8finch picked up the work, moved it to the Gutenberg subclass, added the manifest plumbing and allowed-property change, and carried it forward after @manhar-addweb did not respond to a request to continue. @simison suggested consolidating Storybook's hardcoded keyword list into `manifest.json` and contributed additional keyword terms; @jasmussen reviewed and offered to help with keyword vocabulary. The keyword list itself was AI-generated (Claude Opus 5) and reviewed by the author, who flagged it as the most subjective part and invited community suggestions. Two corrections emerged during review: the missing REST exposure was caught by inspecting the live `/wp/v2/icons` payload, and a PHPUnit failure exposed a registration-ordering bug where passing `keywords` to the base registry dropped the icon entirely rather than just losing the keywords.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
