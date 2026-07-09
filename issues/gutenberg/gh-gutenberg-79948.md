# #79948: Add ariaLabel supports for Tab List Block

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jeryj
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Tabs`
- **Merged:** [`d6989b4`](https://github.com/WordPress/gutenberg/commit/d6989b4c62d29539164e79d7d447f6d790ec4856)
- **Discussion:** [#79948](https://github.com/WordPress/gutenberg/pull/79948) · 5 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Adds `ariaLabel` support to the core `tab-list` block, enabling editors to define a custom `aria-label` attribute for the `<div role="tablist">` element. A server-rendered fallback of "Tabbed content" ensures the attribute is always present, aligning the Tabs block with APG component guidelines and improving screen reader navigation.

## Impact

- **Block/Theme Developers**: The `core/tab-list` block now exposes the `ariaLabel` property via block supports. When overriding or extending the block's rendering, account for this new attribute in your PHP callbacks.
- **Editors & Site Owners**: A new Label field appears in the Tab List inspector sidebar to describe the tabbed section for assistive technology. No migration is required; existing tabs remain unchanged until manually updated.
- **Headless & REST Consumers**: The `aria-label` value is stored as a block attribute and rendered on the server, matching standard block JSON payloads.

## Technical details

- **`block.json`** (`packages/block-library/src/tab-list/block.json`): Added `"ariaLabel": true` to the `supports` object.
- **`edit.js`** (`packages/block-library/src/tab-list/edit.js`): Introduced an `InspectorControls` group containing a `ToolsPanel` with a `TextControl`. The control resets the attribute to `undefined` when deselected and passes the value via `setAttributes`.
- **`index.php`** (`packages/block-library/src/tab-list/index.php`): Updated `block_core_tab_list_render_callback` to sanitize and apply the attribute. The merged diff implements a server-side fallback: `$aria_label = empty( $attributes['ariaLabel'] ) ? __( 'Tabbed content' ) : wp_strip_all_tags( $attributes['ariaLabel'] );`. It uses `WP_HTML_Tag_Processor` to locate the `.wp-block-tab-list` node and calls `set_attribute( 'aria-label', $aria_label )`.
- **Testing**: Added PHPUnit tests verifying both custom and default aria-label outputs via `WP_HTML_Tag_Processor`.

## Contribution

Opened by @jeryj with review contributions from @t-hamano and @mamaduka. Merged as commit `d6989b4`. Review discussion centered on fallback strategy; the author initially considered client-side defaults but adopted server-side rendering for the fallback label to prevent double parsing via `WP_HTML_Tag_Processor` and ensure consistent output when attributes are cleared.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
